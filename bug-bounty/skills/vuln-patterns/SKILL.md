---
name: vuln-patterns
description: Vulnerability testing patterns and checklists for common bug classes. Get concrete test cases for IDOR, XSS, SSRF, authentication bypasses, AI/LLM vulnerabilities, and prompt injection — tailored to the target's tech stack. Trigger with "how do I test for", "IDOR checklist", "XSS test cases for", "what should I test on this endpoint", "vulnerability checklist", "common vulns in", "AI/LLM vulnerability", "prompt injection".
---

# Vulnerability Patterns

Concrete testing patterns for the vulnerability classes that pay bounties. Not theory — actionable test cases you can run against a target right now.

## How It Works

```
┌──────────────────────────────────────────────────────────────┐
│                  VULNERABILITY PATTERNS                       │
├──────────────────────────────────────────────────────────────┤
│  ALWAYS (works standalone)                                   │
│  ✓ Testing checklists for major vulnerability classes        │
│  ✓ Payloads and test cases for each pattern                  │
│  ✓ Tech-stack-specific variations                            │
│  ✓ Common bypasses for security controls                     │
│  ✓ What to look for in responses                             │
│  ✓ Severity and impact guidance per finding                  │
├──────────────────────────────────────────────────────────────┤
│  SUPERCHARGED (when you connect your tools)                  │
│  + Vulnerability DB: known CVEs for the target's stack       │
│  + Web scanner: automated testing for pattern validation     │
└──────────────────────────────────────────────────────────────┘
```

---

## Getting Started

Ask about any vulnerability class or testing scenario:

- "How do I test for IDOR on this API?"
- "XSS checklist for a React app"
- "What should I test on this file upload endpoint?"
- "Common authentication bypass patterns"
- "SSRF test cases for a URL preview feature"
- "What vulns should I look for in a GraphQL API?"

I'll give you concrete patterns tailored to the target's technology stack when I know it (from recon data or your description).

---

## Reference Files

This skill uses progressive disclosure. Core web vulnerability patterns are below. For specialized domains, read the appropriate reference file:

**AI/MCP/Agent vulnerabilities** → See [reference/ai-mcp-vulns.md](reference/ai-mcp-vulns.md)
Read this when: target has AI/LLM features, chatbots, MCP integrations, AI coding tools, agent platforms, AI browser features, or when you see `.go` files importing `github.com/mark3labs/mcp-go`, TypeScript MCP SDKs, LangChain/LlamaIndex imports, `.cursorrules`/`.mcp.json` config files, or agentic workflow platforms (n8n, Make, Zapier).

**Advanced web patterns** → See [reference/advanced-web-vulns.md](reference/advanced-web-vulns.md)
Read this when: target involves GraphQL endpoints, JWT tokens, OAuth/OIDC flows, race conditions in payment/checkout, deserialization (Java/PHP/Python/React), or prototype pollution in Node.js apps.

---

## MITRE CWE Top 25 (2025)

The 2025 list, compiled from 39,000+ vulnerabilities disclosed June 2024-June 2025, shows **authorization flaws climbing fast** while traditional injection remains at the top:

| Rank | CWE | Name | Trend |
|------|-----|------|-------|
| 1 | CWE-79 | Cross-Site Scripting (XSS) | Stable at #1 (score 60.38) |
| 2 | CWE-89 | SQL Injection | Up 1 position |
| 3 | CWE-352 | Cross-Site Request Forgery (CSRF) | Up 1 position |
| 4 | **CWE-862** | **Missing Authorization** | **Up 5 positions — biggest climber** |
| 5 | CWE-787 | Out-of-Bounds Write | Stable |

**Key trend:** Authorization-related vulnerabilities (IDOR, BOLA, missing access controls) are rising as the most impactful vulnerability class, displacing traditional injection flaws in bounty value.

---

## Vulnerability Classes

### Access Control / IDOR

**What it is:** Accessing or modifying resources belonging to other users by manipulating identifiers.

**Where to look:**
- Any endpoint with user-controlled IDs (numeric, UUID, encoded)
- API endpoints: `/api/users/{id}`, `/api/orders/{id}`, `/api/files/{id}`
- Download/export endpoints
- Account settings and profile endpoints
- Admin/management panels

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Horizontal access | Change numeric ID to another user's | Other user's data returned |
| 2 | Vertical access | Use low-priv token on admin endpoints | Admin data/actions accessible |
| 3 | ID enumeration | Increment/decrement sequential IDs | Valid responses for other users |
| 4 | UUID prediction | Check if UUIDs are v1 (time-based) | Predictable pattern |
| 5 | Encoded IDs | Decode base64/hex IDs, modify, re-encode | Access to other records |
| 6 | Parameter pollution | Add duplicate ID params: `?id=1&id=2` | Server uses unexpected value |
| 7 | HTTP method switch | Try GET→POST, POST→PUT on same endpoint | Different auth checks per method |
| 8 | Object reference in body | Change IDs in JSON request body | Server trusts body over session |
| 9 | Indirect references | Manipulate filenames, slugs, or paths | Access to unauthorized resources |
| 10 | State manipulation | Change object status via ID reference | Skip workflow steps |

