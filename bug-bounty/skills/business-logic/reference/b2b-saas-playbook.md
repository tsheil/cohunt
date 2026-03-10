# B2B SaaS Enterprise Workflow Playbook

Testing patterns for multi-tenant B2B SaaS platforms — the highest-value, lowest-competition attack surface in bug bounty. These workflows combine business logic with authorization and are systematically undertested by autonomous tools.

## Table of Contents

- [SCIM Provisioning](#scim-provisioning)
- [SSO / JIT Provisioning](#sso--jit-provisioning)
- [Invitation & Onboarding Flows](#invitation--onboarding-flows)
- [Role & Permission Sync](#role--permission-sync)
- [Seat & License Enforcement](#seat--license-enforcement)
- [Data Export / Import](#data-export--import)
- [Support & Impersonation](#support--impersonation)
- [Audit Log Integrity](#audit-log-integrity)
- [Approval Workflows](#approval-workflows)
- [Webhook Trust & Replay](#webhook-trust--replay)
- [Billing & Subscription Logic](#billing--subscription-logic)

---

## SCIM Provisioning

SCIM (System for Cross-domain Identity Management) syncs users between IdP and SaaS. Bugs here = cross-tenant access or unauthorized admin creation.

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Cross-tenant user creation | Send SCIM POST `/Users` with a different org's tenant ID in the path or body | User created in wrong tenant |
| 2 | Role escalation via SCIM | Set `roles` or `entitlements` in SCIM payload to admin values | Admin user provisioned via SCIM without IdP authorization |
| 3 | SCIM token scope bypass | Use SCIM bearer token from tenant A to create users in tenant B | Token not scoped to tenant |
| 4 | Deprovisioning race | Deprovision user via SCIM while they have active session | Session not invalidated — user retains access after removal |
| 5 | SCIM attribute injection | Include non-standard attributes (`isAdmin`, `isSuperUser`) in SCIM payload | App processes unexpected attributes |
| 6 | Group membership manipulation | Modify SCIM PATCH `/Groups` to add user to privileged group across tenant boundary | Cross-tenant group membership |
| 7 | SCIM filter injection | Inject filter operators in SCIM query parameters: `filter=userName eq "admin" or 1 eq 1` | Data from other tenants returned |

**Severity guidance:** Cross-tenant SCIM bugs are typically Critical. Same-tenant role escalation via SCIM is High.

---

## SSO / JIT Provisioning

Just-In-Time provisioning creates accounts on first SSO login. Bugs here create orphaned access or bypass tenant boundaries.

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | JIT into wrong tenant | SSO login with email domain matching a different tenant's configured domain | Account created in wrong org |
| 2 | Email domain takeover | Register a new tenant with a domain already claimed by another tenant | Tenant domain collision — JIT routes users to attacker tenant |
| 3 | IdP metadata swap | Replace SAML IdP metadata URL with attacker-controlled IdP | App trusts attacker's IdP for authentication |
| 4 | SSO bypass via direct login | After SSO enforcement, attempt password login at `/login` or legacy endpoints | SSO bypassed — password auth still works |
| 5 | Stale SSO session | Disable user in IdP, check if existing SaaS session remains valid | Session not revoked on IdP-side changes |
| 6 | JIT role assignment | Modify SAML assertion attributes (groups, roles) before app processes them | Elevated role on first login |
| 7 | Domain verification bypass | Claim ownership of email domain without DNS verification | JIT provisioning for domains you don't own |

**Severity guidance:** SSO bypass = High-Critical. JIT cross-tenant = Critical.

---

## Invitation & Onboarding Flows

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Invite role escalation | Accept invite, modify role parameter in acceptance request | Joined with higher permissions than invited |
| 2 | Invite to wrong org | Accept invite token but change `org_id` in acceptance payload | Added to different organization |
| 3 | Expired invite reuse | Use invite link after expiration or after inviter revokes it | Stale invite still works |
| 4 | Invite enumeration | Iterate invite tokens (if sequential or predictable) | Discover and accept invites meant for others |
| 5 | Multi-accept | Accept same invite from multiple email addresses/accounts | Multiple accounts get access from single-seat invite |
| 6 | Invite privilege persistence | Accept invite as admin, get downgraded, check if admin API access persists | Permissions cached from initial invite |
| 7 | Self-invite via API | Call invite API endpoint to invite yourself to orgs you don't belong to | Unauthorized org access |
| 8 | Invite email injection | Include CRLF or additional headers in invite email field | Email sent with injected content/recipients |

---

## Role & Permission Sync

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Role downgrade delay | Admin removes user's role — test if cached permissions still work for API calls | Authorization checked against stale cache |
| 2 | Custom role privilege creep | Create custom role, add sensitive permissions, assign to low-privilege context | Custom role bypasses permission hierarchy |
| 3 | Permission inheritance gaps | Parent org admin modifies child org — test if permission inheritance is enforced | Child org permissions bypass parent restrictions |
| 4 | Role sync race condition | Change role and make privileged API call simultaneously | Request processed with old (higher) role |
| 5 | API vs UI permission mismatch | Action blocked in UI — try same action via API directly | UI-only enforcement, API unprotected |

---

## Seat & License Enforcement

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Exceed seat limit | Invite more users than seat count allows via API (bypass UI validation) | Users added beyond license limit |
| 2 | Shadow seats | Deactivate user in UI, check if they can still authenticate via API/SSO | Deactivated users retain access |
| 3 | Seat type confusion | Modify user type from `viewer` (free seat) to `editor` (paid seat) via API | Feature access without corresponding license |
| 4 | Grace period abuse | Let subscription lapse, continue using during grace period, re-cancel before charge | Infinite grace period loop |
| 5 | Seat transfer bypass | Transfer seat assignment without admin approval | Unauthorized seat reassignment |

---

## Data Export / Import

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Export IDOR | Change `org_id` or `workspace_id` in export request | Download another tenant's data export |
| 2 | Import cross-tenant | Import data file with references to other tenant's resource IDs | Cross-tenant data injection |
| 3 | Export scope bypass | Request export of data types not included in your plan tier | Premium data exported on free tier |
| 4 | Bulk export without audit | Export all user data — check if audit log captures the export | Data exfiltration not logged |
| 5 | Import formula injection | Import CSV/XLSX with `=HYPERLINK()` or `=CMD()` formulas | Formula execution on admin's export review |
| 6 | Export timing attack | Request export of large dataset, monitor for incremental file access | Partial export files accessible before completion |

---

## Support & Impersonation

Support impersonation features ("login as customer") are high-value targets — a bug here = full account takeover at scale.

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Impersonation scope escape | As support, impersonate user, then access resources outside that user's org | Support session crosses tenant boundary |
| 2 | Impersonation persistence | After impersonation ends, check if support retains user's session tokens | Persistent unauthorized access |
| 3 | Self-impersonation escalation | Impersonate an admin user when you're a support agent with lower privileges | Privilege escalation via impersonation |
| 4 | Impersonation without audit | Perform sensitive actions while impersonating — check audit logs | Actions not attributed to support agent |
| 5 | Customer-initiated impersonation | Find the impersonation API endpoint, call it as a regular user | Non-support users can impersonate others |
| 6 | Impersonation token reuse | Capture impersonation session token, use it after impersonation officially ends | Token not properly invalidated |

**Severity guidance:** Impersonation bugs are almost always Critical — they represent full ATO at scale.

---

## Audit Log Integrity

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Audit log bypass | Perform sensitive actions via alternative API paths or legacy endpoints | Actions not captured in audit log |
| 2 | Audit log tampering | Check if audit entries can be modified or deleted by admin | Logs are mutable — forensic integrity compromised |
| 3 | Audit log IDOR | View audit logs for a different tenant by changing org identifier | Cross-tenant audit log access |
| 4 | Audit log injection | Include control characters or HTML in action fields | Log injection or stored XSS in audit viewer |
| 5 | Audit log completeness | Compare actions taken via API vs what appears in audit | Missing event types in audit coverage |

---

## Approval Workflows

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Self-approval | Submit request, modify approver ID to your own user ID | Bypass separation of duties |
| 2 | Approval bypass via API | Skip approval step by calling execution endpoint directly | Action performed without approval |
| 3 | Post-approval modification | Get approval, then modify the request payload before execution | Approved request tampered — different action executed |
| 4 | Approval replay | Replay an old approval response for a new request | Stale approval reused |
| 5 | Parallel approval race | Submit request to two approvers simultaneously | Both approve, doubling the authorized action |
| 6 | Approval chain skip | In multi-level approval (L1 → L2 → L3), call L3 endpoint directly | Intermediate approvals bypassed |

---

## Webhook Trust & Replay

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Webhook signature bypass | Remove or modify `X-Signature` header on webhook delivery | App processes webhook without signature verification |
| 2 | Webhook replay | Capture legitimate webhook, replay it hours/days later | No timestamp validation — replay accepted |
| 3 | Webhook SSRF | Set webhook URL to internal IP (127.0.0.1, 169.254.169.254, RFC 1918 ranges) | Internal service or cloud metadata accessed |
| 4 | Webhook cross-tenant | Configure webhook that receives events from another tenant | Cross-tenant data via event system |
| 5 | Webhook event injection | Forge webhook event payload to trigger actions (e.g., fake payment confirmation) | Business action triggered by forged event |
| 6 | Webhook URL redirect | Set webhook URL that 302-redirects to internal service | Redirect chain bypasses SSRF filter |

---

## Billing & Subscription Logic

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Plan confusion | Modify `plan_id` in upgrade request to premium plan but with free plan's price | Premium at free-tier pricing |
| 2 | Billing address tax bypass | Change billing country mid-checkout to avoid tax jurisdiction | Tax removed from invoice |
| 3 | Proration abuse | Upgrade mid-cycle, get prorated credit, downgrade immediately, repeat | Credit accumulation via upgrade/downgrade cycling |
| 4 | Free trial stacking | Create trials with email variants or modified payment tokens | Unlimited free trials |
| 5 | Invoice IDOR | Change invoice ID in download request | Download another customer's invoice |
| 6 | Payment method swap | Add payment method, use it for purchase, remove before charge processes | Purchase without payment |
| 7 | Discount code reuse | Apply single-use discount, cancel order, reuse code | Infinite discount reuse |
| 8 | Negative quantity billing | Submit negative quantity or negative addon in billing API | Credit generated instead of charge |

---

## Testing Methodology Summary

For each B2B SaaS target, systematically work through:

```
1. Map all admin/settings endpoints (org settings, SSO config, SCIM, billing)
2. For each endpoint, test:
   a. Cross-tenant access (change org_id/tenant_id)
   b. Role escalation (modify role/permission params)
   c. State machine violations (skip steps, replay, parallel execution)
   d. Enforcement gaps (UI vs API, cached vs live permissions)
3. Prioritize by impact:
   - Cross-tenant data access → Critical
   - Admin creation/impersonation → Critical
   - Billing/payment bypass → High-Critical
   - Seat/license enforcement → Medium-High
   - Audit log gaps → Medium (but valued by compliance-heavy programs)
```
