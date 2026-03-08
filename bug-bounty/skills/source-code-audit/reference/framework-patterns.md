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
| `unserialize()` on user input | Object injection / RCE |
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
