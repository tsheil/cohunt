---
name: cloud-security
description: Cloud security misconfiguration patterns for AWS, GCP, and Azure — storage, identity, serverless, metadata, AI/ML services, and multi-cloud pivots. Use when a target is cloud-hosted, uses cloud services (S3, GCS, Azure Blob, Lambda, Cloud Functions, SageMaker, Vertex AI, Bedrock, Azure OpenAI), or when recon reveals cloud infrastructure. Trigger with "cloud misconfig", "S3 bucket", "AWS security", "GCP security", "Azure security", "cloud enumeration", "serverless security", "metadata endpoint", "cloud storage", "IAM", "SageMaker", "Vertex AI", "Bedrock", "Azure OpenAI", "cloud AI", "multi-cloud", "workload identity", "federation", or when target-recon identifies cloud hosting. Also activates when SSRF testing reveals cloud metadata endpoints, or when storage URLs (s3.amazonaws.com, storage.googleapis.com, blob.core.windows.net) appear in scope. For SSRF techniques to reach cloud metadata, use vuln-patterns. For CI/CD pipeline attacks in cloud environments, use supply-chain-security. For prompt injection and model-behavior attacks on AI features, use ai-hunting (cloud-security owns IAM/network/storage/backing-store for AI services; ai-hunting owns model-behavior attacks).
---

# Cloud Security Misconfiguration Patterns

Cloud misconfigs are consistently rewarded in bug bounties — storage exposure, metadata SSRF, IAM overreach, and AI/ML service misconfigurations. This skill covers all three major providers plus the rapidly growing AI cloud attack surface.

## Quick Start — First 5 Minutes

1. **Storage** — Find bucket/container names from JS, subdomains, error pages. `curl https://{bucket}.s3.amazonaws.com/` — look for `<ListBucketResult>`.
2. **Metadata** — Any SSRF? Hit `http://169.254.169.254/latest/meta-data/`. If IMDSv2, try redirect/DNS rebinding bypass.
3. **IAM keys** — Search JS, `.env`, CI configs, Terraform state for `AKIA*` (AWS), JSON key files (GCP), `?sv=` SAS tokens (Azure).
4. **Serverless** — Find Lambda/Function URLs. Test `?code=` key exposure, anonymous invoke, event injection.
5. **AI services** — Target uses SageMaker/Vertex/Bedrock/Azure OpenAI? Test API parity, backing-store exposure, execution identity overreach. See [reference/cloud-ai-ml.md](reference/cloud-ai-ml.md).

If you find something, run `/scope-check` first (cloud assets are often vendor-owned), then `/reportability-check` before `/write-report`.

---

## Execution Flow

```
1. Identify cloud provider(s) from recon data
2. Run scope gate — is this the target's asset or vendor infrastructure?
3. Enumerate cloud attack surface (storage, compute, APIs, AI services)
4. Test storage misconfigurations
5. Test metadata/SSRF paths
6. Test identity and access misconfigurations
7. Test serverless and AI/ML service issues
8. Check for secrets and credential exposure
9. Test multi-cloud federation and lateral movement
10. Run validation loop before reporting
```

---

## Scope Gate (Run First)

Cloud findings get N/A'd fast when the asset is vendor-owned. Before testing:

| Signal | Verdict |
|--------|---------|
| S3 bucket owned by AWS, not the program | **Out of scope** — vendor infrastructure |
| Azure Blob on `microsoft.com` tenant | **Out of scope** — verify target ownership |
| GCS bucket matches target's naming convention | **Likely in scope** — verify with program policy |
| Cloud metadata via SSRF on in-scope target | **In scope** — the SSRF is the finding |
| AI service endpoint (SageMaker/Vertex/Bedrock) | **Check** — endpoint itself is often AWS/GCP-owned; the misconfiguration in target's setup is the finding |

When in doubt: `/scope-check <finding>`. Cloud scope is the #1 reason for wasted cloud reports.

---

## Decision Logic — When to Invest vs. Move On

| Signal | Action |
|--------|--------|
| Public bucket listing returns `<ListBucketResult>` | **INVEST** — enumerate objects, check for PII/creds/backups |
| SSRF reaches metadata, returns IAM role name | **INVEST** — extract creds, test what the role can access |
| Leaked access key (`AKIA*`) found in JS/source | **INVEST** — `aws sts get-caller-identity`, enumerate permissions |
| Public bucket listing but only static assets (images, CSS) | **LOG + MOVE ON** — report as low/medium, don't spend hours |
| IMDSv2 enforced, no redirect/rebinding bypass works | **MOVE ON** — note the hardening, test other surfaces |
| Cognito identity pool returns unauthenticated ID | **INVEST** — test what the unauthenticated role can access (often surprising) |
| Function URL responds to unauthenticated requests | **INVEST** — test what the function does, check for injection/data exposure |
| AI service endpoint accessible but only returns your own data | **MOVE ON** — self-only access is not a finding |
| Federation trust allows cross-account assume-role | **INVEST** — test what the assumed role can access in the other account |
| Terraform state file found in public bucket | **CRITICAL** — contains full infrastructure map, secrets, connection strings |

