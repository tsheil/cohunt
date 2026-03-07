# Advanced Web Vulnerability Patterns

Specialized testing patterns for race conditions, deserialization, prototype pollution, GraphQL, JWT, OAuth, and API rate limiting. This is the reference file for the vuln-patterns skill — load it when targeting these specific vulnerability classes.

## Contents
- Race Conditions
- Deserialization
- Prototype Pollution
- GraphQL
- JWT (JSON Web Token)
- OAuth 2.0 / OpenID Connect
- API Rate Limiting & Resource Exhaustion

---

## Race Conditions

**What it is:** Exploiting timing between check and use of a resource to achieve unintended behavior.

**Where to look:**
- Payment/checkout flows
- Coupon/gift card redemption
- Account balance operations
- Invitation/join workflows
- Rate-limited actions
- File upload processing

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | TOCTOU (time of check to time of use) | Send parallel requests that modify the same resource | Double-spend, duplicate rewards |
| 2 | Limit bypass | Send N parallel requests to a rate-limited endpoint (e.g., coupon use) | More successful uses than allowed |
| 3 | Balance manipulation | Concurrent withdrawal/transfer requests | Balance goes negative or doubles |
| 4 | Invite race | Accept same invite from two sessions simultaneously | Duplicated permissions or roles |
| 5 | File overwrite | Upload two files concurrently to the same resource path | Unintended overwrite or data leak |
| 6 | Session race | Trigger password change + sensitive action in parallel | Action completes under old session |
| 7 | Last-byte sync | Use Turbo Intruder/HTTP/2 single-packet attack for precise timing | Sub-millisecond race windows exploitable |

**Tools:** Turbo Intruder (Burp extension), HTTP/2 single-packet attack technique, `race-the-web`.

---

## Deserialization

**What it is:** Exploiting unsafe deserialization of user-controlled data to achieve RCE, privilege escalation, or data manipulation.

**Where to look:**
- APIs accepting serialized objects (Java, PHP, Python pickle, .NET)
- Session tokens stored in cookies (especially Base64-encoded objects)
- WebSocket messages with serialized payloads
- Caching layers (Redis, Memcached) with serialized data
- Message queues (RabbitMQ, Kafka) with serialized payloads
- React Server Components Flight protocol (React2Shell, CVE-2025-55182)

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Java deserialization | Send ysoserial gadget chains via Java serialized data (magic bytes `AC ED 00 05`) | RCE, stack traces revealing deserialization |
| 2 | PHP deserialization | Inject crafted `O:` (object) payloads in PHP serialized fields | Arbitrary object instantiation, RCE |
| 3 | Python pickle | Send crafted pickle payloads to endpoints accepting pickled data | RCE via `__reduce__` method |
| 4 | JSON deserialization | Include `@type`, `$type`, or `class` fields in JSON to trigger polymorphic deserialization | Type confusion, RCE via gadget chains |
| 5 | .NET ViewState | Tamper with ViewState MAC validation; test for insecure deserialization | RCE via BinaryFormatter/ObjectStateFormatter |
| 6 | React Flight protocol | Test React Server Components for insecure deserialization in streaming responses | React2Shell (CVE-2025-55182, CVSS 10.0): pre-auth RCE |
| 7 | LLM framework serialization | Test LangChain/LlamaIndex streaming/serialization paths with untrusted metadata | LangGrinch pattern: prompt cascades through deserialization |

**Tools:** ysoserial (Java), PHPGGC (PHP), Freddy (Burp extension), SerializationDumper.

---

## Prototype Pollution

**What it is:** Modifying JavaScript object prototypes to affect application behavior across the entire runtime.

**Where to look:**
- APIs accepting deeply nested JSON objects
- Object merge/extend operations (lodash `merge`, `defaultsDeep`)
- GraphQL mutations with deeply nested inputs
- Configuration endpoints accepting arbitrary key-value pairs
- Any endpoint that recursively processes user-controlled objects

**Test patterns:**

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | Basic pollution | `{"__proto__":{"admin":true}}` | Privilege escalation, role bypass |
| 2 | Constructor pollution | `{"constructor":{"prototype":{"isAdmin":true}}}` | Bypasses `__proto__` filtering |
| 3 | Nested pollution | `{"a":{"__proto__":{"polluted":"yes"}}}` | Deep merge vulnerability |
| 4 | Array pollution | `{"__proto__":{"length":1000000}}` | DoS via resource exhaustion |
| 5 | Template pollution | `{"__proto__":{"outputFunctionName":"x;process.mainModule.require('child_process').execSync('id')//"}}` | RCE via EJS template engine |
| 6 | Server-side pollution → XSS | Pollute a rendering property used by the view engine | Reflected prototype pollution to XSS |

**Impact escalation:** Prototype pollution alone is often Medium, but chains into RCE (via template engines), XSS (via rendering libraries), or privilege escalation (via auth checks on polluted properties).

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
| 11 | JWE-wrapped PlainJWT bypass | If target uses encrypted JWTs (JWE), wrap a PlainJWT (alg=none) inside JWE using server's RSA public key | CVE-2026-29000 (pac4j-jwt, CVSS 10.0) |

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
