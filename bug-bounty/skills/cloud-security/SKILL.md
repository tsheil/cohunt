---
name: cloud-security
description: Cloud security misconfiguration patterns for AWS, GCP, and Azure. Use when a target is cloud-hosted, uses cloud services (S3, GCS, Azure Blob, Lambda, Cloud Functions), or when recon reveals cloud infrastructure. Trigger with "cloud misconfig", "S3 bucket", "AWS security", "GCP security", "Azure security", "cloud enumeration", "serverless security", "metadata endpoint", "cloud storage", "IAM", or when target-recon identifies cloud hosting. Also activates when SSRF testing reveals cloud metadata endpoints, or when storage URLs (s3.amazonaws.com, storage.googleapis.com, blob.core.windows.net) appear in scope. For SSRF techniques to reach cloud metadata, use vuln-patterns. For CI/CD pipeline attacks in cloud environments, use supply-chain-security.
---

Cloud misconfiguration patterns for bug bounty hunting. Covers storage, compute, identity, serverless, and metadata attack surfaces across AWS, GCP, and Azure.

## Execution Flow

```
1. Identify cloud provider(s) from recon data or user input
2. Enumerate cloud-specific attack surface (storage, compute, APIs)
3. Test storage misconfigurations
4. Test metadata/SSRF paths
5. Test identity and access misconfigurations
6. Test serverless-specific issues
7. Check for secrets and credential exposure
8. Synthesize findings with cloud-specific impact assessment
```

## Cloud Storage Misconfigurations

The most common and consistently rewarded cloud bug class.

### AWS S3

| # | Pattern | Test |
|---|---------|------|
| 1 | Public bucket listing | `curl https://{bucket}.s3.amazonaws.com/` — look for `<ListBucketResult>` |
| 2 | Public object read | `curl https://{bucket}.s3.amazonaws.com/{key}` — test known/guessable keys |
| 3 | Unauthenticated write | `curl -X PUT https://{bucket}.s3.amazonaws.com/test.txt -d "test"` |
| 4 | ACL misconfiguration | `curl https://{bucket}.s3.amazonaws.com/?acl` — check for public grants |
| 5 | Bucket policy exposure | `curl https://{bucket}.s3.amazonaws.com/?policy` |
| 6 | Presigned URL abuse | Check if presigned URLs have excessive expiry or broad permissions |
| 7 | Cross-account access | Authenticated requests from your own AWS account to target buckets |
| 8 | Bucket name enumeration | HTTP 403 = exists, 404 = doesn't — enumerate naming patterns |
| 9 | Object version exposure | `curl https://{bucket}.s3.amazonaws.com/?versions` — access deleted files |
| 10 | Website-enabled bucket | `curl http://{bucket}.s3-website-{region}.amazonaws.com/` |

**Finding bucket names:**
- Subdomains pointing to S3 (CNAME records)
- JavaScript source referencing S3 URLs
- Error pages leaking bucket names
- Response headers containing `x-amz-*` values
- robots.txt / sitemap.xml referencing S3 paths

### Google Cloud Storage (GCS)

| # | Pattern | Test |
|---|---------|------|
| 1 | Public bucket listing | `curl https://storage.googleapis.com/{bucket}/` |
| 2 | Public object read | `curl https://storage.googleapis.com/{bucket}/{object}` |
| 3 | Unauthenticated upload | `curl -X PUT -d "test" https://storage.googleapis.com/upload/storage/v1/b/{bucket}/o?uploadType=media&name=test.txt` |
| 4 | IAM policy exposure | `curl https://storage.googleapis.com/storage/v1/b/{bucket}/iam` |
| 5 | allUsers / allAuthenticatedUsers | Check if roles granted to public principal |
| 6 | Bucket metadata | `curl https://storage.googleapis.com/storage/v1/b/{bucket}` |
| 7 | Object ACL | `curl https://storage.googleapis.com/storage/v1/b/{bucket}/o/{object}/acl` |
| 8 | Signed URL abuse | Test expiry, scope, and reuse of signed URLs |

### Azure Blob Storage

| # | Pattern | Test |
|---|---------|------|
| 1 | Public container listing | `curl "https://{account}.blob.core.windows.net/{container}?restype=container&comp=list"` |
| 2 | Public blob read | `curl https://{account}.blob.core.windows.net/{container}/{blob}` |
| 3 | Anonymous write | Attempt PUT to public containers |
| 4 | SAS token abuse | Check for overly permissive shared access signatures in URLs |
| 5 | SAS token in source | Search JS/HTML for `?sv=` parameters containing SAS tokens |
| 6 | Account enumeration | DNS resolution of `{name}.blob.core.windows.net` |
| 7 | Snapshot access | Append `?snapshot={timestamp}` to access point-in-time versions |
| 8 | Soft-deleted blobs | List with `?comp=list&include=deleted` |