**Time-boxing:** Spend max 30 minutes on cloud enumeration per provider. If no high-signal indicators after 30 minutes, move to another attack surface.

---

## Cloud Report Killers

These patterns get N/A or Informational — don't waste time reporting:

| Pattern | Why It's N/A |
|---------|-------------|
| Public bucket with only static assets (CSS, images, fonts) | No sensitive data — by design |
| Bucket exists but returns 403 on all operations | Access denied = working as intended |
| SSRF to metadata but IMDSv2 blocks it | Hardened — no impact demonstrated |
| Exposed endpoint returning only your own data | Self-only access, no cross-user impact |
| Theoretical IAM abuse without demonstrated artifact/data access | Need to prove actual access, not just permissions |
| SAS token in URL with narrow scope (single blob, short expiry) | By-design functionality |
| AI endpoint accessible but behind proper auth | Not a misconfig |
| Subdomain takeover on non-cloud-hosted asset | Wrong skill — use `target-recon` |

---

## Validation Loop (Before Reporting)

Before writing up any cloud finding, run these checks:

```
┌─ CLOUD FINDING VALIDATION ──────────────────────────────────────┐
│ □ 1. SCOPE — Is this the target's cloud asset, not the vendor's?│
│      Run /scope-check if uncertain                               │
│ □ 2. DATA PLANE — Did you access actual sensitive data?          │
│      (Creds, PII, backups, source code, Terraform state)         │
│      If only static assets: downgrade severity                   │
│ □ 3. CONTROL PLANE — Can you modify resources?                   │
│      (Write to bucket, invoke function, assume role)             │
│      Demonstrate the action, don't just claim it                 │
│ □ 4. CROSS-BOUNDARY — Does this cross a trust boundary?          │
│      (Unauth→auth, tenant-A→tenant-B, user→admin)               │
│ □ 5. EVIDENCE — Capture:                                         │
│      - HTTP requests/responses showing the access                │
│      - Redacted samples of sensitive data found                  │
│      - IAM role/policy showing excessive permissions             │
│      - Two-account proof if access control issue                 │
│ □ 6. REPRODUCIBILITY — Can another person reproduce this?        │
│      (Not dependent on your specific AWS account, IP, etc.)      │
└──────────────────────────────────────────────────────────────────┘
```

---

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
| 8 | Object version exposure | `curl https://{bucket}.s3.amazonaws.com/?versions` — access deleted files |

**Finding bucket names:** Subdomains (CNAME → `s3.amazonaws.com`), JavaScript source, error pages, `x-amz-*` headers, robots.txt/sitemap.xml.

### GCS

| # | Pattern | Test |
|---|---------|------|
| 1 | Public bucket listing | `curl https://storage.googleapis.com/{bucket}/` |
| 2 | Public object read | `curl https://storage.googleapis.com/{bucket}/{object}` |
| 3 | IAM policy exposure | `curl https://storage.googleapis.com/storage/v1/b/{bucket}/iam` |
| 4 | allUsers / allAuthenticatedUsers | Check if roles granted to public principal |
| 5 | Signed URL abuse | Test expiry, scope, and reuse of signed URLs |

### Azure Blob Storage

| # | Pattern | Test |
|---|---------|------|
| 1 | Public container listing | `curl "https://{account}.blob.core.windows.net/{container}?restype=container&comp=list"` |
| 2 | Public blob read | `curl https://{account}.blob.core.windows.net/{container}/{blob}` |
| 3 | SAS token abuse | Check for overly permissive shared access signatures in URLs |
| 4 | SAS token in source | Search JS/HTML for `?sv=` parameters containing SAS tokens |
| 5 | Snapshot access | Append `?snapshot={timestamp}` to access point-in-time versions |
| 6 | Soft-deleted blobs | List with `?comp=list&include=deleted` |

---

## Cloud Metadata / SSRF

When you find SSRF, cloud metadata endpoints are the highest-impact targets. See `vuln-patterns` for SSRF techniques.

### Endpoints by Provider

