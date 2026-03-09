# Framework-Specific Security Anti-Patterns

Language and framework-specific vulnerability patterns for source code audits. Reference file for the source-code-audit skill.

> **Related:** [SKILL.md](../SKILL.md) for the core audit methodology and execution flow

---

## Table of Contents

- [Node.js / Express](#nodejs--express)
- [Python / Django / Flask](#python--django--flask)
- [PHP / Laravel](#php--laravel)
- [Ruby / Rails](#ruby--rails)
- [Java / Spring](#java--spring)
- [Go](#go)
- [TypeScript / Node](#typescript--node-type-specific-issues)
- [Rust (Web Frameworks)](#rust-web-frameworks--actix-axum-rocket)
- [AI/LLM Integration Patterns](#aillm-integration-patterns)
- [MCP Server Implementation Patterns](#mcp-server-implementation-patterns)

---

## Node.js / Express

| Pattern | Risk |
|---------|------|
| `eval()`, `Function()` with user input | RCE |
| Prototype pollution via `Object.assign`, `_.merge` | Various |
| `req.query` used directly in MongoDB queries | NoSQL injection |
| Missing `helmet()` or security headers | Various |
| `express.static()` serving sensitive directories | Information disclosure |
| `child_process.exec()` with string interpolation | Command injection |
| API data processing with unsanitized input (CWE-94) | Code injection — CVE-2026-25888 (Chartbrew, CVSS 8.8): improperly sanitized input in API data processing enables arbitrary code execution |

---

## Python / Django / Flask

| Pattern | Risk |
|---------|------|
| `os.system()`, `subprocess.call(shell=True)` | Command injection |
| `pickle.loads()` on untrusted data | RCE |
| `render_template_string()` with user input | SSTI |
| `DEBUG = True` in production settings | Information disclosure |
| Missing CSRF middleware | CSRF |
| `mark_safe()` with user-controlled content | XSS |

---

## PHP / Laravel

| Pattern | Risk |
|---------|------|
| `$_GET`/`$_POST` in SQL without binding | SQL injection |
| `unserialize()` on user input without class restrictions | Object injection / RCE — CVE-2026-3452 (Concrete CMS, CVSS 8.9): admin-stored serialized data passed to `unserialize()` without `allowed_classes` |
| `include()`/`require()` with user input | LFI/RFI |
| Missing `csrf_field()` in forms | CSRF |
| `{!! $var !!}` in Blade templates | XSS |
| `env()` values in client-exposed config | Secret leakage |

---

## Ruby / Rails

| Pattern | Risk |
|---------|------|
| `params.permit!` or weak strong params | Mass assignment |
| `.where("name = '#{params[:name]}'")` | SQL injection |
| `render inline:` with user input | SSTI |
| `send(params[:method])` | Arbitrary method call |
| Missing `protect_from_forgery` | CSRF |

---

## Java / Spring

| Pattern | Risk |
|---------|------|
| String concatenation in JPQL/HQL | Injection |
| `ObjectInputStream.readObject()` on untrusted data | Deserialization RCE |
| XML parsing without disabling external entities | XXE |
| `@RequestMapping` without method restriction | Method confusion |
| SpEL expressions with user input | RCE |

---

## Go

| Pattern | Risk |
|---------|------|
| `fmt.Sprintf` in SQL queries | SQL injection |
| `os/exec.Command` with user input | Command injection |
| `template.HTML()` to bypass auto-escaping | XSS |
| Missing error checks (unchecked `err`) | Various |
| Race conditions from shared state without mutex | Data corruption, auth bypass |

---

## TypeScript / Node (Type-Specific Issues)

| Pattern | Risk |
|---------|------|
| `as any` type assertions bypassing type safety | Type confusion, injection |
| `@ts-ignore` / `@ts-expect-error` hiding security issues | Various suppressed warnings |
| Zod/Yup schema too permissive (`.passthrough()`, loose unions) | Input validation bypass |
| `express.Request` with unvalidated `req.params` types | Type mismatch exploitation |
| Server Actions (Next.js) without auth checks | Unauthorized server-side execution |
| tRPC procedures missing `.input()` validation | Unvalidated API input |
| Prisma raw queries: `$queryRaw` with template interpolation | SQL injection despite ORM |
| `dangerouslySetInnerHTML` in React SSR with user data | Stored XSS |
| Edge runtime (Cloudflare Workers, Vercel Edge) with `eval()` | RCE at edge |
| Missing `Content-Security-Policy` in `next.config.js` | XSS amplification |
| **Runtime type erasure in security-critical code** | **Sandbox escape, auth bypass** |
| **Content-Type header not verified before body parsing** | **File upload forge, session hijack** |

### TypeScript Runtime Type Erasure (Critical Hunt Pattern)

TypeScript types are **erased at compilation** — they provide zero runtime enforcement. Any security-critical function that relies on TypeScript type annotations for input validation is vulnerable:

**The Pattern (CVE-2026-25049, n8n, CVSS 9.8):**
```typescript
// DANGEROUS: TypeScript annotation says 'string' but runtime doesn't enforce it
function sanitize(key: string): string {
  // Assumes key is always a string — attacker passes object/array at runtime
  return key.replace(/dangerous/g, '');
}
```

**What to Audit:**
1. **Sandbox/expression engines** — any function that sanitizes user input based on TypeScript type annotations alone. Look for `key: string` parameters that accept destructured values
2. **API input validation** — functions annotated with TypeScript types but missing runtime `typeof`/`instanceof` checks
3. **Security-critical middleware** — auth, rate limiting, or access control functions that assume parameter types
4. **process.binding() access** — sandbox escapes via `process.binding('spawn_sync')` bypassing module restrictions

**Code Patterns to Flag:**
```typescript
// FLAG: Security function relying on TS type only
function validateInput(data: string): boolean { ... }

// FLAG: Destructuring can bypass type assumptions
const { key } = userInput; // key could be any type at runtime

// SAFE: Explicit runtime type check
if (typeof key !== 'string') throw new Error('Invalid type');
```

**Key Insight:** If you find a TypeScript codebase with a sandbox, expression evaluator, or any security boundary that filters based on assumed types — test with non-string values (objects, arrays, symbols). This pattern caused a CVSS 9.8 RCE in n8n affecting ~100K instances.

### Content-Type Confusion in Request Handlers

When request handlers don't verify `Content-Type` before parsing the body, attackers can forge file uploads and manipulate request processing:

**The Pattern (CVE-2026-21858, n8n Ni8mare, CVSS 10.0):**
```typescript
// DANGEROUS: Processing file data without verifying Content-Type is multipart/form-data
app.post('/webhook', (req, res) => {
  const files = req.body.files; // Attacker controls req.body.files without actual file upload
  processFiles(files);          // Leads to arbitrary file read, session forge, RCE
});
```

**What to Audit:**
1. Webhook endpoints that accept file uploads — verify `Content-Type: multipart/form-data` is checked
2. Any endpoint where `req.body.files` or similar is accessed without content-type verification
3. Form handlers that process file metadata from the request body rather than actual multipart data

---

## Rust (Web Frameworks — Actix, Axum, Rocket)

| Pattern | Risk |
|---------|------|
| `unsafe` blocks with user-controlled data | Memory corruption, UB |
| `.unwrap()` on user input parsing | DoS via panic |
| `format!()` in SQL queries instead of parameterized | SQL injection |
| Missing CSRF protection on state-changing handlers | CSRF |
| `std::process::Command` with user input | Command injection |
| Deserialization of untrusted data via `serde` without validation | Object injection |
| `include_str!()` / `include_bytes!()` with relative paths | Path traversal at build |
| Missing rate limiting on auth endpoints in Actix/Axum | Brute force |
| `tokio::spawn` without proper error propagation | Silent auth failures |
| Shared mutable state via `Arc<Mutex<>>` without deadlock prevention | DoS, race conditions |

---

## AI/LLM Integration Patterns

When auditing code that integrates with LLM APIs (OpenAI, Anthropic, Google AI):

| Pattern | What to Look For | Severity |
|---------|-----------------|----------|
| **Prompt construction** | User input concatenated directly into prompts without sanitization | High — enables prompt injection |
| **Output trust** | LLM output passed to `eval()`, `exec()`, SQL queries, or rendered as HTML without escaping | Critical — enables RCE, SQLi, XSS via LLM output |
| **API key handling** | OpenAI/Anthropic/Google API keys hardcoded, in env vars without rotation, or leaked in frontend bundles | High — credential exposure |
| **System prompt exposure** | System prompts stored in client-accessible files, frontend code, or version control | Medium — information disclosure |
| **Token/cost controls** | No rate limiting on LLM API calls, no max token limits, no cost caps | Medium — economic DoS |
| **Multi-tenant isolation** | Shared conversation context between users, no per-user session isolation | Critical — cross-user data leakage |
| **Tool call validation** | LLM tool/function calls executed without parameter validation or allowlist enforcement | Critical — arbitrary action execution |
| **Memory/RAG poisoning** | Vector database ingestion from user-controlled sources without sanitization | High — persistent prompt injection (LPCI) |

**Code Patterns to Flag:**

```
# DANGEROUS: User input directly in prompt
prompt = f"Summarize this: {user_input}"
response = openai.chat.completions.create(messages=[{"role": "user", "content": prompt}])

# DANGEROUS: LLM output trusted and executed
result = llm.generate(user_query)
eval(result)  # or subprocess.run(result) or cursor.execute(result)

# DANGEROUS: No output sanitization before HTML rendering
return render_template("result.html", content=llm_response)

# FLAG: Check if API keys are in source
OPENAI_API_KEY = "sk-..."  # hardcoded key
```

---

## MCP Server Implementation Patterns

When auditing MCP server code:

| Pattern | What to Look For | Severity |
|---------|-----------------|----------|
| **Command injection** | User-supplied arguments passed to `exec()`, `spawn()`, `system()` without sanitization | Critical — 67% of MCP servers have Code Injection risks (Endor Labs) |
| **Path traversal** | File paths from MCP tool parameters not validated — `../` traversal | Critical — 82% use file system operations prone to Path Traversal |
| **Authentication** | No auth on MCP endpoints, or static tokens without rotation | High — 38% of scanned MCP servers lack auth entirely |
| **Tool description poisoning** | Tool descriptions loaded from external/user-controlled sources | High — enables invisible manipulation of AI behavior |
| **Rug pull potential** | Tool definitions mutable between sessions without re-approval | High — post-deployment behavior modification |
| **OAuth endpoint validation** | Authorization endpoints accepted without URL validation (CVE-2025-6514 pattern) | Critical — RCE via malicious auth endpoint |
| **Case sensitivity in JSON parsing** | Field name handling differs from specification (CVE-2026-27896 pattern in Go SDK) | Medium — validation bypass |
| **Symlink handling** | File operations don't resolve symlinks before access checks | Critical — sandbox escape |
| **React Server Component deserialization** | Flight protocol deserialization of untrusted data in RSC implementations (React2Shell pattern, CVE-2025-55182) | Critical — pre-auth RCE, CVSS 10.0 |
| **Extension/plugin loading** | AI IDE extension recommendations loading from unvalidated registries; namespace squatting risk | High — IDEsaster: malicious code served to 1.8M+ developers |
| **Cloud identity token flows** | Actor Token, managed identity, or token exchange mechanisms without proper authorization checks | Critical — CVE-2025-55241: Global Admin via Actor Tokens |
| **MCP stdio blacklist bypass** | Command whitelists that accept `npx`/`uvx` but don't block flag injection (`-p` flag with `npx node`) | Critical — CVE-2026-30861 (WeKnora, CVSS 9.9): blacklist bypass via argument flags |
| **PostgreSQL expression bypass** | SQL injection protection that fails to recursively inspect child nodes within array/row expressions | Critical — CVE-2026-30860 (WeKnora, CVSS 9.9): dangerous functions smuggled inside array expressions |
| **JavaScript `with` statement sandbox escape** | Sandboxed expression engines using deprecated `with` syntax enabling scope chain manipulation | Critical — CVE-2026-1470 (n8n, CVSS 9.9): `with` statement breaks sandbox boundary |
| **Edge framework URL decode mismatch** | Different URL decoding functions (e.g., `decodeURI` vs `decodeURIComponent`) in router vs static file handler | High — CVE-2026-29045 (Hono v4.12.4): encoded slashes bypass route-based auth middleware while resolving to protected paths |
| **LLM framework hardcoded dangerous defaults** | LLM orchestration code nodes with `allow_dangerous_code=True`, `sandbox=False`, or similar permissive settings hardcoded | Critical — CVE-2026-27966 (Langflow, CVSS 9.8): hardcoded dangerous default → prompt injection to RCE via Python REPL |
| **GraphQL transport-inconsistent depth limits** | GraphQL depth limits enforced on HTTP but not WebSocket subscriptions | High — CVE-2026-30241 (Mercurius): WebSocket subscriptions bypass `queryDepth` entirely |
| **Webhook URL incomplete IP validation** | URL validators checking only `is_loopback` but not RFC 1918 ranges (10.x, 172.16.x, 192.168.x) | High — CVE-2026-30242 (Plane, CVSS 8.5): SSRF with full response read-back via private IPs |
| **SSE/cookie header injection** | `setCookie()` or `writeSSE()` accepting unsanitized semicolons, CR, LF characters | Medium — CVE-2026-29086/29085 (Hono): session fixation via cookie injection, SSE field injection |