## Cloud Metadata / SSRF

When you find SSRF, cloud metadata endpoints are the highest-impact targets.

### AWS IMDSv1 / IMDSv2

```
# IMDSv1 (no authentication required)
http://169.254.169.254/latest/meta-data/
http://169.254.169.254/latest/meta-data/iam/security-credentials/
http://169.254.169.254/latest/meta-data/iam/security-credentials/{role-name}
http://169.254.169.254/latest/user-data

# IMDSv2 (requires token — test if target enforces it)
TOKEN=$(curl -X PUT http://169.254.169.254/latest/api/token -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/

# Alternative endpoints
http://[fd00:ec2::254]/latest/meta-data/  # IPv6
http://169.254.169.254/latest/dynamic/instance-identity/document  # Instance identity
```

**What to extract from AWS metadata:**
- IAM role credentials (AccessKeyId, SecretAccessKey, Token)
- Instance identity document (account ID, region, instance ID)
- User-data scripts (often contain secrets, bootstrap configs)
- Network interfaces and security group info

### GCP Metadata

```
# Requires Metadata-Flavor header
curl -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/
curl -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token
curl -H "Metadata-Flavor: Google" http://metadata.google.internal/computeMetadata/v1/project/attributes/

# Alternative
http://169.254.169.254/computeMetadata/v1/  # Also works
```

**What to extract from GCP metadata:**
- Service account access tokens (OAuth2)
- Project-level attributes and secrets
- Instance attributes and startup scripts
- Kubernetes service account tokens (on GKE)

### Azure IMDS

```
curl -H "Metadata: true" "http://169.254.169.254/metadata/instance?api-version=2021-02-01"
curl -H "Metadata: true" "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/"

# Managed identity token
curl -H "Metadata: true" "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://storage.azure.com/"
```

**What to extract from Azure IMDS:**
- Managed identity access tokens (for Azure services)
- Subscription ID, resource group, VM name
- Custom data and user data

### SSRF Bypasses for Metadata

When basic `169.254.169.254` is blocked:

| Bypass | Technique |
|--------|-----------|
| Decimal IP | `http://2852039166/` (169.254.169.254 as decimal) |
| Hex IP | `http://0xa9fea9fe/` |
| Octal IP | `http://0251.0376.0251.0376/` |
| IPv6 mapped | `http://[::ffff:169.254.169.254]/` |
| DNS rebinding | Point your domain at 169.254.169.254 |
| Redirect | SSRF to your server → 302 redirect to metadata |
| URL encoding | `http://169.254.169.254%23@your-domain/` |
| Alternate domains | `http://metadata.google.internal/` (GCP) |
| Container endpoints | `http://169.254.170.2/` (ECS task metadata) |

## Identity & Access Misconfigurations

### AWS IAM

| # | Pattern | Test |
|---|---------|------|
| 1 | Overprivileged access keys | Leaked keys → enumerate permissions with `aws iam get-user`, `aws sts get-caller-identity` |
| 2 | AssumeRole chain | Enumerate roles, test cross-account assume-role paths |
| 3 | Cognito identity pool | Check for unauthenticated identity pool access — `aws cognito-identity get-id` |
| 4 | Cognito user pool | Test for self-registration, attribute manipulation |
| 5 | STS token from SSRF | Metadata credentials → escalate via STS |
| 6 | Lambda execution role | Invoke function → check what role it runs with |
| 7 | EC2 instance profile | SSRF → metadata → role credentials |
| 8 | S3 bucket policy wildcards | `Principal: "*"` or `Principal: {"AWS": "*"}` |
| 9 | Long-term access key exposure | Enumerate exposed IAM access keys (Crimson Collective pattern, March 2026: ~570GB exfiltrated from Red Hat GitLab via exposed keys) | Unauthorized API access using leaked long-term credentials |

### GCP IAM

| # | Pattern | Test |
|---|---------|------|
| 1 | allUsers binding | Check if roles are granted to allUsers or allAuthenticatedUsers |
| 2 | Service account key exposure | Search for JSON key files in repos, configs |
| 3 | Default service account | Compute Engine default SA often has project-editor role |
| 4 | OAuth consent screen | Test if app can request broad scopes |
| 5 | Workload identity | GKE pods with overprivileged service accounts |

### Azure AD / Entra ID

| # | Pattern | Test |
|---|---------|------|
| 1 | App registration secrets | Exposed client secrets in source code |
| 2 | Overprivileged managed identity | VM/function identity with broad role assignments |
| 3 | Tenant enumeration | `https://login.microsoftonline.com/{domain}/.well-known/openid-configuration` |
| 4 | Guest user escalation | Guest accounts with access to internal resources |
| 5 | Conditional access gaps | Test from different locations, devices, risk levels |
| 6 | Actor Token abuse | Test Actor Tokens authentication mechanism for privilege escalation — CVE-2025-55241 (CVSS 10.0) enabled Global Administrator takeover |
| 7 | Token exchange attacks | Test token exchange flows between services for privilege escalation via intermediate tokens |

