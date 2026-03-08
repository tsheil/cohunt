# Advanced Web Vulnerability Patterns

Extended testing patterns for complex web vulnerability classes — GraphQL, JWT manipulation, OAuth/OIDC flows, API rate limiting bypasses, and workflow automation sandbox escapes. These patterns complement the core vulnerability checklist in the main skill and target higher-severity, often-overlooked attack surfaces.

---

## GraphQL

**What it is:** A query language for APIs that introduces unique attack surface beyond REST.

**Where to look:**
- `/graphql`, `/api/graphql`, `/gql` endpoints
- Introspection queries for schema discovery
- Mutation operations for state changes
- Nested queries for authorization testing

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Introspection | `{__schema{types{name,fields{name}}}}` | Full schema disclosure |
| 2 | Field suggestion | Send typos in field names | Error messages leak valid fields |
| 3 | Batching abuse | Send array of operations `[{query:...},{query:...}]` | Bypass rate limiting |
| 4 | Nested query DoS | Deeply nested relationships `{users{posts{comments{author{posts{...}}}}}}` | Server resource exhaustion |
| 5 | Authorization on nodes | Query node IDs directly: `{node(id:"BASE64_ID"){... on User{email}}}` | Access other users' data |
| 6 | Mutation IDOR | Change IDs in mutation inputs | Modify other users' resources |
| 7 | Alias-based batching | `{a:login(u:"a",p:"1") b:login(u:"a",p:"2") ...}` | Brute force via aliases |
| 8 | Directive injection | `@include`, `@skip` with tautologies | Bypass field-level auth |
| 9 | Subscription abuse | Subscribe to events you shouldn't see | Information disclosure via WebSocket |
| 10 | Fragment injection | Overlapping fragments on union types | Access fields from wrong type |

**Bypasses:**
- If introspection is disabled, try `__type(name:"User"){fields{name}}` for individual types
- Some WAFs don't inspect POST body for GraphQL — try GET with `?query=` parameter
- Use field suggestion errors to enumerate schema without introspection
- Try sending queries as `application/x-www-form-urlencoded` instead of JSON

---

## JWT (JSON Web Token)

**What it is:** Stateless authentication tokens that can be manipulated if improperly validated.

**Where to look:**
- `Authorization: Bearer` headers
- Cookies containing base64-encoded JSON
- Any token with three dot-separated segments

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Algorithm none | Change header `alg` to `none`, remove signature | Token accepted without signature |
| 2 | Algorithm confusion | Change RS256 to HS256, sign with public key as HMAC secret | Forged token accepted |
| 3 | Weak secret | Brute-force HMAC secret (hashcat/jwt_tool) | Forge arbitrary tokens |
| 4 | Missing expiry | Check for `exp` claim | Token valid indefinitely |
| 5 | Expired token | Send expired token | Server doesn't check `exp` |
| 6 | Key ID injection | Inject `kid` header: `../../../dev/null` | Path traversal in key loading |
| 7 | JWK header injection | Embed attacker's JWK in token header | Server uses embedded key |
| 8 | Claim tampering | Change `sub`, `role`, `admin` claims | Privilege escalation |
| 9 | Token reuse | Use token after password change/logout | No token invalidation |
| 10 | Cross-service | Use token from service A on service B | Shared secret, no audience check |
| 11 | JWE-wrapped PlainJWT bypass | If target uses encrypted JWTs (JWE), wrap a PlainJWT (alg=none) inside JWE using server's RSA public key | Signature verification skipped — authenticate as any user including admin (CVE-2026-29000, pac4j-jwt, CVSS 10.0) |

**Tools:** jwt.io (decode), jwt_tool (attack), hashcat mode 16500 (crack)

---

## OAuth 2.0 / OpenID Connect

**What it is:** Authorization framework with complex redirect flows that are frequently misconfigured.

**Where to look:**
- Login flows ("Sign in with Google/GitHub/etc.")
- `/authorize`, `/callback`, `/oauth/token` endpoints
- `redirect_uri`, `state`, `code` parameters

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Open redirect via redirect_uri | Change `redirect_uri` to attacker-controlled domain | Auth code sent to attacker |
| 2 | Subdomain redirect | Set `redirect_uri` to `evil.legitimate.com` | Lax subdomain matching |
| 3 | Path traversal redirect | `redirect_uri=https://legit.com/callback/../evil` | Parser bypass |
| 4 | Missing state | Remove `state` parameter from flow | CSRF on OAuth login |
| 5 | State reuse | Replay a used `state` value | No single-use enforcement |
| 6 | Code reuse | Submit authorization code twice | No single-use enforcement |
| 7 | Scope escalation | Request `scope=admin` or add extra scopes | Elevated access granted |
| 8 | Token leakage | Check referrer headers after redirect | Token in URL fragment leaks |
| 9 | IdP confusion | Mix tokens between identity providers | Cross-IdP authentication bypass |
| 10 | PKCE downgrade | Remove `code_verifier` from token request | Server doesn't enforce PKCE |

