---
name: vuln-patterns
description: Testing patterns and checklists for web vulnerability classes that pay bounties. Covers IDOR, XSS, SSRF, authentication bypasses, injection, business logic, and more — with concrete test cases and payloads. Use when testing a web target, asking "how do I test for X", or needing a vulnerability checklist. For AI/LLM/MCP patterns, use ai-hunting instead.
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

## MITRE CWE Top 25 (2025)

The 2025 list, compiled from 39,000+ vulnerabilities disclosed June 2024-June 2025, shows **authorization flaws climbing fast** while traditional injection remains at the top:

| Rank | CWE | Name | Trend |
|------|-----|------|-------|
| 1 | CWE-79 | Cross-Site Scripting (XSS) | Stable at #1 (score 60.38) |
| 2 | CWE-89 | SQL Injection | Up 1 position |
| 3 | CWE-352 | Cross-Site Request Forgery (CSRF) | Up 1 position |
| 4 | **CWE-862** | **Missing Authorization** | **Up 5 positions — biggest climber** |
| 5 | CWE-787 | Out-of-Bounds Write | Stable |

**New entries in 2025:** CWE-120 (Classic Buffer Overflow), CWE-121 (Stack-based Buffer Overflow), CWE-122 (Heap-based Buffer Overflow), CWE-284 (Improper Access Control), CWE-639 (Authorization Bypass Through User-Controlled Key), CWE-770 (Allocation of Resources Without Limits).

**Key trend:** Authorization-related vulnerabilities (IDOR, BOLA, missing access controls) are rising as the most impactful vulnerability class, displacing traditional injection flaws in bounty value. Programs are shifting rewards toward identity, access, and business logic flaws.

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

**What it is:** Flaws in how the application implements its business rules — not technical vulnerabilities but logical errors that let you do things the developers didn't intend.

**Where to look:**
- Payment/checkout flows — coupons, credits, refunds, pricing
- Subscription/plan management — upgrade, downgrade, trial abuse
- User invitations and role management
- Export/import functionality
- Rate limiting on business-critical operations
- Multi-tenancy boundaries

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Price manipulation | Modify price/quantity params in purchase requests | Negative prices, zero-cost items, currency confusion |
| 2 | Coupon/credit abuse | Apply expired/other-user coupons, stack multiple discounts, race condition on single-use codes | Discounts beyond intended limits |
| 3 | Trial abuse | Re-register for trials, modify trial dates, access premium during cancel grace period | Indefinite free access to paid features |
| 4 | Feature gate bypass | Access premium API endpoints from free-tier account, modify plan ID in requests | Unpaid access to paid features |
| 5 | Refund/chargeback logic | Refund after consuming digital goods, double-refund race conditions | Money back + goods retained |
| 6 | Invitation abuse | Accept invite to wrong org, replay invite tokens, privilege escalation via invite role param | Cross-tenant access, unintended roles |
| 7 | Export data leak | Export functionality reveals more data than UI shows (hidden columns, other users' data) | Data beyond authorized scope |
| 8 | Bulk operation abuse | Import/CSV upload bypasses validation present in UI, mass-assign privileged attributes | Bypassed business rules at scale |
| 9 | State machine violation | Skip required workflow steps (skip payment → access content, skip approval → publish) | Unauthorized state transitions |
| 10 | Multi-tenancy crossover | Access resources by guessing tenant IDs, switch tenant context mid-session | Cross-tenant data access |

**Vertical-specific playbooks:**

**SaaS / DevTools:**
- Seat/license manipulation — add seats without billing, share single-seat across team
- API key scope escalation — create key with broader permissions than account allows
- Webhook abuse — trigger webhooks to internal/cloud-metadata URLs (SSRF via feature)
- SCIM provisioning bypass — provision users into orgs without admin approval
- OAuth app installation — install app into org you don't own via modified install flow

**E-commerce / Marketplace:**
- Seller-side fraud — list items with negative prices, manipulate shipping calculations
- Buyer protection abuse — claim non-delivery after receiving digital goods
- Gift card arbitrage — purchase gift cards at discount, redeem at full value
- Inventory manipulation — buy more units than available via race condition
- Cross-seller data access — access another seller's analytics, orders, or PII

**Fintech / Payments:**
- Double-spend via race conditions on transfers
- Interest calculation manipulation via timezone/date boundary abuse
- KYC bypass — access restricted features before verification completes
- Transaction limit bypass — split transactions to avoid limits
- Fee avoidance — structure transactions to bypass fee tiers

**Impact escalation:** Business logic bugs often start as Medium but chain into High/Critical when combined with financial impact (real money loss), privacy impact (PII exposure), or regulatory impact (compliance violations). Always frame financial impact in dollar terms.

---

### Race Conditions

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

### Deserialization

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
| 7 | LLM framework serialization | Test LangChain/LlamaIndex streaming/serialization paths with untrusted metadata | LangGrinch pattern: prompt cascades through deserialization to exfiltrate secrets |

**Tools:** ysoserial (Java), PHPGGC (PHP), Freddy (Burp extension), SerializationDumper.

---

### Prototype Pollution

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

## Extended Pattern References

For specialized or high-depth testing, load the relevant reference file:

- **`reference/ai-mcp-vulns.md`** — 18 AI/LLM test patterns (OWASP LLM Top 10), 63 MCP test patterns with CVE references, 8 LPCI patterns, severity guidance, bypass techniques, OWASP MCP Top 10 mapping, and 70+ real-world incident references. **Use when:** target has AI features, MCP integrations, agent workflows, AI coding tools, or LLM-powered functionality.

- **`reference/advanced-web-vulns.md`** — GraphQL (10 patterns + bypasses), JWT manipulation (11 patterns including JWE-wrapped PlainJWT), OAuth 2.0/OIDC (10 patterns + redirect_uri bypasses), API rate limiting (10 patterns), and n8n/workflow automation sandbox escapes (8 patterns). **Use when:** target has GraphQL API, JWT auth, OAuth flows, rate-limited endpoints, or workflow automation platforms.

For deep AI/agent hunting methodology beyond test patterns, use the **ai-hunting** skill.

---

## Using This Skill

### With Target Context
If you've already run `target-recon`, I'll tailor patterns to the detected tech stack and attack surface.

### Without Context
I'll give you the full checklist for the vulnerability class you ask about.

### During Hunting
Ask as you go: "I found a URL parameter that reflects in the page — what XSS patterns should I try?"

---

## Routing Guide

| If you need... | Go to... |
|---|---|
| AI/LLM/MCP/agent test patterns (63+ MCP, 18 AI/LLM, 8 LPCI) | `reference/ai-mcp-vulns.md` |
| AI/LLM hunting methodology, tools, market context | `ai-hunting` skill |
| GraphQL, JWT, OAuth, rate limiting, sandbox escape patterns | `reference/advanced-web-vulns.md` |
| OAuth, JWT, SSO, MFA bypass methodology | `auth-testing` skill |
| HTTP smuggling, cache poisoning, race conditions | `http-desync` skill |
| Cloud misconfiguration patterns (S3, IAM, serverless) | `cloud-security` skill |
| Mobile-specific testing (APK, IPA, deep links) | `mobile-security` skill |
| Full source code security review | `source-code-audit` skill |

---

## Related Skills

- **target-recon** — Identify the tech stack first, then use patterns for that stack
- **program-research** — Know what the program pays for before prioritizing test patterns
- **report-writing** — When you find something, write it up properly
- **hunt-plan** (command) — Combine patterns with recon into a structured hunting session
- **ai-hunting** — Specialized techniques for hunting AI/LLM vulnerabilities