## Serverless Security

### AWS Lambda

| # | Pattern | Test |
|---|---------|------|
| 1 | Event injection | Inject payloads through event sources (API Gateway, S3, SNS, SQS) |
| 2 | Environment variables | Functions often store secrets in env vars — SSRF → metadata → function config |
| 3 | /tmp persistence | Data persists in /tmp between warm invocations |
| 4 | Layer poisoning | Check if function uses public layers that could be modified |
| 5 | Dependency confusion | Function packages pulling from public registries |
| 6 | Timeout abuse | Long-running operations for resource exhaustion |
| 7 | Cold start injection | Modify initialization code path via environment manipulation |

### GCP Cloud Functions / Cloud Run

| # | Pattern | Test |
|---|---------|------|
| 1 | Unauthenticated invocation | Check if `allUsers` has `cloudfunctions.functions.invoke` |
| 2 | Source code exposure | `gcloud functions describe` may reveal source location |
| 3 | Service account token | Access token via metadata endpoint from within function |
| 4 | Environment variable secrets | Similar to Lambda — secrets in env vars |

### Azure Functions

| # | Pattern | Test |
|---|---------|------|
| 1 | Function key exposure | Check for `?code=` parameter in URLs, source code |
| 2 | Anonymous authentication | Functions configured with `authLevel: anonymous` |
| 3 | Managed identity escalation | Function identity with broad Azure role assignments |
| 4 | Host key leakage | Master key exposure allows access to all functions |

## Kubernetes / Container Security

When the target runs on Kubernetes (EKS, GKE, AKS):

| # | Pattern | Test |
|---|---------|------|
| 1 | Exposed API server | `curl -k https://{ip}:6443/api` |
| 2 | Kubelet API | `curl -k https://{node-ip}:10250/pods` |
| 3 | etcd exposure | `curl http://{ip}:2379/v2/keys/` |
| 4 | Service account token | `/var/run/secrets/kubernetes.io/serviceaccount/token` |
| 5 | Dashboard exposure | Check for unauthenticated Kubernetes Dashboard |
| 6 | Helm tiller | Legacy Helm 2 Tiller API (port 44134) |
| 7 | Container escape | Privileged containers, mounted Docker socket |
| 8 | Network policy gaps | Pod-to-pod communication without restriction |
| 9 | ConfigMap/Secret exposure | Secrets stored as base64 in ConfigMaps |
| 10 | RBAC misconfiguration | ClusterRoleBinding with `cluster-admin` to default SA |

## Secrets & Credential Exposure

Cloud-specific secrets to hunt for:

| Secret Type | Where to Find | Impact |
|-------------|---------------|--------|
| AWS access keys | Source code, env vars, CI configs | Full account access |
| GCP service account JSON | Repos, Docker images, configs | Project-level access |
| Azure client secrets | App configs, env vars | Tenant-level access |
| Kubernetes kubeconfig | Developer machines, CI pipelines | Cluster admin |
| Cloud storage URLs with tokens | JavaScript, API responses | Data access |
| Database connection strings | Environment variables, configs | Data breach |
| Terraform state files | Public S3/GCS buckets | Full infrastructure map |
| `.env` files | Public repos, misconfigured servers | Multiple services |

## Cloud-Specific Impact Assessment

When reporting cloud misconfigurations, frame impact in terms of:

1. **Data exposure** — What sensitive data is accessible? PII, credentials, business data?
2. **Lateral movement** — Can the misconfiguration be used to access other cloud resources?
3. **Privilege escalation** — Can a low-privilege finding lead to admin access?
4. **Persistence** — Can an attacker maintain access after the initial misconfiguration is fixed?
5. **Blast radius** — How many accounts, services, or users are affected?

### Severity Guidelines

| Finding | Typical Severity | Notes |
|---------|-----------------|-------|
| Public S3 bucket with PII | Critical–High | Depends on data sensitivity |
| SSRF to metadata → IAM creds | Critical | Full account compromise potential |
| Exposed service account key | Critical–High | Depends on key permissions |
| Unauthenticated Lambda invoke | Medium–High | Depends on function purpose |
| Public storage listing (no sensitive data) | Low–Medium | Still worth reporting |
| Subdomain pointing to deprovisioned cloud resource | Medium | Subdomain takeover |
| Exposed Kubernetes Dashboard | High–Critical | Cluster compromise |

## Related Skills

- `target-recon` — Identifies cloud hosting during reconnaissance
- `vuln-patterns` — SSRF patterns for reaching metadata endpoints
- `source-code-audit` — Find hardcoded cloud credentials in source code
