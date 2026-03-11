# Multi-Cloud Federation & Lateral Movement

Patterns for pivoting between cloud providers, abusing workload identity federation, and chaining cloud misconfigurations for cross-boundary access. These techniques turn a single-cloud finding into a multi-cloud compromise.

---

## Table of Contents

- [Workload Identity Federation](#workload-identity-federation)
- [Cross-Cloud Credential Chains](#cross-cloud-credential-chains)
- [Federation Trust Abuse Patterns](#federation-trust-abuse-patterns)
- [Multi-Cloud Secrets Sprawl](#multi-cloud-secrets-sprawl)
- [Cloud-to-On-Prem Pivots](#cloud-to-on-prem-pivots)

---

## Workload Identity Federation

Modern cloud deployments use federation to let workloads in one provider authenticate to another without static credentials. Misconfigured federation trusts are high-value findings — they enable cross-cloud lateral movement.

### AWS OIDC Identity Provider

AWS allows external identity providers (including GCP and Azure) to assume IAM roles via OIDC federation.

| # | Test | What to Look For |
|---|------|-----------------|
| 1 | List OIDC providers | `aws iam list-open-id-connect-providers` — find federated trust relationships |
| 2 | Check trust policy | Examine AssumeRolePolicyDocument for overly broad `Condition` blocks |
| 3 | Audience validation | Is `aud` claim validated? Missing audience check = any token from the IdP works |
| 4 | Subject validation | Is `sub` claim validated? Missing subject check = any workload from the IdP can assume the role |
| 5 | Cross-account federation | Federation trust allows roles in other AWS accounts → test assume-role chain |

**Common misconfiguration:** Trust policy checks the OIDC issuer but not the `sub` claim — any GCP service account or Azure managed identity from the trusted provider can assume the AWS role.

### Google Cloud Workload Identity Federation

GCP allows AWS and Azure workloads to authenticate as GCP service accounts without static keys.

| # | Test | What to Look For |
|---|------|-----------------|
| 1 | Workload identity pools | `gcloud iam workload-identity-pools list` — find federation configs |
| 2 | Provider attribute mapping | Check `attribute.aws_role` or `attribute.azure_subject` mapping — is it too broad? |
| 3 | Attribute condition | Missing condition = any identity from the provider can impersonate the service account |
| 4 | Service account impersonation | Which SA can be impersonated? Check permissions of the target SA |

**Common misconfiguration:** Workload identity pool maps all AWS roles from a trusted account to a single GCP service account with broad permissions.

### Microsoft Entra Workload Identity Federation

Azure allows AWS and GCP workloads to authenticate as Entra applications.

| # | Test | What to Look For |
|---|------|-----------------|
| 1 | Federated credentials | Check app registration for federated credential configs |
| 2 | Subject validation | Is the `subject` field specific (single SA/role) or broad (wildcard)? |
| 3 | Issuer validation | Is the issuer locked to a specific provider account? |
| 4 | Application permissions | What can the Entra application access? (Microsoft Graph, Azure resources) |

**Common misconfiguration:** Federated credential accepts any subject from the trusted issuer — a compromised low-privilege workload in AWS/GCP can access Azure resources.

---

## Cross-Cloud Credential Chains

When you find credentials from one cloud provider, test if they can be used to pivot to another.

### Chain: AWS → GCP

```
1. Find AWS credentials (SSRF → metadata, leaked keys, Cognito pool)
2. Check if AWS account has workload identity federation to GCP
3. Use AWS credentials to get a federated token for GCP
4. Check GCP service account permissions — storage, compute, BigQuery, etc.
```

### Chain: GCP → AWS

```
1. Find GCP service account token (metadata, leaked JSON key)
2. Check if GCP project trusts AWS via OIDC provider
3. Use GCP token to assume AWS role via STS AssumeRoleWithWebIdentity
4. Check AWS role permissions — S3, Secrets Manager, EC2, etc.
```

### Chain: Azure → AWS/GCP

```
1. Find Azure managed identity token (SSRF → IMDS, app config)
2. Check if Azure app has federated credentials for AWS/GCP
3. Use Azure token to get AWS STS token or GCP access token
4. Enumerate cross-cloud permissions
```

### Chain: On-Prem → Cloud

```
1. Find on-prem service credentials (Active Directory, LDAP, service accounts)
2. Check for AD Connect / Entra Connect syncing identities to Azure
3. Synced identities may have cloud permissions (Azure RBAC, conditional access gaps)
4. Test if on-prem credentials grant access to cloud resources
```

**Severity escalation:** A single-cloud credential finding is typically High. A cross-cloud pivot demonstrating access to resources in a second provider is Critical — it shows blast radius beyond the initial compromise.

---

## Federation Trust Abuse Patterns

| Pattern | Description | How to Test |
|---------|-------------|-------------|
| **Overly broad audience** | OIDC trust accepts any `aud` claim, not just the intended application | Generate token with different audience, attempt AssumeRoleWithWebIdentity |
| **Missing subject restriction** | Trust accepts any identity from the provider, not just specific workloads | Use a different SA/role from the same provider to assume the target role |
| **Stale federation trust** | Trust points to decommissioned account/project — takeover potential | If you can recreate the trusted account/project, you inherit the trust |
| **Circular trust** | A→B→C→A creates a privilege escalation loop | Map all federation relationships, look for cycles |
| **Token scope escalation** | Federated token has broader scope than the source identity | Compare permissions of source identity vs. federated role |
| **Cross-tenant federation** | Azure tenant trusts external tenant for workload identity | External tenant compromise → internal Azure access |

---

## Multi-Cloud Secrets Sprawl

Targets using multiple clouds often have secrets for one provider stored in another. Finding these cross-cloud secrets enables lateral movement.

| Secret Location | What to Look For |
|-----------------|-----------------|
| AWS Secrets Manager | GCP service account JSON keys, Azure client secrets, Kubernetes kubeconfig |
| GCP Secret Manager | AWS access keys, Azure connection strings, cross-cloud API tokens |
| Azure Key Vault | AWS credentials, GCP keys, database connection strings for cross-cloud data stores |
| CI/CD pipelines | GitHub Actions/GitLab CI secrets containing credentials for multiple clouds |
| Terraform state | All cloud credentials in plaintext — S3/GCS/Azure Blob state files |
| Kubernetes secrets | Cross-cloud credentials mounted as secrets in pods |
| `.env` files | Multi-cloud API keys in environment files committed to repos |

**Test procedure:**
1. When you find access to a secrets store, search for cross-cloud credential patterns
2. AWS keys: `AKIA*`, `aws_access_key_id`, `aws_secret_access_key`
3. GCP keys: `"type": "service_account"`, `"private_key_id"`
4. Azure: `"clientSecret"`, `"tenant"`, `"AZURE_CLIENT_SECRET"`
5. Test each cross-cloud credential — what can it access?

---

## Cloud-to-On-Prem Pivots

Cloud environments often have network connectivity or credential access back to on-premise infrastructure.

| Pivot Path | How to Test |
|------------|-------------|
| **VPN/VPC peering** | Cloud VPC peered with on-prem network — SSRF or RCE in cloud can reach internal services |
| **Direct Connect / ExpressRoute** | Dedicated network link — cloud compromise → internal network access |
| **Active Directory sync** | Entra Connect syncs on-prem AD to Azure — compromised Azure identity may have on-prem AD permissions |
| **Hybrid identity** | Same credentials work both cloud and on-prem — password spray on cloud → access on-prem |
| **Cloud-managed VPN appliances** | VPN gateways with management interfaces accessible from cloud — config leak, credential extraction |
| **Backup/DR replication** | Cloud-based disaster recovery copies of on-prem data — easier to access than the original |

---

## Decision Logic for Multi-Cloud Testing

| Signal | Action |
|--------|--------|
| Target uses multiple cloud providers (found in recon) | **INVEST** — check for federation trusts between them |
| Found credentials for one cloud | **TEST** — can they be used to pivot to another? |
| Federation trust exists but properly scoped (specific sub, aud) | **LOG + MOVE ON** — note the trust, but it's configured correctly |
| Federation trust with missing subject/audience checks | **CRITICAL** — demonstrate cross-cloud access |
| Cross-cloud secrets found in secrets manager | **INVEST** — test what each credential can access |
| Terraform state in public bucket | **CRITICAL** — contains all cross-cloud credentials |
| CI/CD pipeline with multi-cloud deploy permissions | **INVEST** — pipeline compromise = multi-cloud access |

**Severity guidance:** Cross-cloud lateral movement findings are almost always High-Critical because they demonstrate blast radius beyond a single provider. Always quantify: "Compromised AWS role X can also access GCP project Y containing Z."