**Bypasses for redirect_uri validation:**
- URL encoding: `%2F%2Fevil.com`
- Parameter pollution: `redirect_uri=legit.com&redirect_uri=evil.com`
- Fragment: `redirect_uri=legit.com%23@evil.com`
- Subdomain: `redirect_uri=legit.com.evil.com`
- Path: `redirect_uri=legit.com/callback/../../evil`

---

## API Rate Limiting & Resource Exhaustion

**What it is:** Abusing insufficient rate limiting or resource controls on API endpoints.

**Where to look:**
- Authentication endpoints (login, password reset, 2FA)
- Search and data export endpoints
- Any endpoint that triggers expensive operations

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | No rate limit | Rapid-fire requests to login endpoint | No blocking or throttling |
| 2 | Header bypass | Add `X-Forwarded-For: 127.0.0.1` to reset rate counter | Rate limit bypassed |
| 3 | Case variation | Alternate `user@email.COM` and `user@email.com` | Different rate limit buckets |
| 4 | Endpoint variation | Try `/api/v1/login` vs `/api/v2/login` | Per-path rate limiting |
| 5 | IP rotation bypass | Add varying `X-Originating-IP`, `X-Remote-IP` headers | Trust in client headers |
| 6 | Null byte bypass | Append `%00` to parameters | Different rate limit key |
| 7 | Regex DoS | Send crafted input to regex-heavy endpoints | Response time spike (ReDoS) |
| 8 | Pagination abuse | Request `?page_size=999999` | Full database dump |
| 9 | Parallel requests | Send concurrent requests before rate limit kicks in | Race window exploitation |
| 10 | API key enumeration | Iterate API keys on validation endpoint | No rate limit on key checking |

---

## n8n / Workflow Automation Sandbox Escapes

**What it is:** Exploiting sandbox bypass vulnerabilities in workflow automation platforms (n8n, Make, Zapier) that allow code execution beyond intended boundaries.

**Where to look:**
- Expression evaluation engines in workflow automation tools
- Code node implementations (JavaScript, Python)
- Merge/transform nodes with SQL capabilities
- Git integration nodes
- Any workflow tool allowing user-defined expressions

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | JavaScript `with` statement bypass | Use the deprecated `with` statement to access blocked constructors | Sandbox blocking `constructor.constructor` but allowing `with` statement access (CVE-2026-1470, CVSS 9.9) |
| 2 | TypeScript type confusion | Use destructuring syntax that TypeScript type annotations don't enforce at runtime | Sanitization bypassed because runtime types differ from declared types (CVE-2026-25049, CVSS 9.4) |
| 3 | Python code node escape | In workflow tools with Python execution nodes, test for arbitrary Python execution | Full Python execution on "Internal" configuration instances (CVE-2026-0863, CVSS 8.5) |
| 4 | SQL Query mode file write | In merge/transform nodes with SQL mode, test for arbitrary file system operations | File write outside intended directories via crafted SQL expressions (CVE-2026-25056, CVSS 9.4) |
| 5 | Git Node code injection | In workflow tools with Git integrations, inject code via Git operations | RCE on both self-hosted and cloud instances via Git Node (CVE-2026-21877) |
| 6 | Content-Type confusion | Send requests with unexpected Content-Type to webhook/trigger endpoints | Unauthenticated RCE via Content-Type confusion in form handling (CVE-2026-21858 Ni8mare, CVSS 10.0) |
| 7 | Public webhook + expression chain | Access public-facing webhook endpoint and inject expressions that escape the sandbox | Unauthenticated system command execution via single line of JS (CVE-2026-25049) |
| 8 | Destructuring-based sanitization bypass | Use ES6 destructuring to access blocked objects through reassignment | Bypass of `__proto__` and constructor blocks via destructuring syntax |

**Key principle:** Workflow automation platforms are high-value targets because: (a) they often have public webhook endpoints, (b) expression engines run user-controlled code in sandboxes with known bypass techniques, (c) n8n alone had 8 CVEs in January-February 2026 affecting ~100K instances.

**Tools:** n8n Community Edition (self-hosted for testing), Burp Suite for webhook fuzzing, custom JavaScript payloads for sandbox probing.