| Provider | Endpoint | Header Required | What to Extract |
|----------|----------|-----------------|-----------------|
| **AWS IMDSv1** | `http://169.254.169.254/latest/meta-data/iam/security-credentials/{role}` | None | IAM creds (AccessKeyId, SecretAccessKey, Token) |
| **AWS IMDSv2** | Same, but requires PUT for token first | `X-aws-ec2-metadata-token` | Same — test if IMDSv2 is actually enforced |
| **GCP** | `http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token` | `Metadata-Flavor: Google` | OAuth2 access token, project attributes |
| **Azure** | `http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://management.azure.com/` | `Metadata: true` | Managed identity tokens, subscription/resource info |
| **ECS** | `http://169.254.170.2/` | None | Container task credentials |

### SSRF Bypasses for Metadata

| Bypass | Technique |
|--------|-----------|
| Decimal IP | `http://2852039166/` |
| Hex IP | `http://0xa9fea9fe/` |
| IPv6 mapped | `http://[::ffff:169.254.169.254]/` |
| DNS rebinding | Point your domain at 169.254.169.254 |
| Redirect | SSRF to your server → 302 redirect to metadata |
| Alternate domains | `http://metadata.google.internal/` (GCP) |

---

## Identity & Access Misconfigurations

### AWS IAM

| # | Pattern | Test |
|---|---------|------|
| 1 | Overprivileged access keys | Leaked keys → `aws sts get-caller-identity`, enumerate permissions |
| 2 | AssumeRole chain | Enumerate roles, test cross-account assume-role paths |
| 3 | Cognito identity pool | Unauthenticated pool access — `aws cognito-identity get-id` |
| 4 | Cognito user pool | Self-registration, attribute manipulation |
| 5 | STS from SSRF | Metadata creds → escalate via STS |
| 6 | S3 bucket policy wildcards | `Principal: "*"` or `Principal: {"AWS": "*"}` |
| 7 | Long-term key exposure | Crimson Collective pattern (March 2026): ~570GB exfiltrated via exposed keys |

### GCP IAM

| # | Pattern | Test |
|---|---------|------|
| 1 | allUsers binding | Roles granted to allUsers or allAuthenticatedUsers |
| 2 | Service account key exposure | JSON key files in repos, configs, Docker images |
| 3 | Default service account | Compute Engine default SA often has project-editor role |
| 4 | Workload identity | GKE pods with overprivileged service accounts |

### Azure AD / Entra ID

| # | Pattern | Test |
|---|---------|------|
| 1 | App registration secrets | Exposed client secrets in source code |
| 2 | Overprivileged managed identity | VM/function identity with broad role assignments |
| 3 | Tenant enumeration | `https://login.microsoftonline.com/{domain}/.well-known/openid-configuration` |
| 4 | Guest user escalation | Guest accounts with access to internal resources |
| 5 | Actor Token abuse | CVE-2025-55241 (CVSS 10.0) — Global Administrator takeover |
| 6 | Conditional access gaps | Test from different locations, devices, risk levels |

---

## Serverless Security

| Provider | # | Pattern | Test |
|----------|---|---------|------|
| **AWS Lambda** | 1 | Event injection | Inject payloads through API Gateway, S3, SNS, SQS event sources |
| | 2 | Environment variables | SSRF → metadata → function config; secrets in env vars |
| | 3 | Layer poisoning | Public layers that could be modified |
| | 4 | /tmp persistence | Data persists between warm invocations |
| **GCP Functions** | 1 | Unauthenticated invocation | `allUsers` has `cloudfunctions.functions.invoke` |
| | 2 | Source code exposure | `gcloud functions describe` reveals source location |
| | 3 | Service account token | Access token via metadata endpoint |
| **Azure Functions** | 1 | Function key exposure | `?code=` parameter in URLs, source code |
| | 2 | Anonymous auth | `authLevel: anonymous` |
| | 3 | Host key leakage | Master key → access to all functions |

---

## Kubernetes / Container Security

When the target runs on Kubernetes (EKS, GKE, AKS):

| # | Pattern | Test |
|---|---------|------|
| 1 | Exposed API server | `curl -k https://{ip}:6443/api` |
| 2 | Kubelet API | `curl -k https://{node-ip}:10250/pods` |
| 3 | etcd exposure | `curl http://{ip}:2379/v2/keys/` |
| 4 | Service account token | `/var/run/secrets/kubernetes.io/serviceaccount/token` |
| 5 | Dashboard exposure | Unauthenticated Kubernetes Dashboard |
| 6 | Container escape | Privileged containers, mounted Docker socket |
| 7 | RBAC misconfiguration | ClusterRoleBinding with `cluster-admin` to default SA |
| 8 | Network policy gaps | Pod-to-pod communication without restriction |

