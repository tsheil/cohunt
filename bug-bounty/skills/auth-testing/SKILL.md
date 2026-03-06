---
name: auth-testing
description: Test authentication and authorization — OAuth, JWT, session management, IDOR, privilege escalation, BOLA, BFLA, SSO bypass, MFA bypass, and access control patterns that consistently earn top payouts.
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

## 3. OAuth & OpenID Connect

### Common Misconfigurations

```
□ Open redirect in redirect_uri → steal authorization code
  Test: redirect_uri=https://attacker.com
  Test: redirect_uri=https://legit.com@attacker.com
  Test: redirect_uri=https://legit.com/.attacker.com
  Test: redirect_uri=https://legit.com%0d%0a.attacker.com
  Test: redirect_uri=https://legit.com/callback/../../../attacker

□ Insufficient redirect_uri validation
  Test: subdomain — redirect_uri=https://sub.legit.com (do you control any sub?)
  Test: path traversal — redirect_uri=https://legit.com/callback/../../
  Test: fragment — redirect_uri=https://legit.com/callback#@attacker.com
  Test: parameter pollution — redirect_uri=a&redirect_uri=https://attacker.com

□ Missing state parameter → CSRF on OAuth flow
  Test: Remove state entirely
  Test: Reuse expired state tokens
  Test: Use state from different session

□ Authorization code reuse → replay attack
  Test: Use code twice — second use should fail AND revoke tokens
  Test: Race condition — use code simultaneously in two requests

□ Token leakage via Referer
  Test: After callback, click external link — check Referer header for tokens
  Test: response_type=token with open redirect → token in URL fragment

□ Scope escalation
  Test: Request elevated scopes after initial consent
  Test: Modify scope parameter in authorization request
  Test: Token with scope A can access scope B endpoints?

□ Client secret exposure
  Test: Check JS bundles, mobile apps, public repos for client_secret
  Test: PKCE flow — is code_verifier actually validated?

□ Account linking flaws
  Test: Link attacker's OAuth to victim's account
  Test: Race condition in linking flow
  Test: Missing email verification in OAuth account creation
```

### OAuth Account Takeover Chain

The classic high-payout pattern:
1. Find open redirect or lax redirect_uri validation
2. Craft authorization URL that leaks code/token to attacker
3. Exchange stolen code for access token
4. Access victim's account

---

## 4. JWT (JSON Web Tokens)

### Implementation Flaws

```
□ Algorithm confusion
  Test: Change alg from RS256 to HS256 — sign with public key as HMAC secret
  Test: Change alg to "none" (with and without signature)
  Test: Change alg to "None", "NONE", "nOnE" (case variations)

□ Missing signature verification
  Test: Modify payload without updating signature
  Test: Remove signature entirely (keep the trailing dot)
  Test: Use empty string as signature

□ Weak signing key
  Test: Brute-force HMAC secret with common passwords
  Test: Try "secret", "password", app name, company name
  Tools: jwt_tool, hashcat with jwt mode

□ Key ID (kid) injection
  Test: kid=../../dev/null (sign with empty key)
  Test: kid=path/to/known/file (sign with known content)
  Test: SQL injection in kid parameter
  Test: kid pointing to URL you control

□ JWK/JWKS injection
  Test: Add jwk header with your own public key
  Test: jku header pointing to your JWKS endpoint
  Test: x5u header pointing to your certificate

□ Claim manipulation
  Test: Change sub (subject) to another user
  Test: Change role/permissions claims
  Test: Extend exp (expiration) far into the future
  Test: Modify iss (issuer) to bypass validation
  Test: Add admin: true or role: admin

□ Token lifecycle
  Test: Tokens valid after password change?
  Test: Tokens valid after logout?
  Test: Tokens valid after account deletion?
  Test: Refresh token rotation — old refresh tokens still work?
```

---

## 5. Session Management

### Testing Checklist