**Bypasses when blocked:**
- Wrap ID in array: `{"id": [2]}` instead of `{"id": 2}`
- Use string: `{"id": "2"}` instead of numeric
- Add `.json` extension to path
- URL encode the ID parameter name
- Try GraphQL if REST is protected (or vice versa)
- Change API version: `/v1/` → `/v2/`

---

### Cross-Site Scripting (XSS)

**What it is:** Injecting client-side scripts that execute in another user's browser.

**Where to look:**
- Search fields and results pages
- User profile fields (name, bio, location)
- Comment and messaging features
- URL parameters reflected in page content
- Error messages that include user input
- File upload names and metadata
- Markdown/rich text editors

**Test patterns:**

| # | Test | Payload | Context |
|---|------|---------|---------|
| 1 | Basic reflection | `<script>alert(1)</script>` | Unfiltered HTML context |
| 2 | Attribute escape | `" onfocus=alert(1) autofocus="` | Inside HTML attribute |
| 3 | Tag escape | `</title><script>alert(1)</script>` | Inside a tag |
| 4 | Event handler | `<img src=x onerror=alert(1)>` | Image/media context |
| 5 | SVG injection | `<svg onload=alert(1)>` | SVG-allowed context |
| 6 | JavaScript URL | `javascript:alert(1)` | href/src attributes |
| 7 | Template injection | `{{constructor.constructor('alert(1)')()}}` | Angular/template engines |
| 8 | DOM-based | Check `location.hash` → `innerHTML` sinks | Client-side rendering |
| 9 | Markdown XSS | `[click](javascript:alert(1))` | Markdown renderers |
| 10 | CSP bypass | Check for unsafe-inline, unsafe-eval, loose whitelists | When CSP blocks basic payloads |

**Framework-specific:**

| Framework | Where to focus |
|-----------|---------------|
| React | `dangerouslySetInnerHTML`, `href` attributes, SSR hydration |
| Angular | Template injection `{{}}`, `bypassSecurityTrust*` APIs |
| Vue | `v-html` directive, template expressions |
| jQuery | `.html()`, `.append()` with user input |
| Server-rendered | Any reflected parameter, error messages |

---

### Server-Side Request Forgery (SSRF)

**What it is:** Making the server issue requests to unintended destinations (internal services, cloud metadata, etc.).

**Where to look:**
- URL preview/unfurl features
- Webhook configuration
- File import from URL
- PDF/image generation from URLs
- API integrations (OAuth callbacks, etc.)
- Proxy or redirect endpoints

**Test patterns:**

| # | Test | Payload | Target |
|---|------|---------|--------|
| 1 | Localhost | `http://127.0.0.1` | Local services |
| 2 | Cloud metadata | `http://169.254.169.254/latest/meta-data/` | AWS metadata |
| 3 | Internal DNS | `http://internal-service.local` | Internal network |
| 4 | Alternative IP | `http://0x7f000001` (hex localhost) | Bypass IP filters |
| 5 | DNS rebinding | Use a rebinding service | Bypass DNS-based checks |
| 6 | Redirect chain | URL that 302s to internal target | Bypass URL validation |
| 7 | Protocol smuggling | `gopher://`, `file:///` | Non-HTTP protocols |
| 8 | IPv6 | `http://[::1]` | Bypass IPv4-only filters |
| 9 | Decimal IP | `http://2130706433` | Bypass regex filters |
| 10 | URL parser confusion | `http://evil.com#@internal` | Parser differential |

**Cloud-specific targets:**
- AWS: `169.254.169.254` — IAM credentials, instance metadata
- GCP: `metadata.google.internal` — service account tokens
- Azure: `169.254.169.254` with `Metadata: true` header

---

### Authentication & Session

**What it is:** Bypassing login, session management, or identity verification.

**Where to look:**
- Login endpoints
- Password reset flows
- 2FA/MFA implementation
- Session token handling
- OAuth/SSO implementations
- API key management
- Remember-me functionality

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Brute force | Rapid login attempts | No rate limiting or lockout |
| 2 | Default credentials | Try common admin:admin pairs | Access granted |
| 3 | Password reset token | Check token entropy and expiration | Predictable or long-lived tokens |
| 4 | 2FA bypass | Skip 2FA step, go directly to post-auth endpoint | Access without 2FA |
| 5 | Session fixation | Set session before login, check if reused after | Same session ID post-auth |
| 6 | Token in URL | Check if tokens appear in URLs or referrer headers | Token leakage |
| 7 | OAuth misconfiguration | Manipulate redirect_uri, state parameter | Account takeover |
| 8 | Race condition | Send parallel requests during auth state change | Bypass checks via timing |
| 9 | JWT issues | Check for none algorithm, weak secret, no expiry | Forged or extended tokens |
| 10 | Password reset poisoning | Manipulate Host header in reset request | Reset link points to attacker |

---

### Injection (SQL, Command, Template)

**What it is:** Injecting code or commands that the server executes unintentionally.

