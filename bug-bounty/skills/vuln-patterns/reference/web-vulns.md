# Web & API Vulnerability Patterns

Advanced testing patterns for API-specific, GraphQL, JWT, OAuth, and specialized web attack surfaces. Reference file for the vuln-patterns skill.

## Table of Contents

- [Tech Stack Patterns](#tech-stack-patterns)
- [GraphQL](#graphql)
- [JWT (JSON Web Token)](#jwt-json-web-token)
- [OAuth 2.0 / OpenID Connect](#oauth-20--openid-connect)
- [API Rate Limiting & Resource Exhaustion](#api-rate-limiting--resource-exhaustion)
- [AI/Agent Attack Surface Routing](#aiagent-attack-surface-routing)
- [n8n / Workflow Automation Sandbox Escapes](#n8n--workflow-automation-sandbox-escapes)
- [Edge Framework Path Normalization Bypass](#edge-framework-path-normalization-bypass)
- [GraphQL WebSocket Subscription Depth Bypass](#graphql-websocket-subscription-depth-bypass)
- [Admin Panel Logic Errors — Typo-to-RCE Pattern](#admin-panel-logic-errors--typo-to-rce-pattern)
- [HTTP/3 Race Conditions (QUICker)](#http3-race-conditions-quicker)
- [React Server Components DoS](#react-server-components-dos)

> **Infrastructure & platform patterns** (CSS exfiltration, SSRF chains, Node.js bypass, remote desktop, MDM, webmail RCE, critical infra auth bypass, MotW bypass): See [infrastructure-vulns.md](infrastructure-vulns.md)

---

## Tech Stack Patterns

When you know the target's technology, focus your testing:

| Stack | Priority Vulns | Why |
|-------|---------------|-----|
| **Node/Express** | Prototype pollution, SSRF, NoSQL injection | Loose typing, MongoDB common |
| **PHP/Laravel** | SQL injection, file upload, deserialization | Legacy patterns, file handling |
| **Python/Django** | SSTI, SSRF, IDOR | Template engines, URL handling |
| **Ruby/Rails** | Mass assignment, IDOR, SSRF | ActiveRecord patterns, open redirect |
| **Java/Spring** | Deserialization, SSRF, XXE | XML processing, heavy serialization |
| **GraphQL** | IDOR via node queries, introspection, DoS | Query complexity, authorization gaps |
| **REST API** | IDOR, broken auth, rate limiting | Stateless design, ID exposure |
| **SPA (React/Vue/Angular)** | DOM XSS, broken access control, API abuse | Client-side rendering, exposed APIs |
| **WordPress** | Plugin vulns, SQLi, file upload | Plugin ecosystem, legacy code |
| **AWS-hosted** | SSRF → metadata, S3 misconfig, IAM issues | Cloud-specific attack surface |
| **AI/LLM features** | Prompt injection, system prompt leak, output injection, excessive agency, cross-agent privilege escalation | OWASP LLM Top 10 #1 risk; 540% increase in reports; present in 73% of production AI; CVEs in GitHub Copilot (9.6), Cursor (9.8), MS Copilot (9.3); second-order cross-agent injection in ServiceNow Now Assist |
| **AI coding IDEs** | Project file supply chain (hooks, MCP configs, env vars), extension recommendation squatting, legacy browser flaws, workspace trust bypass, pre-trust API exfiltration, unauthenticated local servers | IDEsaster: 30+ vulns, 24 CVEs; Cursor Workspace Trust disabled by default; CVE-2026-21852 pre-trust API key exfil; CVE-2026-22812 OpenCode local server RCE; 94+ Chromium CVEs affecting 1.8M devs |
| **MCP integrations** | Tool poisoning, rug pull attacks, command injection (CVE-2025-6514, CVSS 10.0), eval() epidemic, SDK cross-client leaks, WebSocket hijacking, sandbox/container escape, TypeScript type confusion, cross-system data exfiltration | 8,000+ servers exposed; 30+ CVEs in 60 days; CVE-2026-25049 TS sandbox escape; CVE-2026-27001/27002 OpenClaw sandbox+Docker escape; Adversa AI TOP 25; VulnerableMCP.info tracks CVEs |
| **Agentic AI apps** | Agent goal hijack, tool misuse, privilege escalation, memory poisoning, rogue agents, regex denylist bypass in framework shell tools | OWASP Top 10 for Agentic Applications (Dec 2025); CVE-2026-2256 MS-Agent regex bypass; Gravitee: 88% of orgs had AI agent incidents, 47% unmonitored; compounds with MCP vulns |
| **MCP protocol layer** | Token mismanagement, command injection, supply chain poisoning, tool poisoning, shadow servers, insecure data handling | OWASP MCP Top 10 (2026); 30+ CVEs in 60 days; map findings to MCP01-MCP10 |
| **LLM frameworks** | Serialization injection, unsafe deserialization, prompt cascading through streaming ops | LangGrinch (CVE-2025-68664, CVSS 9.3) in LangChain Core; test serialization paths in LangChain, LlamaIndex, Haystack |
| **Web3/Blockchain** | Reentrancy, access control, oracle manipulation, flash loan attacks | $3B+ in Web3 losses H1 2025; access control flaws caused $953.2M; OWASP Smart Contract Top 10 ranks access control #1 |
| **Hardware/IoT** | Firmware extraction, JTAG/UART access, BLE attacks, default credentials | 88% increase in hardware vulns (Bugcrowd 2025), Samsung paying up to $1M |
| **Identity/Access** | BOLA, BFLA, privilege escalation, session management, OAuth flows | Fastest growing vuln class (HackerOne 2025); organizations shifting rewards here as XSS/SQLi decline |
| **A2A protocol** | Agent identity spoofing, capability forgery, task chain poisoning, trust graph attacks | Google's Agent-to-Agent protocol: east-west traffic bypasses perimeters; 40+ academic threats identified (arXiv:2505.12490); no token lifetime limits |

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

**GraphQL CSRF via Content-Type Switching:**
- Many GraphQL endpoints accept `application/x-www-form-urlencoded` in addition to `application/json`
- Form-encoded requests are not blocked by same-origin policy — can be sent cross-origin via HTML form
- Test: submit GraphQL mutation as `POST` with `Content-Type: application/x-www-form-urlencoded` and `query=mutation{...}`
- If the endpoint accepts the form-encoded mutation without CSRF token, any mutation is exploitable cross-origin
- Severity: High if mutations include account changes, data modification, or admin actions

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

## AI/Agent Attack Surface Routing

The following AI/agent-specific test patterns are maintained in the **ai-mcp-vulns.md** reference to avoid duplication. Load that file when testing these surfaces:

| Attack Surface | Patterns | Reference |
|---|---|---|
| **Agentic Browser Hijacking** | Zero-click agent trigger, file system exfil, credential manager access, extension escalation | `ai-mcp-vulns.md` → Agentic Browser Hijacking |
| **MCP OAuth Bypass** | Missing state param, auth code replay, redirect URI bypass, role confusion | `ai-mcp-vulns.md` → MCP OAuth |
| **AI IDE Configuration** | Workspace trust bypass, rules file backdoor, extension squatting, MCP config injection | `ai-mcp-vulns.md` → AI IDE Configuration |
| **ContextCrush / Doc Supply Chain** | Custom rules injection, library doc poisoning, RAG poisoning, env file exfil | `ai-mcp-vulns.md` → ContextCrush |

For full AI/LLM hunting methodology, see the **ai-hunting** skill.

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

---

## Edge Framework Path Normalization Bypass

**What it is:** Exploiting URL decoding inconsistencies between routers and static file handlers in edge/serverless frameworks (Hono, Deno Fresh, etc.) to bypass middleware-based authentication.

**Key CVE:** CVE-2026-29045 (Hono, High severity) — `decodeURI` in router vs `decodeURIComponent` in `serveStatic` allows encoded slashes to bypass route-based middleware protections (e.g., `/admin/*` auth checks) while still resolving to protected filesystem paths. Fixed in v4.12.4.

**Where to look:**
- Applications using edge/serverless frameworks (Hono, Deno Fresh, Bun, Cloudflare Workers)
- Static file handlers behind authentication middleware
- Any route-based access control relying on path matching

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Encoded slash bypass | Request `/admin%2Fsecret-file` to bypass `/admin/*` middleware | Static file served without auth middleware executing |
| 2 | Double encoding | Request `/%252Fadmin/config` to bypass path-based rules | Path resolves after second decode round |
| 3 | Mixed encoding | Use `%2F` for slashes but normal characters otherwise | Router and file handler disagree on resolved path |
| 4 | Cookie attribute injection | Inject `\r\n` or `;` into cookie domain/path parameters | Session fixation or cookie poisoning (CVE-2026-29086, Hono) |
| 5 | SSE field injection | Inject CR/LF into `event`, `id`, `retry` fields of Server-Sent Events | Additional SSE fields injected into stream (CVE-2026-29085, Hono) |

**Severity Guidance:** High when path normalization bypass enables authentication bypass to protected resources. The root cause — decoder mismatch between components — is a recurring pattern across frameworks.

---

## GraphQL WebSocket Subscription Depth Bypass

**What it is:** GraphQL query depth limits enforced on HTTP queries/mutations but not on WebSocket subscriptions, allowing DoS via unbounded subscription nesting.

**Key CVE:** CVE-2026-30241 (Mercurius/Fastify, severity pending) — `queryDepth` limit not applied to WebSocket subscription operations. Allows arbitrarily deep subscription queries causing exponential data resolution. Fixed in v16.8.0.

**Where to look:**
- GraphQL APIs using Mercurius, Apollo Subscriptions, or any library with separate HTTP and WebSocket transports
- Applications with `queryDepth` or `depthLimit` configured

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | WebSocket depth bypass | Send deeply nested subscription over WebSocket that would be rejected over HTTP | Subscription accepted without depth check |
| 2 | Transport comparison | Send identical deep query over HTTP (blocked) and WebSocket (allowed) | Different enforcement per transport |
| 3 | Subscription DoS | Deeply nested subscription triggering exponential resolver calls | Server resource exhaustion, timeout, or crash |

**Severity Guidance:** High for DoS if depth limits are the only protection against query complexity attacks. Critical if subscription resolvers access sensitive data without per-field auth.

---

## Admin Panel Logic Errors — Typo-to-RCE Pattern

**What it is:** Simple comparison operator or logic errors in admin panels that disable security validation entirely, chaining to full RCE.

**Key CVE:** CVE-2026-26279 (Froxlor, CVSS 9.1) — single-character typo (`==` instead of `=`) disabled email validation → admin account creation → cron injection → root RCE.

**Where to look:** Self-hosted management panels (Froxlor, Webmin, ISPConfig, Plesk), admin panels with email verification, any auth flow with bypassable validation steps.

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Validation bypass | Create admin account — does email verification actually block access? | Account usable without email confirmation |
| 2 | Cron injection | If admin access achieved, test cron job creation with injected commands | Commands execute as root via cron |
| 3 | Comparison confusion | Test numeric vs string comparison in login/validation flows | `0 == "string"` evaluates true in PHP |

**Severity Guidance:** Critical when chaining to root RCE. These bugs are human-only (Tier A) — automated scanners don't understand the business logic chain from typo to root access. Froxlor has ~15K installations on Shodan.

---

## HTTP/3 Race Conditions (QUICker)

**What it is:** Race condition testing extended to HTTP/3 (QUIC transport), a previously untestable attack surface.

**Tool:** QUICker — first race condition exploitation tool for HTTP/3 (academic research, ScienceDirect 2026).

**Why it matters:** Many applications migrating to HTTP/3 assumed race condition attacks only applied to HTTP/1.1 and HTTP/2. QUIC's multiplexed streams create new timing windows.

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | H3 single-packet attack | Send multiple requests in a single QUIC Initial packet | Requests processed simultaneously, bypassing serialization |
| 2 | Transport comparison | Test identical race condition on HTTP/1.1, HTTP/2, and HTTP/3 | Different race windows across transports |
| 3 | Stream multiplexing abuse | Use QUIC stream multiplexing for tighter timing windows | Sub-millisecond race windows exploitable |

**Severity Guidance:** Same as equivalent HTTP/1.1-2 race conditions. The novelty is that HTTP/3-only targets previously considered immune to race conditions are now testable.

---

## React Server Components DoS

**What it is:** DoS and RCE via crafted requests to Server Function endpoints in React Server Components (RSC).

**Key CVEs:** CVE-2026-23864 (RSC DoS, CVSS 7.5, multiple vectors, Jan 2026) and CVE-2025-55182 (React2Shell, CVSS 10.0, pre-auth RCE via insecure deser — #1 on HackerOne, **RondoDox botnet** mass-exploiting).

**Where to look:** Next.js Server Actions/Functions, React metaframeworks (Remix, Hydrogen), `/_next/` endpoints, any RSC implementation.

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Malformed RSC payloads | Send oversized/malformed serialized data to Server Function endpoints | Server crash, OOM, or CPU spike |
| 2 | Recursive object nesting | Deeply nested objects in Server Function arguments | Memory exhaustion or stack overflow |
| 3 | Deserialization boundary | Test if arguments are validated before deserialization | Code execution or type confusion |

**Severity Guidance:** DoS = Medium-High (CVSS 7.5). Deser-to-RCE = Critical (CVSS 10.0). React2Shell remains actively exploited — verify patching on any Next.js target.