```
□ Session fixation
  Test: Set session cookie before authentication — does it persist after login?
  Test: Session ID in URL parameter — can you force a session?

□ Session invalidation
  Test: Logout — is server-side session actually destroyed?
  Test: Password change — are all other sessions terminated?
  Test: Role change — is session updated immediately?
  Test: Account deactivation — existing sessions still work?

□ Cookie security
  Test: Missing Secure flag → session cookie sent over HTTP
  Test: Missing HttpOnly flag → accessible via JavaScript (XSS → session theft)
  Test: Missing SameSite → CSRF via cookie
  Test: Overly broad Domain → cookie sent to subdomains
  Test: Overly broad Path → cookie accessible from unintended paths

□ Concurrent session limits
  Test: Can you have unlimited active sessions?
  Test: New login doesn't invalidate old sessions?
  Test: Session limit bypass via different user agents

□ Session token entropy
  Test: Are session tokens predictable? (sequential, timestamp-based)
  Test: Collect multiple tokens — check for patterns
  Test: Short tokens — brute-forceable?
```

---

## 6. Multi-Factor Authentication (MFA) Bypass

### Bypass Techniques

```
□ Direct endpoint access — skip MFA step, go directly to post-auth endpoint
□ Response manipulation — change {"mfa_required": true} to false in response
□ Backup code brute-force — often 8-digit numeric, may lack rate limiting
□ Race condition — submit MFA code multiple times simultaneously
□ Previously valid code — reuse old TOTP codes (clock skew tolerance too wide)
□ MFA on login but not on sensitive actions — password change, email change
□ MFA enrollment bypass — skip enrollment step, access account without MFA
□ Fallback mechanism — SMS fallback has weaker security than TOTP
□ API endpoints — MFA enforced on web but not on API/mobile endpoints
□ OAuth/SSO bypass — authenticate via SSO to skip MFA entirely
□ Remember device — token valid forever? Transferable between accounts?
□ Recovery flow — password reset bypasses MFA requirement
```

---

## 7. Password Reset Poisoning

### Attack Patterns

```
□ Host header poisoning
  Test: Host: attacker.com → reset link points to attacker.com
  Test: X-Forwarded-Host: attacker.com
  Test: X-Forwarded-For + Host manipulation

□ Token predictability
  Test: Reset tokens sequential or timestamp-based?
  Test: Token entropy — short enough to brute-force?
  Test: Same token generated for repeated requests?

□ Token lifecycle
  Test: Token valid after password is changed?
  Test: Token valid after second reset request?
  Test: Token expiration — how long is it valid?
  Test: Token valid after account settings change?

□ Flow bypasses
  Test: Skip email verification step — go directly to reset endpoint
  Test: Reset another user's password with your token
  Test: Rate limiting on reset endpoint — brute-force tokens
  Test: Manipulate email parameter — reset for different user but token sent to you
```

---

## 8. SSO & SAML

### Testing Checklist

```
□ SAML signature bypass
  Test: Remove signature entirely
  Test: Sign with your own key
  Test: Signature wrapping — move signed element, add unsigned copy
  Test: Comment injection in NameID — user<!---->.admin@company.com

□ SAML assertion manipulation
  Test: Modify NameID to another user
  Test: Modify role/group attributes
  Test: Replay expired assertions
  Test: Modify audience restriction
  Test: Inject additional assertions

□ SSO account linking
  Test: Link SSO identity to existing account without email verification
  Test: Race condition — link to account being created simultaneously
  Test: Case sensitivity — Admin@company.com vs admin@company.com

□ IdP confusion
  Test: Authenticate with wrong IdP for target tenant
  Test: Tenant confusion — access Tenant B resources with Tenant A SSO
  Test: IdP metadata injection — point to attacker's IdP
```

---

## 9. Privilege Escalation Patterns

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

## 10. Rate Limiting & Brute Force

### Bypass Techniques

```
□ IP rotation — X-Forwarded-For: random-ip on each request
□ Header manipulation — X-Real-IP, X-Originating-IP, X-Remote-IP
□ Case variation — admin@EXAMPLE.com vs admin@example.com
□ Padding — admin@example.com vs admin@example.com (with space/null bytes)
□ Parameter pollution — email=valid&email=target
□ JSON array — {"email": ["target@example.com"]}
□ Unicode normalization — homoglyph substitution
□ Endpoint variation — /login vs /Login vs /api/v1/login
□ HTTP method change — POST /login vs PUT /login
□ Null byte injection — admin%00@example.com
□ Race condition — send many requests simultaneously before counter updates
□ Distributed — rate limit per IP? Use multiple IPs
□ Long password DoS — send extremely long password to exhaust server CPU (bcrypt)
```

---

## Reporting Auth Bugs

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
