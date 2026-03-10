---
name: auth-testing
description: Test authentication and authorization flaws — BOLA, BFLA, privilege escalation, IDOR, SSO bypass, MFA bypass, session management, and access control patterns. Use when testing who-can-access-what, role boundaries, or multi-tenant isolation. Use proactively whenever the user mentions login, signup, password, session, cookie, token, OAuth, JWT, SAML, SSO, MFA, 2FA, rate limiting, role, permission, access control, tenant, or any authentication/authorization concept. Trigger on "test auth", "bypass login", "escalate privileges", "session fixation", "password reset", "account takeover", "broken access control". For protocol-level API testing (GraphQL, gRPC, WebSocket), use api-security. For OAuth/JWT token format bugs, use vuln-patterns reference/web-vulns.md.
---

# Authentication & Authorization Testing

Auth bugs are the highest-paid vulnerability class in bug bounties. This skill covers the full spectrum: broken authentication, broken authorization (BOLA/BFLA), privilege escalation, and access control bypass.

## Quick Reference: What Pays

| Bug Class | Typical Severity | Avg Payout Range | Duplicate Risk |
|-----------|-----------------|------------------|----------------|
| Full account takeover | Critical | $5k–$50k+ | Medium |
| BOLA (IDOR on objects) | High–Critical | $2k–$15k | High |
| BFLA (function-level authz) | High–Critical | $3k–$20k | Medium |
| Privilege escalation | High–Critical | $3k–$25k | Medium |
| OAuth misconfiguration | Medium–Critical | $1k–$15k | Low–Medium |
| JWT implementation flaw | Medium–High | $1k–$10k | Medium |
| MFA bypass | High–Critical | $2k–$15k | Low |
| Session fixation/hijack | Medium–High | $500–$5k | Medium |
| SSO bypass | High–Critical | $3k–$20k | Low |
| Password reset poisoning | Medium–High | $500–$5k | Medium–High |

---

## 1. Broken Object-Level Authorization (BOLA / IDOR)

The most common high-severity bug class in API-heavy targets.

### Discovery Pattern

1. **Identify object references** — Any endpoint with IDs, UUIDs, slugs, or sequential identifiers
2. **Map access control boundaries** — What should User A see vs User B?
3. **Test cross-account access** — Swap identifiers between accounts
4. **Check indirect references** — Nested resources, related objects, file paths

### Testing Checklist

```
□ Sequential integer IDs → increment/decrement
□ UUIDs → swap between accounts (don't assume unguessable = secure)
□ Slugs/usernames → substitute another user's slug
□ Nested resources → /orgs/{orgA}/users/{userB} cross-tenant
□ Bulk/batch endpoints → include other users' IDs in arrays
□ Export/download endpoints → change resource ID in export URL
□ Webhook/callback URLs → reference other users' resources
□ GraphQL node queries → query(id: "other-user-id")
□ File/attachment references → swap file IDs or paths
□ API versioning → old API version may lack authz checks
```

### Bypasses When Basic Swap Fails

- **HTTP method switching** — GET blocked? Try PUT, PATCH, DELETE, HEAD
- **Parameter pollution** — `?id=mine&id=theirs` — which one wins?
- **Case manipulation** — `/Users/123` vs `/users/123`
- **Wrapping** — `{"id": 123}` vs `{"id": {"$eq": 123}}` (NoSQL)
- **Encoding** — URL-encode, double-encode, unicode normalization
- **Wildcard/glob** — `*`, `%`, `_` in ID fields
- **Array injection** — `id[]=123&id[]=456` may return both
- **Old API versions** — `/api/v1/users/123` when current is v3
- **GraphQL aliases** — Query same field with different IDs in one request
- **Numerical overflow** — Negative IDs, MAX_INT, zero

### Impact Escalation

Don't just read — demonstrate write/delete:
- Read another user's data (information disclosure)
- Modify another user's data (data tampering)
- Delete another user's resources (destructive)
- Act as another user (full impersonation)

---

## 2. Broken Function-Level Authorization (BFLA)

Higher severity than BOLA — accessing admin or privileged functions, not just objects.

### Discovery Pattern

1. **Find privileged endpoints** — Admin panels, management APIs, internal tools
2. **Identify role boundaries** — User vs admin, free vs paid, member vs owner
3. **Test vertical escalation** — Call admin endpoints with regular user tokens
4. **Check horizontal escalation** — Access other org's management functions

### Testing Checklist

