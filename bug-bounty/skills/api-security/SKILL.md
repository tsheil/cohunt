---
name: api-security
description: API architecture security — design flaws in REST, GraphQL, WebSocket, and gRPC that create unique attack surfaces. Covers versioning gaps, batch/bulk abuse, schema introspection, subscription hijacking, and protocol-specific misconfigurations. Use when testing API design rather than individual vuln classes. Trigger when target recon reveals a REST API, GraphQL endpoint, WebSocket, or gRPC service; when multiple API versions are detected (v1, v2, internal); when user asks about batch endpoints, bulk operations, or rate limiting; or when API documentation or OpenAPI/Swagger spec is available. For auth bugs (BOLA, BFLA, access control), use auth-testing. For vuln class checklists (SSRF, IDOR, XSS), use vuln-patterns.
---

# API Security

Architecture-specific testing patterns for APIs. While `vuln-patterns` covers vulnerability classes (IDOR, SSRF, etc.) and `auth-testing` covers authentication/authorization, this skill focuses on how API design and protocol choices create unique attack surfaces.

> **Deep dive:** [reference/api-patterns.md](reference/api-patterns.md) for advanced GraphQL federation, mass assignment, JWT bypass, rate limit evasion, webhook SSRF, and microservice-specific patterns.

## API Threat Landscape (2025-2026)

APIs are now the primary digital attack surface. Key stats:
- **17% of all 2025 published security bulletins** (11,053 of 67,058) were API-related
- **43% of newly added CISA KEVs** in 2025 were API-related
- **Missing authentication** is the #1 API vulnerability (17% of incidents)
- **Broken authentication** present in 52% of API incidents
- **97% of API vulnerabilities** exploitable with a single request (42Crunch 2026)
- **Only 21% of organizations** can detect attacks at the API layer; only 13% can prevent >50% of API attacks (42Crunch 2026)
- **36% of AI vulnerabilities** also qualify as API vulnerabilities — AI and API attack surfaces overlap
- Companies shipping microservices and multi-cloud APIs faster than they can secure them

---

## API Discovery

Before testing, map the API surface:

### Finding API Endpoints

| Technique | How | What to look for |
|-----------|-----|-----------------|
| **Documentation** | `/docs`, `/swagger`, `/api-docs`, `/graphql`, `/.well-known/openapi` | Full endpoint inventory |
| **JavaScript analysis** | Inspect bundled JS for API calls | Fetch/XHR URLs, internal endpoints |
| **OpenAPI/Swagger** | `/swagger.json`, `/openapi.yaml`, `/v2/api-docs` (Spring) | Complete schema with parameters |
| **GraphQL introspection** | Send introspection query | Full type system and available operations |
| **API versioning** | Try `/api/v1/`, `/api/v2/`, `/api/internal/` | Deprecated or internal endpoints |
| **Mobile app** | Decompile APK/IPA, extract API URLs | Endpoints not exposed in web app |
| **Error responses** | Send malformed requests | Stack traces revealing routes |
| **robots.txt / sitemap** | Check for disallowed API paths | Hidden API endpoints |

### Version Discovery

```
Test these patterns for every known API endpoint:
/api/v1/users → /api/v2/users → /api/v3/users
/api/users?version=1 → /api/users?version=2
/v1/api/users → /v2/api/users
Accept: application/vnd.api+json;version=1 → version=2
X-API-Version: 1 → X-API-Version: 2

Why: Older versions often have weaker auth, fewer rate limits, and missing input validation.
```

---

## REST API Patterns

### API Versioning Abuse

The #1 underexplored API attack surface. Programs add new versions but leave old ones running.

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Downgrade to older version | Replace `/v3/` with `/v1/` or `/v2/` | Missing authorization, different behavior |
| 2 | Version header manipulation | Add `X-API-Version: 1` or `Accept-Version: 1` | Older behavior with weaker security |
| 3 | Internal version access | Try `/api/internal/`, `/api/private/`, `/api/admin/` | Unprotected internal endpoints |
| 4 | Deprecated endpoint access | Check if deprecated endpoints still respond | Active endpoints without current security |
| 5 | Version mismatch | Auth on v3, request on v1 | Token valid across versions without authz |

### Batch and Bulk Operations

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Batch endpoint abuse | Send array of IDs: `{"ids": [1,2,3,...100]}` | Mass data extraction bypassing per-request limits |
| 2 | Bulk create | Submit array of objects in single request | Bypass rate limits on creation |
| 3 | Mixed-auth batching | Include your resources + other users' in batch | Server processes all without per-item auth check |
| 4 | Export endpoints | `/api/users/export`, `/api/data/download` | Unrestricted bulk data access |
| 5 | Pagination abuse | Set `?limit=999999` or `?per_page=100000` | Server returns all records |

