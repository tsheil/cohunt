# Authentication Mechanism Testing Patterns

> Referenced from [auth-testing SKILL.md](../SKILL.md) — detailed checklists for OAuth, JWT, session management, MFA, password reset, SSO/SAML, and rate limiting bypass.

## Table of Contents

- [OAuth & OpenID Connect](#oauth--openid-connect)
- [JWT (JSON Web Tokens)](#jwt-json-web-tokens)
- [Session Management](#session-management)
- [Multi-Factor Authentication (MFA) Bypass](#multi-factor-authentication-mfa-bypass)
- [Password Reset Poisoning](#password-reset-poisoning)
- [SSO & SAML](#sso--saml)
- [Rate Limiting & Brute Force Bypass](#rate-limiting--brute-force-bypass)

---

## OAuth & OpenID Connect

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

### OAuth Silent Redirect Abuse (March 2026)

A new technique documented by Microsoft Defender targeting government and public-sector orgs:

```
□ Silent error redirect
  Test: Add prompt=none&scope=invalid to authorization request
  Effect: Forces silent error redirect bypassing login page — phishing defenses don't trigger
  Test: Check if error redirect preserves authorization parameters
  Test: Check if error redirect URL can be attacker-controlled

□ First-party app trust abuse (ConsentFix)
  Test: Use first-party client IDs (Azure CLI, VS Code, Teams PowerShell) in OAuth flows
  Effect: Bypasses MFA and Conditional Access entirely — tokens obtained without 2FA
  Test: Use Azure CLI client ID to obtain tokens for other tenants
  Test: Check if admin consent restrictions apply to first-party apps
  Applies to: Microsoft Entra ID environments (most enterprises)
  Defense: Entra ID Token Protection with WAM broker

□ JWE-wrapped PlainJWT bypass (CVE-2026-29000, CVSS 10.0)
  Test: If target uses encrypted JWTs (JWE), wrap PlainJWT (alg=none) inside JWE
  Effect: Signature verification completely skipped — forge any user's token
  Applies to: pac4j-jwt library; any target using JWE with RSA public key
  Only requires knowledge of server's RSA public key (often publicly available)
```

> **Deep dive on OAuth/JWT format-level bugs:** See vuln-patterns [reference/web-vulns.md](../../vuln-patterns/reference/web-vulns.md)

---

## JWT (JSON Web Tokens)

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

□ Cross-application JWT acceptance
  Test: Obtain valid JWT from one application, present it to another that shares the same signing key
  Test: Parse Server pattern — if multiple apps share a signing key, JWT issued for App A authenticates as any user on App B
  Applies to: Parse Server < 8.6.10 / < 9.5.0-alpha.11; any multi-tenant JWT system with shared signing keys
  Impact: Full authentication bypass — impersonate any user on any co-hosted application
```

---

## Session Management

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

## Multi-Factor Authentication (MFA) Bypass

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

## Password Reset Poisoning

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

## SSO & SAML

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

## Rate Limiting & Brute Force Bypass

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

## Modern Identity Protocols

Enterprise SaaS targets increasingly use these protocols — each introduces distinct attack surfaces that are underexplored in bug bounty.

### Passkeys / WebAuthn / FIDO2

```
□ Authenticator attestation bypass — can you register a non-compliant authenticator?
□ Challenge replay — is the server challenge tied to a session and single-use?
□ User verification bypass — UV flag not enforced; authenticator claims UV but skips biometric
□ Cross-origin registration — can a subdomain or sibling domain register a passkey for the main domain?
□ Resident key enumeration — does the RP leak which usernames have passkeys registered?
□ Fallback chain weakness — passkey setup present, but password/SMS fallback still active? Test fallback-to-phishable
□ Credential ID leakage — are credential IDs exposed in public API responses? They're not secrets but enable targeted attacks
□ AAGUID-based filtering bypass — if RP restricts authenticator types, test with spoofed AAGUID
```

### SCIM Provisioning

```
□ Cross-tenant user creation — SCIM endpoint accepts user creation for a different tenant
□ SCIM token scope escalation — provisioning token grants access beyond intended tenant/group
□ Attribute injection — inject admin roles via SCIM user creation (e.g., roles: ["admin"])
□ Bulk operation abuse — SCIM bulk endpoint allows mass user modification without rate limiting
□ Deprovisioning race — delete user via SCIM while active session persists; test if sessions are revoked
□ Filter injection — SCIM filter parameter (e.g., filter=userName eq "x" or 1 eq 1) allows enumeration
□ Schema extension abuse — custom SCIM schema extensions that accept unexpected attributes
```

### Device Authorization / Device-Code Flow (RFC 8628)

```
□ Polling interval abuse — device polls faster than specified; server doesn't enforce slow_down
□ User code brute force — short user codes (6-8 chars) may be brutable during long polling windows
□ Device code reuse — can a device code be used after successful authorization for a second token?
□ Cross-client device code — authorize device code from one client_id, exchange from another
□ Phishing via device flow — attacker starts device flow, victim enters code; test if target validates device context
```

### DPoP (Demonstrating Proof of Possession — RFC 9449)

```
□ DPoP proof replay — is the jti (JWT ID) in DPoP proof checked for uniqueness?
□ Missing DPoP binding — access token issued without cnf claim; token works without proof
□ Algorithm downgrade — DPoP header accepts weak algorithms (none, HS256 with leaked key)
□ Key confusion — register DPoP with one key, present proof with another
□ Nonce bypass — server issues DPoP-Nonce but doesn't enforce it on subsequent requests
```

### Token Exchange (RFC 8693)

```
□ Subject token impersonation — exchange a low-privilege token for a higher-privilege one
□ Audience restriction bypass — requested audience not validated; get tokens for any service
□ Actor token escalation — exchange act_token to gain permissions beyond the actor's scope
□ Token type confusion — submit access_token as id_token type or vice versa
□ Cross-tenant exchange — exchange token from tenant A for a token valid in tenant B
```

### JIT (Just-in-Time) Provisioning

```
□ Attribute manipulation at JIT — IdP assertion sets admin role during first-login provisioning
□ Account takeover via JIT — existing account gets overwritten when JIT creates user with same email
□ Orphaned JIT accounts — user removed from IdP but JIT-provisioned account persists with active sessions
□ Group membership injection — JIT maps IdP groups to local roles; inject privileged group claims
```