```
□ Admin endpoints → call with regular user token
□ Role-specific functions → test each role against functions of higher roles
□ Hidden API routes → /api/admin/*, /api/internal/*, /api/debug/*
□ Management operations → user CRUD, billing, settings, invite
□ Feature flags → paid features accessible to free users
□ Bulk operations → mass update/delete with regular permissions
□ Import/export admin data → accessible to non-admins?
□ GraphQL mutations → admin mutations callable by users?
□ WebSocket admin channels → subscribe to privileged channels
□ Rate limit admin exemptions → admin endpoints lack rate limiting
```

### Where to Find Hidden Admin Endpoints

- JavaScript bundles — search for `/admin`, `/internal`, `/manage`
- Mobile app binaries — extract API routes from app code
- API documentation — Swagger/OpenAPI may expose admin routes
- Error messages — 403 responses confirm the endpoint exists
- Predictable patterns — if `/api/users` exists, try `/api/admin/users`
- Old documentation — cached docs may reference removed-but-still-active endpoints
- GraphQL introspection — query `__schema` for admin types and mutations

---

## 3. OAuth, JWT, Session, MFA, Password Reset, SSO

> **Full checklists:** See [reference/auth-mechanisms.md](reference/auth-mechanisms.md) for detailed test procedures for all authentication mechanism classes.

### Key Patterns (Quick Reference)

| Mechanism | Top Attack | Key Test |
|-----------|-----------|----------|
| **OAuth** | redirect_uri manipulation → code theft → ATO | Swap redirect_uri to attacker domain; test path traversal, subdomain, fragment bypasses |
| **OAuth (2026)** | Silent redirect abuse + first-party trust (ConsentFix) | `prompt=none&scope=invalid`; use Azure CLI client ID to bypass MFA |
| **JWE/JWT** | JWE-wrapped PlainJWT bypass (CVE-2026-29000, CVSS 10.0) | Wrap `alg=none` token inside JWE envelope → signature skipped |
| **JWT** | Algorithm confusion (RS256→HS256) | Sign with public key as HMAC secret; `alg=none` variations |
| **Session** | Fixation + missing invalidation | Pre-set session cookie → login → check persistence; verify logout destroys server session |
| **MFA** | Direct endpoint access + API bypass | Skip MFA step, hit post-auth endpoint directly; test API vs web enforcement |
| **Password Reset** | Host header poisoning → reset link hijack | `Host: attacker.com` or `X-Forwarded-Host`; check token predictability |
| **SSO/SAML** | Signature wrapping + NameID injection | Remove sig; comment injection `user<!---->.admin@company.com`; IdP confusion |
| **Rate Limiting** | IP rotation + parameter pollution | `X-Forwarded-For: random`; case/encoding variations; race conditions |
| **Passkeys/WebAuthn** | Fallback chain + UV bypass | Passkey registered but password fallback still active; UV flag not enforced |
| **SCIM** | Cross-tenant provisioning + attribute injection | SCIM creates users in wrong tenant; inject admin role via SCIM attributes |
| **Device-Code Flow** | User code brute force + phishing | Short codes brutable during long polling; attacker-initiated code + victim authorization |
| **DPoP** | Proof replay + missing binding | jti not checked; access token works without DPoP proof |
| **Token Exchange** | Subject impersonation + audience bypass | Exchange low-priv token for high-priv; audience not validated |

