---
name: auth-testing
description: Tests authentication and authorization flaws — BOLA, BFLA, privilege escalation, IDOR, SSO bypass, MFA bypass, session management, and access control patterns. Activates when testing who-can-access-what or role boundaries. Triggers proactively on login, signup, password, session, cookie, token, OAuth, JWT, SAML, SSO, MFA, 2FA, role, permission, access control. Trigger on "test auth", "bypass login", "escalate privileges", "session fixation", "password reset", "account takeover", "broken access control", "BOLA", "BFLA", "IDOR access". For multi-tenant B2B SaaS workflows (SCIM, invites, seat enforcement, billing, shared links, connectors), use business-logic. For protocol-level API testing (GraphQL, gRPC, WebSocket), use api-security. For OAuth/JWT token format bugs, use vuln-patterns reference/web-vulns.md.
---

# Authentication & Authorization Testing

Auth bugs are the highest-paid vulnerability class in bug bounties. This skill covers the full spectrum: broken authentication, broken authorization (BOLA/BFLA), privilege escalation, and access control bypass.

## Quick Start — Two Accounts, Two Roles, Three Endpoints

1. **Set up** — Create 2 accounts at different privilege levels (free+paid, user+admin, tenant-A+tenant-B).
2. **Swap tokens** — Take Account A's auth token and call 3 endpoints that should be Account B-only.
3. **Check vertically** — Call admin/paid endpoints with the lower-privilege token.
4. **Check horizontally** — Access Account B's resources (IDs, files, settings) with Account A's token.
5. **Try bypasses** — Method switch (GET→PUT), API version (`/v1/` if `/v2/` blocked), parameter pollution (`?id=mine&id=theirs`).

If you have a hit, run `/reportability-check` before `/write-report`. If blocked or it looks patched, run `/variant-hunt`.

### Common Report Killers

| Mistake | Why It Gets N/A'd |
|---------|-------------------|
| Single-account proof | "I accessed my own data differently" — no trust boundary crossed |
| Soft-200 mistaken for auth bypass | Server returns 200 but with empty/error body — not a real bypass |
| Public data mistaken for IDOR | Data is intentionally public; check if it's truly behind auth |
| Cached role state | Browser cache showing old role data; clear cache and retest |
| UI block but API untested | UI hides button ≠ API enforced auth. Always test the API directly |

---

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
| **JWT (cross-app)** | Cross-application JWT acceptance | JWT from App A accepted by App B sharing signing key (Parse Server < 8.6.10) |
| **Async auth** | Missing `await` on async password validation (CVE-2026-28514, CVSS 9.3) | Rocket.Chat: un-awaited `bcrypt.compare()` returns Promise (truthy) → any password accepted; AI-discovered by GitHub Taskflow Agent |

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
| CVE-2025-25291/25292 | ruby-saml | Void canonicalization — parser differentials enable auth bypass (CVSS 9.1) |
| CVE-2025-66568/66567 | Multiple | Void canonicalization attack class — C14N comment handling bypass |
| CVE-2025-27773 | SimpleSAMLphp | Signature confusion via namespace manipulation |
| CVE-2025-29775/29774 | xml-crypto (SAMLStorm) | Multiple SignedInfo elements bypass signature without IdP key |
| CVE-2026-24858 | FortiNet SAML SSO | Cross-device login bypass via SAML assertion manipulation |
| CVE-2023-22515 | Atlassian Confluence | Broken access control → admin account creation |
| CVE-2026-1868 | GitLab AI Gateway | Auth bypass via JWT validation flaw |