### Resource Manipulation

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Mass assignment | Send extra fields in POST/PUT: `{"role":"admin","verified":true}` | Server accepts and applies unexpected fields |
| 2 | Field filtering bypass | Add `?fields=password,secret` or `?include=sensitive` | Server returns fields not in default response |
| 3 | Sort/filter injection | `?sort=password` or `?filter[password][$gt]=a` | Sorting reveals data, filter operators leak info |
| 4 | Nested resource access | `/api/users/1/orders` → `/api/users/2/orders` | Cross-user access via nested routes |
| 5 | HTTP method override | `X-HTTP-Method-Override: DELETE` on GET request | Bypass method-based access controls |

### Rate Limiting Bypass

| # | Technique | How it works |
|---|-----------|-------------|
| 1 | IP rotation headers | `X-Forwarded-For: 1.2.3.4`, `X-Real-IP`, `X-Originating-IP` — rotate IPs |
| 2 | Case variation | `/Api/Users` vs `/api/users` — different rate limit buckets |
| 3 | Path normalization | `/api/users/` vs `/api/users` vs `/api//users` |
| 4 | Parameter pollution | `?id=1` vs `?id=1&dummy=1` — different cache/rate-limit keys |
| 5 | API version switching | Rate limited on v2? Try v1 |
| 6 | HTTP method switching | POST rate limited? Try PUT or PATCH |
| 7 | Encoding variations | URL encode path segments differently |
| 8 | Null byte injection | `/api/users%00` — may bypass path-based rate limiting |

---

## GraphQL Patterns

GraphQL's flexibility is its vulnerability. The type system, query depth, and batching create unique attack surfaces.

### Introspection and Schema Exposure

```
# Full introspection query
{__schema{types{name,fields{name,args{name,type{name}}}}}}

# If introspection is disabled, try:
- Suggestions/autocomplete: send partial field names, check error messages
- Field name bruteforce: common names like "user", "admin", "password", "secret"
- __type queries: {__type(name:"User"){fields{name}}}
- Apollo Studio/GraphiQL endpoints: /graphiql, /playground, /altair
```

### Query Abuse

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Deep nesting | `{user{friends{friends{friends{...}}}}}` | Server hangs or returns excessive data (DoS) |
| 2 | Alias-based batching | `{a:user(id:1){email} b:user(id:2){email} ...}` | Enumerate users in single request, bypass rate limits |
| 3 | Directive abuse | `@include`, `@skip`, `@deprecated` in queries | Bypass field-level authorization |
| 4 | Fragment abuse | Overlapping fragments requesting hidden fields | Access to fields not in normal schema |
| 5 | Batch queries | Send array of queries: `[{query:"..."},{query:"..."}]` | Bypass per-query rate limiting |
| 6 | Mutation side effects | Check if mutations perform actions regardless of return errors | Actions execute even when response indicates failure |
| 7 | Subscription abuse | Subscribe to events for other users' data | Real-time data exfiltration |

### Authorization Gaps

```
GraphQL-specific authorization issues:

1. Field-level vs query-level auth
   - Authenticated for the query but not all fields
   - Request: {user(id:1){name, ssn, salary}}
   - Auth may check user access but not field-level permissions

2. Mutation authorization mismatch
   - Query auth != mutation auth
   - Can query your own data but mutate others'

3. Nested resolver bypass
   - Top-level resolver checks auth, nested doesn't
   - {publicPost{author{privateEmail}}}

4. Interface/Union type confusion
   - Query a union type, get fields from a type you shouldn't access
   - {search(query:"test"){...on AdminUser{adminPanel}}}
```

---

## WebSocket Patterns

WebSocket connections maintain persistent state, creating unique testing opportunities.

### Connection-Level Issues

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Auth on connect only | Authenticate, then invalidate token/session | Messages still accepted after auth revoked |
| 2 | Cross-origin WebSocket | Connect from different origin | Missing origin validation |
| 3 | Protocol downgrade | Force `ws://` instead of `wss://` | Unencrypted WebSocket accepted |
| 4 | Connection hijacking | Steal WebSocket URL + ticket from another user | Access to their real-time data stream |

### Message-Level Issues

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Channel subscription | Subscribe to channels for other users/rooms | Cross-user data access |
| 2 | Message injection | Send crafted messages to server | XSS in other clients, command injection |
| 3 | Message tampering | Modify message IDs/types mid-stream | Bypass message-type authorization |
| 4 | Rate limit absence | Flood messages rapidly | No rate limiting on WebSocket (common) |
| 5 | State manipulation | Send out-of-order messages | Skip workflow steps, bypass validation |

---

## gRPC Patterns

gRPC uses Protocol Buffers and HTTP/2, creating a distinct attack surface.

### Discovery

```
- Check for gRPC reflection: grpcurl -plaintext target:port list
- Look for .proto files in source code or documentation
- Check error messages for protobuf field numbers
- Try gRPC-Web at standard HTTP endpoints (grpc-web proxy)
```