**Where to look:**
- Search/filter parameters
- Sort/order parameters
- Login forms
- Any parameter that interacts with a database or system command
- Template rendering with user input
- GraphQL queries

**Test patterns:**

| Type | Test payload | What to look for |
|------|-------------|-----------------|
| SQL (error) | `' OR 1=1--` | SQL error messages |
| SQL (blind) | `' AND SLEEP(5)--` | Response time difference |
| SQL (union) | `' UNION SELECT NULL,NULL--` | Column count matching |
| Command | `; sleep 5` | Response delay |
| Command | `` `sleep 5` `` | Backtick execution |
| SSTI | `{{7*7}}` | "49" in response |
| SSTI (Jinja2) | `{{config.items()}}` | Config dump |
| SSTI (Twig) | `{{_self.env.display('id')}}` | Command execution |
| NoSQL | `{"$ne": null}` | Bypassed auth/filters |
| GraphQL | Introspection query | Schema disclosure |

---

### File Upload

**What it is:** Exploiting file upload functionality to achieve code execution, XSS, or data exfiltration.

**Where to look:**
- Profile image upload
- Document/attachment upload
- Import functionality
- Any file upload endpoint

**Test patterns:**

| # | Test | Method | Goal |
|---|------|--------|------|
| 1 | Extension bypass | Upload `.php.jpg`, `.php%00.jpg` | Code execution |
| 2 | Content-type mismatch | Set image/jpeg but send PHP content | Bypass MIME check |
| 3 | SVG with script | Upload SVG containing `<script>` | Stored XSS |
| 4 | HTML upload | Upload `.html` file with JavaScript | Stored XSS |
| 5 | Path traversal | Filename: `../../../etc/passwd` | File overwrite |
| 6 | Double extension | `file.php.png` | Server processes first extension |
| 7 | Case variation | `.PHP`, `.pHp` | Bypass case-sensitive filters |
| 8 | Polyglot file | Valid image with embedded PHP | Bypass image validation |
| 9 | XXE via file | Upload XML/DOCX with XXE payload | Internal file read |
| 10 | Size limits | Upload extremely large file | DoS or buffer overflow |

---

### Business Logic

**What it is:** Exploiting flawed application logic rather than technical vulnerabilities.

**Where to look:**
- E-commerce (pricing, discounts, quantities)
- Multi-step workflows (registration, checkout)
- Rate limiting and quotas
- Feature flags and premium features
- Referral/reward systems

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Price manipulation | Modify price in request | Discounted/free purchase |
| 2 | Negative quantities | Set quantity to -1 | Credit instead of charge |
| 3 | Race condition | Parallel identical requests | Double-spend, duplicate rewards |
| 4 | Workflow skip | Jump to step 3, skip validation in step 2 | Bypassed checks |
| 5 | Coupon stacking | Apply multiple exclusive coupons | Over-discounted total |
| 6 | Feature toggle | Modify request to enable premium features | Free premium access |
| 7 | Referral abuse | Refer yourself with different emails | Unlimited referral rewards |
| 8 | Mass assignment | Add extra fields: `{"role":"admin"}` | Privilege escalation |
| 9 | Time manipulation | Modify timestamps in requests | Extended trials, expired tokens working |
| 10 | Currency confusion | Switch currency mid-transaction | Arbitrage opportunity |

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
| **AI/LLM features** | Prompt injection, system prompt leak, excessive agency, cross-agent escalation | → Read [reference/ai-mcp-vulns.md](reference/ai-mcp-vulns.md) |
| **AI coding IDEs** | Project file supply chain, extension squatting, workspace trust bypass | → Read [reference/ai-mcp-vulns.md](reference/ai-mcp-vulns.md) |
| **MCP integrations** | Tool poisoning, command injection, eval() epidemic, SDK cross-client leaks | → Read [reference/ai-mcp-vulns.md](reference/ai-mcp-vulns.md) |
| **Agentic AI apps** | Agent goal hijack, tool misuse, memory poisoning, rogue agents | → Read [reference/ai-mcp-vulns.md](reference/ai-mcp-vulns.md) |
| **Identity/Access** | BOLA, BFLA, privilege escalation, session management, OAuth flows | Fastest growing vuln class |
| **Web3/Blockchain** | Reentrancy, access control, oracle manipulation, flash loan attacks | $3B+ in Web3 losses H1 2025 |
| **Hardware/IoT** | Firmware extraction, JTAG/UART, BLE attacks, default credentials | 88% increase in hardware vulns |

---

## Using This Skill

### With Target Context
If you've already run `target-recon`, I'll tailor patterns to the detected tech stack and attack surface.

### Without Context
I'll give you the full checklist for the vulnerability class you ask about.

### During Hunting
Ask as you go: "I found a URL parameter that reflects in the page — what XSS patterns should I try?"

---

## Related Skills

- **target-recon** — Identify the tech stack first, then use patterns for that stack
- **program-research** — Know what the program pays for before prioritizing test patterns
- **report-writing** — When you find something, write it up properly
- **hunt-plan** (command) — Combine patterns with recon into a structured hunting session
- **ai-hunting** — Specialized techniques for hunting AI/LLM vulnerabilities
