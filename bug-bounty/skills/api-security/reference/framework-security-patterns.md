# Framework-Specific API Security Patterns

CVE-grounded testing patterns for the most common API frameworks. Each section includes detection heuristics, high-value attack surface, concrete test procedures, and worked examples.

> **Related files:** [SKILL.md](../SKILL.md) for core API testing | [api-patterns.md](api-patterns.md) for advanced REST/GraphQL/webhook patterns | [realtime-protocols.md](realtime-protocols.md) for WebSocket/SSE/SignalR

## Table of Contents

- [Express.js / Node.js](#expressjs--nodejs)
- [Django REST Framework](#django-rest-framework)
- [Spring Boot](#spring-boot)
- [Laravel / PHP](#laravel--php)
- [FastAPI / Python](#fastapi--python)
- [Ruby on Rails](#ruby-on-rails)
- [Next.js / React Server Components](#nextjs--react-server-components)
- [Cross-Framework Patterns](#cross-framework-patterns)

---

## Express.js / Node.js

### Detection

| Signal | Confidence |
|--------|-----------|
| `X-Powered-By: Express` header | High |
| `connect.sid` cookie (express-session) | High |
| Stack traces mentioning `node_modules/express` | High |
| `ETag` format: `W/"xx-xxxxxxxxxx"` (weak ETags) | Medium |
| JSON error format: `{"error": "...", "stack": "..."}` | Medium |

### High-Value Attack Surface

**1. Prototype Pollution via Request Body**

Express parses JSON bodies and query strings. When applications spread or merge request data into objects without sanitization, `__proto__` or `constructor.prototype` payloads can modify object behavior globally.

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | JSON body `__proto__` | `{"__proto__": {"isAdmin": true}}` in POST/PUT body | Subsequent requests inherit `isAdmin` property |
| 2 | Query string pollution | `?__proto__[isAdmin]=true` or `?constructor[prototype][isAdmin]=true` | Server-side object has unexpected property |
| 3 | Nested merge pollution | `{"user": {"__proto__": {"role": "admin"}}}` | Deep merge spreads prototype into target |
| 4 | qs library array bypass | `?ids[]=1&ids[]=2` vs `?ids=1,2` — different parsing | Array vs string type confusion in handlers |

**CVE-2022-24999** (qs <6.10.3, Express <4.17.3): `__proto__` in query string causes process hang. Still common — many Express apps pin older qs versions.

**2. Direct Request Body to Database**

The most common Express API vulnerability pattern. When `req.body` is passed directly to database operations:

```
// VULNERABLE — attacker controls all fields
app.put('/api/users/:id', (req, res) => {
  User.findByIdAndUpdate(req.params.id, req.body);
});

// Test: PUT /api/users/123
// Body: {"role": "admin", "verified": true, "credits": 999999}
// If the response reflects the new role/verified/credits, it's mass assignment.
```

**3. Path Traversal in express.static**

`express.static` with encoded dots: `GET /assets/..%2F..%2F..%2Fetc%2Fpasswd` — if the server decodes before resolving, path traversal succeeds. Test double-encoding: `%252e%252e%252f`.

---

## Django REST Framework

### Detection

| Signal | Confidence |
|--------|-----------|
| `csrftoken` cookie + `X-CSRFToken` header | High |
| `/api/` with `?format=json` or `?format=api` (browsable API) | High |
| Error response: `{"detail": "..."}` format | High |
| `sessionid` cookie | Medium |
| `Allow` header listing HTTP methods | Medium |

### High-Value Attack Surface

**1. Filter Backend Injection**

DRF `OrderingFilter` accepts field names via `?ordering=` — if `ordering_fields` is not explicitly set, it defaults to all readable serializer fields. `SearchFilter` searches only `search_fields`. Apps using `django-filter` expose additional ORM lookups:

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | Sort by sensitive field | `?ordering=password` or `?ordering=-balance` | 200 OK = field exists; sorted results reveal data |
| 2 | Search field discovery | `?search=admin` on user endpoint | Returns matches — scope depends on configured `search_fields` |
| 3 | django-filter operator abuse | `?email__contains=@corp.com` or `?role__in=admin,superuser` | Django ORM lookups exposed via `django-filter` FilterSets |
| 4 | Related field traversal | `?ordering=profile__ssn` or `?search=company__billing__card` | Traverse foreign keys to access related model data |

**2. Serializer Field Exposure**

DRF serializers define which fields are returned. Common misconfiguration: using `fields = '__all__'` or forgetting to exclude sensitive fields.

```
# Native DRF does NOT support ?fields= query parameter.
# Test if the app uses a field-selection library (django-rest-flex-fields, etc.):
# ?fields=password,secret_key,api_token
# ?expand=profile,billing,admin_notes
# If the response includes new fields not in default output, it's a finding.
# Also check: does the browsable API (?format=api) expose more fields than JSON?
```

**CVE-2026-30244** (Plane): Unauthenticated workspace member info disclosure via DRF permission misconfiguration — `AllowAny` on protected ViewSet action leaked emails and PII for any workspace.

**3. ViewSet Action Auth Gaps**

DRF ViewSets can have per-action permissions. The `@action` decorator often inherits the ViewSet's default permissions, which may be too permissive:

```
# Test: For any /api/resource/{id}/custom-action endpoint:
# 1. Call without authentication — does it return data?
# 2. Call as low-privilege user — same response as admin?
# 3. Call with another user's ID — IDOR via custom action?
```

---

## Spring Boot

### Detection

| Signal | Confidence |
|--------|-----------|
| `Whitelabel Error Page` in HTML responses | High |
| `/actuator/health` returns `{"status":"UP"}` | High |
| `JSESSIONID` cookie | High |
| `/v2/api-docs` or `/v3/api-docs` (Springfox/SpringDoc) | High |
| `X-Application-Context` header | Medium |

### High-Value Attack Surface

**1. Actuator Endpoint Exploitation**

Spring Boot Actuator is the #1 attack surface for Spring APIs. Over 2,000 IPs scan for actuator endpoints daily (GreyNoise 2025).

| Endpoint | Impact | Severity |
|----------|--------|----------|
| `/actuator/env` | Environment variables with DB passwords, API keys, secrets | Critical |
| `/actuator/heapdump` | Full JVM heap — extract plaintext credentials, session tokens | Critical |
| `/actuator/configprops` | All configuration properties including secrets | High |
| `/actuator/mappings` | Complete URL mapping — reveals hidden endpoints | Medium |
| `/actuator/beans` | All Spring beans — reveals internal architecture | Low |
| `/actuator/loggers` | Read/write log levels — set to DEBUG for verbose output | Medium |
| `/actuator/jolokia` | JMX over HTTP — often leads to RCE via MBean invocation | Critical |
| `/actuator/gateway/routes` | Spring Cloud Gateway route table | High |

**CVE-2025-22235**: `EndpointRequest.to()` creates a matcher for `null/**` when the referenced actuator endpoint is disabled/unexposed. Affects apps that also handle the `/null` path — the unintended matcher can cause security rules to apply incorrectly.

**Worked Example — Heapdump Credential Extraction:**
```
# 1. Discover actuator
GET /actuator HTTP/1.1

# 2. Download heap dump
GET /actuator/heapdump HTTP/1.1
# Response: Binary file (often 100MB+)

# 3. Analyze with jhat or Eclipse MAT
# Search heap for: password, secret, token, apikey, jdbc
# Strings like "spring.datasource.password=P@ssw0rd123" appear in plaintext
#
# Finding: Unauthenticated access to /actuator/heapdump exposes
# database credentials, API keys, and session tokens from JVM memory.
# Severity: Critical (credential disclosure → full backend access)
```

**2. SpEL Injection**

Spring Expression Language (SpEL) injection in error handling, validation messages, or query parameters can lead to RCE:

```
# Test payloads in parameters reflected in error messages:
${7*7}                           → 49 = SpEL evaluated
${T(java.lang.Runtime).getRuntime().exec('id')}  → RCE
#{systemProperties['os.name']}  → OS info disclosure
```

---

## Laravel / PHP

### Detection

| Signal | Confidence |
|--------|-----------|
| `laravel_session` cookie | High |
| `XSRF-TOKEN` cookie (encrypted) | High |
| `X-Powered-By: PHP/8.x` | Medium |
| `/_ignition/health-check` returns 200 | High (debug mode) |
| JSON errors with `exception`, `file`, `line` fields | High (debug mode) |

### High-Value Attack Surface

**1. Ignition Debug Mode RCE (CVE-2021-3129)**

The most exploited Laravel vulnerability. When `APP_DEBUG=true` in production (common in staging environments left accessible):

```
# Test: POST /_ignition/execute-solution
# If 200 OK, debug mode is active and exploitable.
# Full exploit uses file_get_contents/file_put_contents chain
# via MakeViewVariableOptionalSolution to achieve RCE.
# CVSS 9.8, unauthenticated.

# Quick check without exploitation:
GET /_ignition/health-check HTTP/1.1
# 200 = Ignition active (debug mode likely enabled)
# 404 = Ignition not exposed
```

**2. Mass Assignment via Fillable/Guarded**

Laravel Eloquent models use `$fillable` (whitelist) or `$guarded` (blacklist) to control mass assignment. Common mistake: `$guarded = []` (nothing guarded).

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | Role escalation | `{"role": "admin"}` in PUT/POST | User role changes |
| 2 | Verification bypass | `{"email_verified_at": "2026-01-01"}` | Account marked verified without email confirmation |
| 3 | Tenant crossing | `{"team_id": 999}` or `{"organization_id": 1}` | Resource moved to different tenant |
| 4 | Timestamp manipulation | `{"created_at": "2020-01-01"}` | Bypass time-based business rules |

**3. Route Model Binding IDOR**

Laravel auto-resolves `{user}` in routes to User model via primary key. If authorization middleware is missing on the route:

```
# Route: PUT /api/users/{user}/profile
# Laravel resolves {user} to User model automatically
# Test: Change {user} to another user's ID
# If the update succeeds, it's an IDOR — the route relies on
# model binding for lookup but doesn't verify ownership.
```

---

## FastAPI / Python

### Detection

| Signal | Confidence |
|--------|-----------|
| `/docs` returns Swagger UI | High |
| `/redoc` returns ReDoc UI | High |
| `/openapi.json` returns OpenAPI spec | High |
| Error format: `{"detail": [{"loc": [...], "msg": "...", "type": "..."}]}` | High |
| `uvicorn` in `Server` header | Medium |

### High-Value Attack Surface

**1. Auto-Generated Documentation Exposure**

FastAPI serves interactive API docs by default at `/docs` and `/redoc`. In production, these often remain enabled, exposing the full API schema including internal endpoints.

```
# Always check:
GET /docs       → Swagger UI (try endpoints directly)
GET /redoc      → ReDoc documentation
GET /openapi.json → Full schema (import into Burp/Postman)

# The schema reveals all endpoints, request/response models,
# authentication requirements, and parameter types.
# Internal or admin endpoints are often included in the schema
# even when they're intended to be hidden.
```

**2. Pydantic Type Coercion Bypass**

FastAPI uses Pydantic for input validation. Both Pydantic v1 and v2 coerce types by default — string `"1"` becomes integer `1`, `"true"` becomes boolean `True`. This can bypass validation:

```
# If endpoint expects: {"amount": int, "approved": bool}
# Send: {"amount": "99999", "approved": "yes"}
# Pydantic coerces silently. The handler receives int(99999), bool(True).
# v2 changed some coercion rules but still coerces by default.
# Only apps using strict mode (model_config = {"strict": True}) are protected.
```

**CVE-2026-23996** (fastapi-api-key): Timing side-channel — rate-limiting jitter added only to failed auth attempts, not successful ones. Valid API keys return faster than invalid ones, enabling brute-force via response timing.

**CVE-2025-68481** (fastapi-users): OAuth login CSRF — stateless state parameter with no per-request entropy or correlation cookie. Attacker can force victim into attacker's OAuth session.

**3. Dependency Injection Auth Gaps**

FastAPI uses `Depends()` for authentication. Missing or incorrect dependency chains skip auth:

```
# VULNERABLE — no auth dependency
@app.get("/api/admin/users")
async def list_users():
    return await User.all()

# SECURE — auth dependency
@app.get("/api/admin/users")
async def list_users(user: User = Depends(get_admin_user)):
    return await User.all()

# Test: Call admin endpoints without authentication.
# FastAPI returns 200 (not 401/403) if auth dependency is missing.
```

---

## Ruby on Rails

### Detection

| Signal | Confidence |
|--------|-----------|
| `_<appname>_session` cookie (app-specific name) | High |
| `X-Request-Id` header (UUID format) | High |
| `X-Runtime` header (float) | Medium |
| `/rails/info/routes` in development | High |
| `ActionController::RoutingError` in errors | High |

### High-Value Attack Surface

**1. Strong Parameters Bypass**

Rails uses `params.require(:user).permit(:name, :email)` to whitelist fields. Bypasses:

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | Nested attributes | `user[profile_attributes][admin]=true` | `accepts_nested_attributes_for` without attribute filtering |
| 2 | Array parameter | `user[role][]=admin&user[role][]=superuser` | Array where string expected — may bypass permit checks |
| 3 | Type confusion | `user[id]=other_user_id` | ID overwrite via mass assignment |
| 4 | Deep nesting | `user[company_attributes][billing_attributes][card_number]=x` | Multi-level nested attributes traversing associations |

**2. Active Storage File Processing RCE (CVE-2025-24293)**

Active Storage's transformation allowlist permits unsafe methods/parameters when the app passes untrusted input to `variant`:

```
# VULNERABLE — user controls transformation parameters passed to variant()
# Active Storage allows transformation methods that mini_magick passes
# directly to ImageMagick, enabling command injection.
#
# Test: Look for endpoints that accept image transformation parameters
# (resize, crop, format) and pass them to Active Storage variants.
#
# CVSS: 9.2 Critical. Requires image_processing + mini_magick + user-controlled transform.
# Affects Rails ≥5.2.0 before 7.1.5.2 / 7.2.2.2 / 8.0.2.1.
```

**CVE-2025-55193**: ANSI escape injection in Active Record logging — attacker-controlled IDs logged without sanitization. Risk depends on terminal emulator: log spoofing is universal, but some vulnerable terminal emulators may interpret escape sequences as commands.

**3. Render File Disclosure**

```
# If a controller uses render with user input:
# render params[:template]  → read arbitrary files
# render file: params[:path] → direct file read
#
# Test: Add template=../../../etc/passwd or path=/etc/passwd
# If file contents appear in response, it's LFI.
```

---

## Next.js / React Server Components

### Detection

| Signal | Confidence |
|--------|-----------|
| `__NEXT_DATA__` in HTML source | High |
| `/_next/` paths for static assets | High |
| `x-nextjs-cache` header | High |
| `Next-Action` header in requests | High (App Router + Server Actions) |
| `x-vercel-id` header | Medium (Vercel-hosted) |

### High-Value Attack Surface

**1. React Server Components Deserialization RCE (CVE-2025-55182 / CVE-2025-66478)**

CVSS 10.0. The RSC Flight protocol deserializes user input on the server. Insecure deserialization allows attacker-controlled inputs to execute server-side logic. Affects any app using React Server Components (not just Server Actions).

```
# Affected: Next.js App Router 13.3+ (any version using RSC)
# NOT affected: Pages Router, Edge Runtime
#
# Detection: Look for __NEXT_DATA__ or /_next/ paths (App Router).
# Next-Action header indicates Server Actions, but apps using RSC
# without Server Actions are also vulnerable.
#
# Test: Send crafted serialized payload in the RSC request body.
# Patched in: next@14.2.35, 15.0.8, 15.1.12, 15.2.9, 15.3.9,
#             15.4.11, 15.5.10, 16.0.11, 16.1.5
```

**2. Middleware Auth Bypass**

Next.js middleware runs at the edge before page/API routes. Path matching issues can bypass auth:

```
# If middleware protects /dashboard:
# Test: /dashboard → 302 redirect (auth works)
#       /Dashboard → 200 (case-sensitive matching bypass?)
#       /dashboard/ → 200 (trailing slash bypass?)
#       /dashboard%00 → 200 (null byte bypass?)
#       /_next/data/BUILD_ID/dashboard.json → data route bypasses middleware?
#
# The /_next/data/ path for ISR/SSR pages may not trigger middleware,
# exposing page data without authentication.
```

**3. Server Action CSRF**

Server Actions in Next.js App Router use POST requests with specific headers. If SameSite cookie settings are permissive:

```
# Server Actions require Next-Action header + specific body format.
# Test: Can you invoke a server action from a cross-origin page?
# If the server action mutates data (delete, update) and accepts
# cross-origin requests, it's CSRF on server actions.
```

**CVE-2026-23864**: DoS via RSC — crafted requests trigger excessive server-side processing. CVSS 7.5.

---

## Cross-Framework Patterns

These patterns apply across all frameworks and often yield findings:

### 1. Debug/Development Mode in Production

| Framework | Debug Endpoint | What Leaks |
|-----------|---------------|------------|
| Express | Stack traces in JSON errors | File paths, module versions |
| Django | `DEBUG=True` → yellow error page | Settings, SQL queries, full traceback |
| Spring Boot | `/actuator/env` | Environment variables, secrets |
| Laravel | `/_ignition/execute-solution` | File read/write → RCE |
| FastAPI | `/docs`, `/redoc` | Full API schema |
| Rails | `/rails/info/routes` | All application routes |
| Next.js | `/_next/development/...` | Source maps, build info |

### 2. Mass Assignment Summary

| Framework | Mechanism | Bypass Vector |
|-----------|-----------|---------------|
| Express + Mongoose | `req.body` → `Model.update()` | No field filtering by default |
| Django REST | `serializer.save(**data)` | `fields = '__all__'` or missing exclusions |
| Spring Boot | `@ModelAttribute` auto-binding | POJO without field restrictions |
| Laravel | `$model->fill($request->all())` | `$guarded = []` misconfiguration |
| FastAPI | Pydantic model with extra fields | `model_config = {"extra": "allow"}`; coerces types by default (v1+v2) |
| Rails | `params.permit(...)` | Nested attributes, array type confusion |

### 3. Framework Version Fingerprinting

```
Quick version detection (for CVE applicability):
- Express: check package.json in source maps, or npm error pages
- Django: DEBUG error page shows version, or Server header
- Spring Boot: /actuator/info often includes version, or error page
- Laravel: /vendor/laravel path structure, or Telescope debug panel
- FastAPI: /openapi.json may include version metadata
- Rails: X-Runtime header format changed across versions
- Next.js: /_next/static/BUILD_ID, or __NEXT_DATA__.buildId
```

---

## Related Skills & Commands

- `api-security` SKILL.md — Core API testing patterns (REST, GraphQL, WebSocket, gRPC)
- `api-patterns.md` — Advanced GraphQL federation, mass assignment, JWT, rate limiting
- `source-code-audit/reference/framework-patterns.md` — Source-level patterns (10 languages)
- `auth-testing` — BOLA/BFLA/access control testing
- `/quick-test` — Rapid single-endpoint testing