> **Deep dive on void canonicalization and XML parser differentials:** See [parser-differentials.md](../vuln-patterns/reference/parser-differentials.md#void-canonicalization--new-attack-class-2025)

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

## 7b. Middleware Auth Bypass via URL Regex

Route-matcher authentication bypasses are a growing pattern — middleware skips auth when the URL matches a known "public" pattern (webhooks, health checks), but the regex operates on the full URL including query string.

**Key CVE:** CVE-2026-31816 (Budibase, CVSS 9.1) — appending `?/api/webhooks/` to any API endpoint bypassed all auth/CSRF.

```
Quick test for any authenticated API:
□ Append known public paths as query params: ?/api/webhooks/, ?/health, ?/public/
□ Append known bypass paths as fragments: #/api/webhooks/
□ Append known bypass paths as path params: ;/api/webhooks/
□ Test if auth middleware uses regex on full URL vs parsed path
```

**Cross-reference:** See parser-differentials.md for full route-matcher regex bypass patterns.

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

## WordPress/CMS Plugin Authorization Bypass

WordPress plugins are a top bug bounty target. The most common pattern: AJAX handlers registered via `wp_ajax_nopriv_*` missing `current_user_can()` checks, allowing unauthenticated access to admin functionality.

**Key CVE:** CVE-2026-2371 (Greenshift plugin <= 12.8.3, 100K+ installs) — `gspb_el_reusable_load()` AJAX handler accepts arbitrary `post_id`, renders private/draft reusable blocks without checking `current_user_can('read_post')`. Nonce exposed on public pages via `ajax="1"` shortcode.

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Enumerate AJAX handlers | `grep -r 'wp_ajax_nopriv_' wp-content/plugins/` | Handlers accessible without login |
| 2 | Missing capability check | Call AJAX handler with `action=handler_name` — no cookies | Data returned without auth (private posts, settings, user data) |
| 3 | Nonce bypass | Check if nonce is exposed on public pages or predictable | AJAX calls succeed with leaked/missing nonce |
| 4 | Post status leak | Request private/draft/password-protected content IDs | Content rendered regardless of visibility status |

**Applies to:** WordPress, Drupal (route access callbacks), Joomla (component ACL), any CMS with plugin-registered endpoints. **Severity:** Medium-High (private content disclosure) to Critical (admin functionality exposure).

---

## Salesforce Experience Cloud / Aura Guest User Access Control

Salesforce Experience Cloud sites expose the `/s/sfsites/aura` endpoint for the Lightning Aura framework. When guest user profiles are misconfigured with overly broad permissions, unauthenticated users can query CRM objects (Accounts, Contacts, Cases, Documents) directly. **ShinyHunters (UNC6240) breached ~400 organizations** in March 2026 using this pattern — no CVE assigned (Salesforce classifies as customer misconfiguration).

**Discovery:** Send empty POST `{}` to `/s/sfsites/aura` — `aura:invalidSession` response confirms Aura instance. Check custom path prefixes: `/business/s/sfsites/aura`, `/support/s/sfsites/aura`.

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Object enumeration | POST `getConfigData` action to Aura endpoint | `apiNamesToKeyPrefixes` reveals all accessible objects; `__c` suffix = custom objects |
| 2 | Unauthenticated data access | POST `getItems` with `entityNameOrId` for each discovered object | Response >12KB indicates data exposure; records returned without auth |
| 3 | Custom Apex controller | Search proxy history for `compound://c` and `Apex@` strings | Undocumented business logic accessible to guest users |
| 4 | Record limit bypass | Test `sortBy` parameter against 2,000-record GraphQL limit | Bypass returns full dataset beyond intended limit |
| 5 | Self-registration escalation | Register via guest self-registration if enabled | Elevated access beyond guest-tier permissions |

**Severity:** Critical when PII or CRM data exposed (Salesforce paid $6.2M in bounties in 2025, up to $60K/finding). **Tools:** AuraInspector (Mandiant), Misconfig Mapper (Intigriti), poc_salesforce_lightning. **Key misconfiguration:** "API Enabled" permission on guest user profile opens the attack vector — disabling it closes it.

---

## 9. Credential-Based Attacks — "Log In, Don't Break In"

**Cloudflare 2026 data:** 63% of all logins involve previously compromised credentials; 94% of login attempts originate from bots. Attackers are shifting from exploitation to credential abuse — and many applications lack adequate defenses.

### Testing Checklist

```
□ Credential stuffing resistance — High-volume login with known-breached creds detected/blocked?
□ Breached password detection — Registration/password-change checks against HIBP or similar?
□ Bot detection bypass — Login endpoint works without browser headers, JS execution, or CAPTCHA?
□ Rate limiting scope — Per IP only (bypassable via rotation) or also per username/account?
□ Account lockout info leak — Does lockout reveal valid usernames (different response for valid vs invalid)?
□ Credential enumeration — Different error messages for "user not found" vs "wrong password"?
□ MFA enforcement gaps — MFA on web but not API/mobile/SSO/SAML paths?
□ Session token entropy — Sequential or predictable patterns in session tokens?
□ Password spray window — Can you try 1 password across 1000 accounts without triggering lockout?
□ OAuth/social auth bypass — Can you authenticate via social login even if password MFA is enforced?
```

**Severity Guidance:** Credential stuffing → mass ATO is Critical. Missing breached-password detection is Medium. Credential enumeration alone is Low-Medium but chains with stuffing for higher impact. Password spray bypassing lockout is High.

---

## Reference Files

This skill uses progressive disclosure. Detailed reference material is available on demand:

| File | Contents | Lines |
|------|----------|-------|
| [reference/auth-mechanisms.md](reference/auth-mechanisms.md) | OAuth/OIDC attacks (silent redirect, ConsentFix), JWT flaws (JWE-PlainJWT CVE-2026-29000), session management, MFA bypass, password reset poisoning, SSO/SAML, rate limiting, modern identity (Passkeys, SCIM, DPoP) | ~353 |

**Quick search** — find specific auth mechanism patterns:
```
grep -n "OAuth\|redirect_uri\|ConsentFix\|silent redirect" ${CLAUDE_SKILL_DIR}/reference/auth-mechanisms.md
grep -n "JWT\|JWE\|algorithm\|PlainJWT\|CVE-2026-29000" ${CLAUDE_SKILL_DIR}/reference/auth-mechanisms.md
grep -n "session\|cookie\|fixation\|hijack" ${CLAUDE_SKILL_DIR}/reference/auth-mechanisms.md
grep -n "MFA\|2FA\|TOTP\|bypass" ${CLAUDE_SKILL_DIR}/reference/auth-mechanisms.md
grep -n "SAML\|SSO\|SCIM\|Passkey\|WebAuthn\|DPoP" ${CLAUDE_SKILL_DIR}/reference/auth-mechanisms.md
```