### Testing

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Reflection enabled | `grpcurl -plaintext host:port list` | Full service and method listing |
| 2 | Unknown field injection | Add extra fields in protobuf message | Server processes unexpected fields (mass assignment) |
| 3 | Type confusion | Send wrong types (string where int expected) | Error messages with internal details |
| 4 | Streaming abuse | Open server-streaming call, never close | Resource exhaustion |
| 5 | Metadata injection | Inject into gRPC metadata (headers) | Auth bypass, header injection |

---

## API Documentation Exploitation

When you find OpenAPI/Swagger, GraphQL schemas, or API docs:

### What to Extract

| From | Look for |
|------|----------|
| **OpenAPI/Swagger** | Admin endpoints, internal paths, deprecated routes, auth schemes, parameter types |
| **GraphQL schema** | Mutations you shouldn't have, admin types, hidden fields, deprecated directives |
| **API changelogs** | Recently removed features (still available?), security fixes (what was vuln?) |
| **Error responses** | Framework version, internal paths, database type, stack traces |
| **Response headers** | `X-Powered-By`, `Server`, rate limit headers, custom headers |

### Documentation as Attack Map

```
For every endpoint in docs:
1. Is auth required? → Test without auth
2. Is it marked deprecated? → Test with current auth
3. Does it accept IDs? → Test IDOR
4. Does it accept files? → Test upload attacks
5. Does it accept URLs? → Test SSRF
6. Does it have rate limits documented? → Test bypasses
7. Is there an admin version? → Test with user auth
```

---

## API-Specific Chaining

Common API vulnerability chains that elevate severity:

| Chain | Links | Impact |
|-------|-------|--------|
| **Info disclosure → targeted IDOR** | API leaks user IDs + IDOR on user endpoint | Mass data extraction with specific targets |
| **GraphQL introspection → admin mutation** | Schema reveals admin mutations + missing auth | Unauthorized admin actions |
| **Version downgrade → auth bypass** | Old API version + weaker auth checks | Access to protected resources |
| **Rate limit bypass → credential stuffing** | Bypass technique + login endpoint | Account takeover at scale |
| **Batch endpoint → mass IDOR** | Batch API + missing per-item auth | Extract all user data in single request |
| **WebSocket + IDOR** | Subscribe to other users' channels | Real-time data stream of other users |

---

## Framework-Specific Patterns

### Express.js / Node.js
- Prototype pollution via `__proto__` in JSON body
- Route parameter pollution with array values
- Path traversal in `express.static` with encoded dots

### Django REST Framework
- Filter backend injection: `?ordering=password`
- Viewset action authorization gaps
- Serializer field exposure through `?fields=` or `?expand=`

### Spring Boot
- Actuator endpoints: `/actuator/env`, `/actuator/heapdump`
- SpEL injection in error handling
- Request mapping ambiguity between controllers

### Laravel / PHP
- Debug mode: `/_ignition/execute-solution`
- Mass assignment through fillable/guarded misconfiguration
- Route model binding IDOR

### FastAPI / Python
- Auto-generated docs at `/docs` and `/redoc`
- Pydantic validation bypass with coerced types
- Dependency injection authorization gaps

### Ruby on Rails
- Strong parameters bypass through nested attributes
- `render` file disclosure
- ActiveRecord injection through `where` hash conditions

---

## Reference Files

This skill uses progressive disclosure. Detailed reference material is available on demand:

| File | Contents | Lines |
|------|----------|-------|
| [reference/api-patterns.md](reference/api-patterns.md) | GraphQL federation, mass assignment, JWT bypass, rate limit evasion, webhook SSRF, API gateway misconfigs, microservice patterns | ~283 |

**Quick search** — find specific patterns without loading the full file:
```
grep -n "GraphQL\|federation\|persisted query" reference/api-patterns.md
grep -n "mass assignment\|JSON API\|HATEOAS" reference/api-patterns.md
grep -n "JWT\|API key\|authentication bypass" reference/api-patterns.md
grep -n "rate limit\|distributed\|evasion" reference/api-patterns.md
grep -n "webhook\|callback\|SSRF" reference/api-patterns.md
grep -n "gateway\|Kong\|Cloudflare\|AWS API Gateway\|service mesh" reference/api-patterns.md
```

---

## Related Skills & Commands

- `vuln-patterns` — Individual vulnerability class patterns (IDOR, XSS, SSRF, etc.)
- `auth-testing` — Authentication and authorization testing in depth (includes middleware regex bypass patterns)
- `source-code-audit` — Trace API handlers in source code
- `ai-hunting/reference/mcp-playbooks.md` — MCP-specific API testing (gRPC transport, OAuth, tool invocation)
- `/quick-test` — Rapid single-endpoint testing
- `/methodology` — Full testing methodology including API-specific phases

**Note:** For MCP servers using gRPC transport (Google Cloud, January 2026), test JWT/OAuth authentication hooks and per-method authorization policies. gRPC MCP servers may have different trust boundaries than HTTP-based MCP servers — verify auth enforcement across both transports.