> **Deep dive on OAuth/JWT format-level bugs:** See vuln-patterns [reference/web-vulns.md](../vuln-patterns/reference/web-vulns.md)
> **Modern identity protocols (Passkeys, SCIM, DPoP, Token Exchange, JIT):** See [reference/auth-mechanisms.md](reference/auth-mechanisms.md#modern-identity-protocols)

---

## 4. Privilege Escalation Patterns

> For rate limiting bypass techniques, see [reference/auth-mechanisms.md](reference/auth-mechanisms.md#rate-limiting--brute-force-bypass).

### Vertical Escalation (User → Admin)

```
□ Role parameter manipulation — add role=admin to registration or profile update
□ Hidden fields — check if role/permission fields are in API but not UI
□ Mass assignment — send extra fields: {"name": "...", "is_admin": true}
□ GraphQL over-fetching — mutation may accept admin fields not shown in docs
□ Registration with admin email domain — auto-assigned admin role?
□ Invite flow — can you invite yourself as admin?
□ Default role after specific action — certain flows reset to higher role?
□ Admin JWT claim — add admin claims to token
```

### Horizontal Escalation (Tenant A → Tenant B)

```
□ Tenant ID in JWT — modify tenant/org claim
□ Subdomain confusion — access tenant-b.app.com resources from tenant-a session
□ Shared resources — cross-tenant access to shared storage, APIs, webhooks
□ Invitation links — accept invite for wrong tenant
□ API keys — tenant-scoped? Or global access with any key?
□ Webhook endpoints — receive events from other tenants
```

---

## 5. SAML & Enterprise SSO Attacks

Enterprise targets with SSO are high-value — SAML bypasses often mean access to entire corporate applications.

### SAML Attack Patterns

```
□ Signature removal — Strip signature; check if IdP response accepted unsigned
□ Signature wrapping — Move signed assertion, insert attacker assertion in unsigned position
□ NameID injection — comment injection: user<!---->admin@corp.com parsed differently by IdP vs SP
□ XML entity injection — XXE in SAML response parsing (CVE-2024-45409, Ruby SAML CVSS 10.0)
□ Assertion replay — Capture valid SAML assertion, replay after session expires
□ Audience restriction bypass — Modify Audience field to different SP
□ Recipient manipulation — Change AssertionConsumerServiceURL to attacker-controlled
□ InResponseTo bypass — Remove or modify InResponseTo validation
□ Condition time window — Manipulate NotBefore/NotOnOrAfter for expired assertions
```

### Real-World SAML CVEs

| CVE | Product | Impact |
|-----|---------|--------|
| CVE-2024-45409 | Ruby SAML | Signature wrapping → admin access (CVSS 10.0) |
| CVE-2025-25291/25292 | ruby-saml | Parser differentials enable auth bypass |
| CVE-2023-22515 | Atlassian Confluence | Broken access control → admin account creation |
| CVE-2026-1868 | GitLab AI Gateway | Auth bypass via JWT validation flaw |

---

## 6. API Key & Token Lifecycle Testing

Often overlooked — API key management is a growing attack surface, especially with AI/ML integrations proliferating API keys.

### Testing Checklist

```
□ Key rotation — Are old keys invalidated immediately on rotation?
□ Key scoping — Can a read-only key perform write operations?
□ Key revocation — Is the key usable after revocation? (test within seconds)
□ Key enumeration — Are API keys sequential or predictable?
□ Key leakage paths — Check error messages, logs, client-side JS, git history
□ Cross-environment keys — Do staging keys work in production?
□ Orphaned keys — Keys for deleted users/services still active?
□ Rate limiting per key — Or can one key make unlimited requests?
□ Key in URL — API key passed as query parameter (logged in server/proxy logs)?
□ Bearer vs API key confusion — Swapping auth mechanisms accepted?
```

### Where Keys Leak

- **Git history** — `git log --all -p -- '*.env'` or trufflehog/gitleaks scans
- **Client-side JavaScript** — Search bundles for `api_key`, `apiKey`, `Authorization`, `Bearer`
- **Error responses** — Verbose errors revealing internal keys or tokens
- **OpenClaw/MCP configs** — AI agent configurations often embed API keys in plaintext (CVE-2026-21852)
- **Public Postman collections** — Search Postman public API network for target's collections

---

## 7. Multi-Tenant Isolation Testing

Growing attack surface as SaaS platforms proliferate. IDOR/IAC has shown 116% 5-year growth and 29% YoY increase.

### Cross-Tenant Attack Patterns

```
□ Tenant ID in JWT claims — Modify org/tenant claim, replay token
□ Shared database queries — tenant_id parameter omitted → access all tenants
□ Subdomain confusion — tenant-b.app.com session used on tenant-a
□ Webhook cross-contamination — Receive events from other tenants
□ File storage isolation — S3/blob paths predictable across tenants?
□ Search index leakage — Elasticsearch/Algolia returning cross-tenant results
□ Invitation token reuse — Accept invite for wrong tenant
□ GraphQL federation — Query federated service without tenant context
□ Shared infrastructure — Redis/cache keys colliding across tenants
□ AI agent context isolation — Multi-tenant AI platforms leaking context between tenants (CVE-2026-30855 WeKnora CVSS 8.8)
□ Support impersonation escalation — Can support role access exceed intended scope? Test impersonate-as-admin paths
□ Service account scope drift — Service accounts with initial narrow scope accumulate permissions over time; test for over-privileged service tokens
```

### SCIM/JIT Provisioning Attacks

Growing attack surface as enterprises adopt automated identity provisioning:

```
□ SCIM endpoint auth — Is /scim/v2/Users protected? Many deployments expose unauthenticated SCIM endpoints
□ SCIM user creation — Can you provision an admin user via SCIM API? Test role/group assignment in SCIM payloads
□ SCIM attribute injection — Inject unexpected attributes (isAdmin, role, permissions) in SCIM user creation
□ JIT provisioning race — Create user via JIT before legitimate SAML assertion arrives; claim the identity
□ SCIM group manipulation — Add your user to admin groups via SCIM Groups endpoint
□ Deprovisioning bypass — After SCIM deprovisioning, can the user still authenticate via cached sessions or API keys?
□ SCIM token reuse — SCIM bearer tokens often long-lived; test if revoked IdP tokens still work for SCIM operations
```

### Invite Flow Exploitation

```
□ Invite to wrong org — Modify org_id/tenant_id in invite acceptance to join a different organization
□ Invite role escalation — Intercept invite, modify role field (viewer → admin) before acceptance
□ Expired invite reuse — Use invite link after expiration; test if tokens are time-validated server-side
□ Invite enumeration — Sequential invite tokens allow discovering valid invitations for other orgs
□ Multi-accept — Accept same invite from multiple accounts simultaneously (race condition)
□ Self-invite — Call invite endpoint to invite yourself with elevated permissions
□ Invite + deactivated user — Reactivate a deactivated user account by sending them a new invite
```

---

## 8. Reporting Auth Bugs

### Impact Framing

Auth bugs need clear impact statements. Use this structure:

**Account Takeover (ATO):**
> An attacker can take over any user's account by [mechanism]. This gives full access to [list sensitive data/actions]. Affected users have no indication their account has been compromised.

**Unauthorized Data Access (BOLA):**
> A low-privileged user can access [what data] belonging to other users by [mechanism]. This affects [estimated number] of [resource type] across the platform.

**Privilege Escalation:**
> A regular user can escalate to [role] privileges by [mechanism], gaining access to [what admin functions]. This bypasses all intended access controls.

### CVSS Scoring Guide for Auth Bugs

| Scenario | Attack Vector | Attack Complexity | Privileges Required | User Interaction | Scope | Typical CVSS |
|----------|--------------|-------------------|--------------------|--------------------|-------|-------------|
| Full ATO via OAuth | Network | Low | None | Required (click link) | Changed | 8.1–9.6 |
| BOLA read any user data | Network | Low | Low | None | Unchanged | 6.5–7.5 |
| BOLA read+write any user data | Network | Low | Low | None | Unchanged | 8.1–8.5 |
| Admin function access | Network | Low | Low | None | Unchanged | 8.1–8.8 |
| JWT algorithm confusion → ATO | Network | Low | None | None | Changed | 9.1–9.8 |
| MFA bypass → ATO | Network | Low | Low | None | Unchanged | 7.5–8.5 |
| Session fixation → ATO | Network | Low | None | Required | Changed | 7.1–8.1 |

### What Makes Reports Get Paid

- **Clear reproduction steps** — from zero to account takeover in numbered steps
- **Two test accounts** — demonstrate cross-account access with real evidence
- **Impact quantification** — "affects all N users" not "could theoretically affect users"
- **Business impact** — regulatory (GDPR, HIPAA), financial, reputational consequences
- **No theoretical attacks** — demonstrate actual exploitation, not just misconfiguration
- **Screenshots/recordings** — show the authorization check failing with evidence

---

## Reference Files

This skill uses progressive disclosure. Detailed reference material is available on demand:

| File | Contents | Lines |
|------|----------|-------|
| [reference/auth-mechanisms.md](reference/auth-mechanisms.md) | OAuth/OIDC attacks (silent redirect, ConsentFix), JWT flaws (JWE-PlainJWT CVE-2026-29000), session management, MFA bypass, password reset poisoning, SSO/SAML, rate limiting, modern identity (Passkeys, SCIM, DPoP) | ~353 |

**Quick search** — find specific auth mechanism patterns:
```
grep -n "OAuth\|redirect_uri\|ConsentFix\|silent redirect" reference/auth-mechanisms.md
grep -n "JWT\|JWE\|algorithm\|PlainJWT\|CVE-2026-29000" reference/auth-mechanisms.md
grep -n "session\|cookie\|fixation\|hijack" reference/auth-mechanisms.md
grep -n "MFA\|2FA\|TOTP\|bypass" reference/auth-mechanisms.md
grep -n "SAML\|SSO\|SCIM\|Passkey\|WebAuthn\|DPoP" reference/auth-mechanisms.md
```