---

## Cloud SaaS Analytics — Cross-Tenant Data Access

Cloud analytics platforms (Looker Studio, Power BI, Tableau Cloud, Databricks) connect to backend databases using service credentials. Connector misconfigurations create cross-tenant data access — one customer's report can query another customer's data.

### LeakyLooker Pattern (Tenable, March 2026)

Nine cross-tenant vulnerabilities in Google Looker Studio across multiple distinct issue types:

| # | Vector | How It Works | Impact |
|---|--------|-------------|--------|
| 1 | Stored-credential inheritance (PostgreSQL-like connectors) | Clone a public report → cloned report retains original owner's database credentials | Read/write access to victim's PostgreSQL/MySQL databases |
| 2 | Linking API abuse (BigQuery/Spanner) | Separate API paths allow arbitrary query execution across GCP projects | Cross-project data access via BigQuery/Spanner |
| 3 | Browser-proxied exfiltration | Shared report forces victim's browser to execute queries against attacker-controlled project → data reconstructed from logs | Mass data exfiltration via browser proxy |

**Where to hunt this pattern** (Looker Studio confirmed; others are untested hunt hypotheses):

| Platform | Test | What to look for |
|----------|------|-----------------|
| **Google Looker Studio** | Clone public reports with database connectors | Cloned report queries original owner's data (confirmed by Tenable) |
| **Power BI** | Shared datasets, embedded reports | Dataset credential sharing across tenants (hypothesis) |
| **Tableau Cloud** | Published datasources with embedded credentials | Cross-site datasource access (hypothesis) |
| **Databricks** | Shared notebooks with Unity Catalog | Cross-workspace table access (hypothesis) |
| **Metabase** | Shared dashboards, public question links | Database credential leakage via shared artifacts (hypothesis) |

**Hunt heuristic:** Any cloud analytics tool that allows sharing/copying reports while using service-level database credentials is a candidate. Test: Can you access data from a project/tenant you don't own by copying or viewing a shared report?

---

## Secrets & Credential Exposure

| Secret Type | Where to Find | Impact |
|-------------|---------------|--------|
| AWS access keys (`AKIA*`) | Source code, env vars, CI configs | Full account access |
| GCP service account JSON | Repos, Docker images, configs | Project-level access |
| Azure client secrets | App configs, env vars | Tenant-level access |
| Kubernetes kubeconfig | Developer machines, CI pipelines | Cluster admin |
| SAS tokens / presigned URLs | JavaScript, API responses | Data access |
| Terraform state files | Public S3/GCS buckets | Full infrastructure map + secrets |
| `.env` files | Public repos, misconfigured servers | Multiple services |

---

## Severity Guidelines

| Finding | Typical Severity | Notes |
|---------|-----------------|-------|
| Public bucket with PII/creds | Critical-High | Depends on data sensitivity |
| SSRF → metadata → IAM creds | Critical | Full account compromise |
| Exposed service account key | Critical-High | Depends on key permissions |
| AI service backing-store exposed | High-Critical | Training data, prompts, embeddings leaked |
| Unauthenticated Lambda/Function invoke | Medium-High | Depends on function purpose |
| Public storage listing (no sensitive data) | Low-Medium | Report but don't oversell |
| Terraform state in public bucket | Critical | Contains secrets, full infra map |
| Cross-cloud federation trust abuse | High-Critical | Lateral movement across providers |
| Exposed Kubernetes Dashboard | High-Critical | Cluster compromise |
| Cross-tenant analytics connector abuse | High-Critical | Arbitrary SQL on victim's databases (LeakyLooker pattern) |

---

## Reference Files

| File | Contents |
|------|----------|
| [reference/cloud-ai-ml.md](reference/cloud-ai-ml.md) | AI/ML cloud service attack patterns — SageMaker, Vertex AI, Bedrock, Azure OpenAI, API parity testing, backing-store exposure, execution identity overreach |
| [reference/multi-cloud-pivots.md](reference/multi-cloud-pivots.md) | Workload identity federation, cross-cloud lateral movement, multi-cloud credential chains |

## Related Skills

- `target-recon` — Identifies cloud hosting during reconnaissance
- `vuln-patterns` — SSRF patterns for reaching metadata endpoints
- `source-code-audit` — Find hardcoded cloud credentials in source code
- `ai-hunting` — Model-behavior attacks (prompt injection, tool poisoning) on AI cloud features
- `supply-chain-security` — CI/CD pipeline attacks in cloud environments
- `auth-testing` — OAuth/SSO/IAM access control testing
