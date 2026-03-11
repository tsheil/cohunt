---
name: target-recon
description: Recons a web target — external fingerprinting AND authenticated attack surface mapping. Fingerprints tech stack, enumerates subdomains, detects WAF/CDN, then maps roles, states, endpoints, and tenant boundaries behind login. Outputs a Role × State × Endpoint matrix, state fixtures with transition endpoints and durable artifacts, and proof-first test cards ready for hunting. ALWAYS use this skill FIRST when a user mentions any target domain, URL, or application name — it should fire before any vulnerability testing. Trigger with "recon this target", "fingerprint example.com", "enumerate subdomains for", "what tech stack does X use", "map the attack surface of", "Google dork", "OSINT", "subdomain takeover", "ASN lookup", "CIDR range", "JavaScript analysis", "endpoint discovery", "what runs on", "what changed recently", "new features", "scope additions", "recent changes", "check security headers", "list endpoints", "API discovery", "map endpoints per role", "role-endpoint matrix", "authenticated recon", "set up test accounts". Use proactively when user mentions a target domain and hasn't done recon yet. For program research (rewards, scope, competition), use program-research. For testing vulnerabilities found during recon, use vuln-patterns or the specific skill for that vuln class. For cloud asset enumeration, use cloud-security.
---

# Target Recon

Map the full attack surface — external perimeter AND authenticated internals — before hunting. External recon finds the front door. Authenticated recon finds the bugs that pay.

**Priority order:** Authenticated recon (Steps 3-4) is where payable bugs live — IDOR, privilege escalation, business logic, tenant isolation. Do Steps 0-2 quickly, then invest most time in Steps 3-4. External enumeration (Steps 5-9) runs after or in parallel.

## Quick Start — First 5 Minutes

1. **Headers** — `curl -sI https://[target]` — Server, X-Powered-By, cookies, security headers.
2. **Subdomains** — `curl -s "https://crt.sh/?q=%25.[domain]&output=json" | jq '.[].name_value' | sort -u` — CT log discovery.
3. **Paths** — `curl -s https://[target]/robots.txt` + `/sitemap.xml` + `/.well-known/security.txt` — Hidden routes, bug bounty info.
4. **JS endpoints** — View page source, extract API endpoints from fetch/axios calls and `__NEXT_DATA__`.
5. **AI configs** — `curl -s https://[target]/.mcp.json` + `/CLAUDE.md` + `/.cursorrules` — API keys, internal URLs.

If you find something interesting, run full recon. If you have a finding, run `/reportability-check` before `/write-report`. If blocked, run `/variant-hunt`.

---

## Output Format

**→ Use the full output template at [reference/output-template.md](reference/output-template.md).** Sections: Quick Take, Target Profile, Tech Stack, Security Headers, Subdomain Inventory, Interesting Paths, WAF/CDN Analysis, Attack Surface Summary, **State Fixtures (with transition endpoints + artifacts), Role-Endpoint Matrix, Role × State × Endpoint Delta Matrix, Tenant Boundary Map, Proof-First Test Cards**, Sources.

---

## Execution Flow

### Step 0: Recent Changes Diff

Before recon, ask: **"What changed recently?"** New features, patches, scope additions, API versions, AI rollouts, and mobile releases are the highest-signal attack surface. Check:

```
1. Web search: "[target] changelog" OR "[target] release notes" OR "[target] new feature" (last 30 days)
2. Wayback Machine: curl -s "https://web.archive.org/web/timemap/link/https://[target]" | head -20
3. Compare current JS bundles vs archived versions for new endpoints
4. Check program updates: recent scope changes, new assets, bounty table changes
```

If the target changed recently, route to `/regression-hunt` thinking — test new surfaces first.

### Step 1: Parse Target

```
Identify what to recon:
- "recon example.com" → Full domain recon
- "fingerprint api.example.com" → Single host fingerprint
- "enumerate subdomains for example.com" → Subdomain-focused
- "what stack does example.com use" → Tech detection focus
```

### Step 2: HTTP Fingerprinting (Always)

```
Run these curl commands:
1. curl -sI https://[target] → Response headers, server, cookies
2. curl -sI http://[target] → HTTP redirect behavior
3. curl -s https://[target]/robots.txt → Disallowed paths
4. curl -s https://[target]/sitemap.xml → Site structure
5. curl -s https://[target]/.well-known/security.txt → Bug bounty info
6. curl -s https://[target]/ | head -200 → Page source for tech detection
7. curl -sI https://[target]/nonexistent-path-404 → Error page fingerprinting
```

**Extract:**
- Server software and version
- Framework indicators (X-Powered-By, meta tags, script sources)
- Cookie names and attributes (session handling clues)
- Security header presence and configuration
- Interesting disallowed paths from robots.txt

### Step 3: Authenticated Recon (Where the Money Is)

**Most high-severity bounties (IDOR, privilege escalation, business logic) live behind login.** If you have or can create an account, do this NOW — before spending hours on external enumeration.

> **No credentials yet?** Skip to Step 5 (subdomain enumeration) and return here after getting access.

#### State Fixture Setup (REQUIRED)

The gap between a good plan and a payable bug is almost always **missing test state**:

```
□ ACCOUNTS: 2+ at different privilege levels (free + paid, user + admin, tenant-A + tenant-B)
  Can't create these? → BLOCKER — note it, adjust plan
□ ROLES: Inventory every role the app supports (free, paid, admin, support, API-only, service account)
□ OAST COLLECTOR: Blind callback receiver ready (Burp Collaborator, interactsh, webhook.site)
□ API TOKEN PAIR: Tokens from 2 different users/roles
□ TOOLS: Burp/mitmproxy intercepting, browser profiles set
```

**State fixtures are test-case generators, not just setup.** For each state below, record the transition endpoint, durable artifacts to reuse post-transition, and whether you achieved it:

```
STATE FIXTURE TABLE — record for each state you create:

State ID             | Role  | Enter Via                    | Artifacts to Reuse After Transition
downgraded_from_paid | free  | PUT /api/billing/subscription | old session, refresh token, export IDs, signed URLs
pending_invite       | invited| POST /api/invites            | invite token, accept URL
suspended            | member| POST /api/admin/users/:id/suspend | session cookie, API token, websocket
expired_trial        | free  | time/billing event           | trial-only artifact IDs, feature flags
pending_approval     | user  | POST /api/approvals          | approval ID, linked resource
archived_workspace   | admin | DELETE /api/workspaces/:id    | workspace ID, shared link tokens
```

**Why:** Single-account testing is the #1 reason auth bugs get missed. Two accounts at different privilege levels is the minimum for any IDOR/BOLA/BFLA testing. State-transition bugs pay $2K-$15K — but only if you created the state AND saved artifacts to reuse afterward.

#### Transition Endpoints (Map These First)

State bugs live on the endpoints that **change** account state. Identify these before building the matrix:

```
□ billing/subscription — downgrade, cancel, plan change, payment failure
□ user lifecycle — suspend, deactivate, archive, delete, restore
□ invitations — send, accept, revoke, expire, resend
□ approvals — submit, approve, reject, cancel
□ onboarding — complete trial, verify email, activate
□ async jobs — start export, cancel import, abort processing
□ credentials — rotate API key, revoke OAuth token, reset password
```

**After each transition:** immediately retest with the old session/token/artifact. The bug is in what the server **forgets to revoke**.

#### Per-Role Endpoint Mapping → Role-Endpoint Matrix

```
For each user role (free, paid, admin, support, API-only):
1. Log in as that role
2. Spider the application — capture all requests in Burp/mitmproxy
3. Extract unique endpoint + method combinations
4. Build a role-endpoint matrix:

   Endpoint              | Free | Paid | Admin | API
   POST /api/users       |  ✓   |  ✓   |   ✓   |  ✓
   DELETE /api/users/:id  |  ✗   |  ✗   |   ✓   |  ✓
   GET /api/admin/stats   |  ✗   |  ✗   |   ✓   |  ✗
   PUT /api/billing       |  ✗   |  ✓   |   ✓   |  ✓

5. Every ✗ cell is a test target — try accessing with the blocked role
6. Pay special attention to: DELETE endpoints, admin-only GETs,
   export/billing endpoints, and endpoints with user IDs in the path
```

**This matrix IS a key recon deliverable.** Include it in the output. It directly generates test cases for auth-testing.

#### Role × State × Endpoint Delta Matrix

The role-endpoint matrix above uses **active state** as baseline. Now overlay **state transitions** — these are the highest-value test targets because servers systematically fail to revoke access after state changes.

**Build the delta:** For each state fixture you created, test the endpoints where state SHOULD change access. Only record rows where state matters (sparse — not every combination):

```
Card  | Role  | State              | Endpoint                     | Baseline | Expected | First Probe
TC-01 | free  | downgraded_from_paid| GET /api/exports/:id/download | paid-only| deny     | reuse pre-downgrade export ID with stale + fresh token
TC-02 | invited| pending_invite     | POST /api/invites/:id/accept  | single-use| deny    | replay invite token after admin revokes it
TC-03 | member| suspended           | GET /api/projects/:id         | ✓ own    | deny all | retry existing session cookie after suspension
TC-04 | free  | expired_trial       | POST /api/ai/generate         | trial-only| deny   | retry with expired-trial session
```

**Ranking heuristic:** Prioritize tuples where the endpoint touches a **monetized feature** (exports, AI, billing), uses a **durable artifact** (signed URL, invite token, export ID), or involves an **async workflow** (export job, approval queue). These consistently pay $3K-$15K.

#### Hidden Route Discovery

```
□ Predictable admin paths — /admin, /internal, /debug, /status, /_admin
□ API version exploration — if /api/v2/users exists, try /api/v1/users, /api/v3/users
□ Framework-specific routes — /actuator (Spring Boot), /__debug__ (Django), /elmah (ASP.NET)
□ GraphQL introspection — query { __schema { types { name fields { name } } } }
□ Swagger/OpenAPI endpoints — /swagger.json, /openapi.yaml, /api-docs
□ Sitemap and robots.txt — may reference internal paths
□ JavaScript bundle analysis — see ${CLAUDE_SKILL_DIR}/reference/javascript-analysis.md
□ Error-based discovery — send invalid requests to trigger stack traces with route info
□ AI/MCP config files — /.mcp.json, /.claude/, /.cursor/mcp.json, /.amazonq/, /CLAUDE.md, /AGENTS.md
```

#### Multi-Tenant Isolation Probing

```
□ Tenant ID in URL — /orgs/{tenant_id}/resource → swap tenant IDs
□ Tenant ID in headers — X-Tenant-ID, X-Organization-ID → swap values
□ Subdomain-based tenants — company1.app.com vs company2.app.com → cross-tenant requests
□ Shared resources — /shared/files/{id} → IDs may span tenants
□ Admin endpoints — tenant admin vs platform admin authorization boundaries
□ API key scoping — does API key for tenant A work against tenant B endpoints?
```

#### Webhook & Integration Mapping

```
□ Outgoing webhooks — where does the app send data? Test for SSRF via webhook URL
□ Incoming webhooks — /webhooks/*, /callbacks/* — test for signature bypass
□ OAuth integrations — which third-party apps are connected? Test token scope
□ Email notifications — do they include sensitive data? HTML injection?
□ Export features — CSV/PDF/Excel export — test for formula injection, IDOR on exports
□ Import features — CSV/file upload import — test for XXE, SSRF, path traversal
```

### Step 4: First 10 Test Cases

After authenticated recon, generate the first 10 test cases sorted by payout likelihood. This is the final recon deliverable — the hunter should be able to start testing immediately.

```
Priority order for test case generation:
1. IDOR on endpoints with user IDs in path (from role-endpoint matrix ✗ cells)
2. Privilege escalation — lower role accessing admin-only endpoints
3. Tenant isolation bypass — cross-tenant resource access (if multi-tenant)
4. Downgrade retained access — premium endpoints after plan downgrade
5. Business logic — payment/billing/checkout manipulation
6. SSRF — any URL input (webhooks, imports, preview, unfurl features)
7. Pending state abuse — cancel/modify after approval submitted
8. API version downgrade — v1 endpoints with weaker auth than v2
9. Mass assignment — add "role":"admin" or "verified":true to POST/PUT bodies
10. Race conditions — single-use actions (coupons, invites) via parallel requests
```

**For each test case, use the proof-first test card format** (consistent with hunt-session and hunt-plan):

```
Test Card TC-01: Retained premium export after downgrade
  Tuple: free + downgraded_from_paid + GET /api/exports/:id/download
  Baseline: Before downgrade, paid account downloads export → 200 + file bytes
  Attack: After downgrade, retry same export ID with stale session + fresh token + signed URL
  Verify: Vulnerable if any request returns the file. Safe if all return 403/404.
  FP Check: CDN cache? Intentional grandfathering? Check product docs.
  Evidence: Plan page showing free status, pre/post responses, export ID, timestamps
  Route To: auth-testing, business-logic

Test Card TC-02: BOLA on DELETE /api/v2/projects/:id
  Tuple: free + active + DELETE /api/v2/projects/:id (from R×E matrix ✗ cell)
  Baseline: Account A deletes own project → 204
  Attack: Account A sends Account B's project ID
  Verify: Vulnerable if 200/204 + project actually deleted. Safe if 403.
  FP Check: Is the project shared/public? Single-account test is invalid.
  Evidence: Two-account comparison, project existence before/after
  Route To: auth-testing → BOLA section
```

**Source test cards from three matrices:** Role × Endpoint ✗ cells (BOLA/BFLA), Role × State delta rows (state-transition bugs), and Tenant Boundary Map (cross-tenant access). Rank by monetized feature first.

---

### Step 5: Subdomain Enumeration

```
Run these lookups:
1. Certificate Transparency: curl -s "https://crt.sh/?q=%25.[domain]&output=json" | jq
2. DNS records: dig [domain] A AAAA CNAME TXT CAA +short (avoid deprecated ANY)
3. Name servers: dig ns [domain] +short
4. Mail servers: dig mx [domain] +short
5. Passive DNS: web search "site:securitytrails.com [domain]" for historical records
6. For each discovered subdomain: curl -sI → Status and server
7. Check for subdomain takeover indicators (CNAME to unclaimed services)
```

**Extract:** Subdomain inventory, DNS records, live vs. dead subdomains, takeover candidates, interesting subdomains (staging, api, admin, dev, internal).

### Step 6: WAF/CDN Detection

Check response headers for CDN signatures (Cloudflare, Akamai, Fastly) and WAF-specific headers/cookies. Compare direct IP vs domain response. Test with common WAF trigger payloads in User-Agent. Note rate limiting behavior.

### Step 7: Google Dorking & OSINT

```
Search for exposed assets and data:
1. site:[target] filetype:pdf OR filetype:xlsx OR filetype:doc → Sensitive documents
2. site:[target] inurl:admin OR inurl:dashboard OR inurl:panel → Admin interfaces
3. site:[target] inurl:api OR inurl:swagger OR inurl:graphql → API documentation
4. site:[target] "password" OR "secret" OR "api_key" → Leaked credentials in pages
5. site:[target] ext:env OR ext:yml OR ext:conf → Configuration files
6. "[target]" site:pastebin.com OR site:github.com → Leaked data on third parties
7. intitle:"index of" site:[target] → Directory listings
8. inurl:".git" site:[target] → Exposed git repositories
9. site:github.com "[target]" ".env" OR "mcp.json" OR ".cursorrules" → AI config + secret leaks
10. web.archive.org/web/*/[target]/* → Wayback Machine: deprecated endpoints, old API versions
11. site:github.com "[target]" "password" OR "secret_key" OR "api_key" → Leaked secrets
12. site:github.com "[target]" filename:.env OR filename:config → Config files in public repos
```

#### AI/MCP Configuration Artifact Discovery

```
Check for exposed AI agent and MCP configuration files:
1. curl -s https://[target]/.claude/settings.json → Claude Code config
2. curl -s https://[target]/CLAUDE.md → Project instructions (may leak internal architecture)
3. curl -s https://[target]/.mcp.json → MCP server configs (may contain API keys, internal URLs)
4. curl -s https://[target]/.cursor/mcp.json → Cursor MCP config
5. curl -s https://[target]/.amazonq/mcp.json → Amazon Q Developer config
6. curl -s https://[target]/.cursorrules → Cursor system prompts (may reveal business logic)
7. curl -s https://[target]/.github/copilot-instructions.md → Copilot custom instructions
8. curl -s https://[target]/AGENTS.md → Agent definitions
9. Web search: site:github.com "[company]" ".mcp.json" OR "CLAUDE.md" OR ".cursorrules"
```

**Why:** AI config files are the new `.env` — developers commit them with embedded API keys, internal URLs, MCP server credentials, and system prompts that reveal architecture.

### Step 8: JavaScript Analysis (Always for SPAs)

```
For any SPA target (React, Angular, Vue, Next.js):
1. Extract all JavaScript file URLs from page source
2. Check each JS file for .map source maps (append .map to URL)
3. Run secret patterns against all JS files (API keys, tokens, internal URLs)
4. Extract API endpoints from fetch/axios/XHR calls
5. Map SPA routes for hidden/admin pages
6. Check for __NEXT_DATA__, window.__CONFIG__, Redux state exposure
7. Enumerate webpack chunks for lazy-loaded modules
```

**Why:** JavaScript recon yields $5K-$25K payouts for hidden endpoints and exposed credentials.

> **Full JavaScript analysis methodology, tools, secret patterns, and framework techniques:** See [reference/javascript-analysis.md](reference/javascript-analysis.md)

### Step 9: Synthesize

Combine all sources (fingerprint + auth recon + external recon). Map tech stack, assess security posture from headers, identify attack surface areas, prioritize by interest level. Generate next steps and route to appropriate skills.

---

## Reference Files

- [reference/javascript-analysis.md](reference/javascript-analysis.md) — JS bundle analysis, source maps, secret patterns, SPA route extraction (~266 lines)
- [reference/output-template.md](reference/output-template.md) — Full recon output template with state fixtures, R×S×E matrix, test cards (~170 lines)

---

## Cloud Asset Discovery During Recon

When recon reveals cloud indicators, route to **cloud-security** skill for deeper testing:

- **S3 buckets** — Subdomain patterns like `assets.example.com` → check CNAME for `s3.amazonaws.com`; test `https://example-assets.s3.amazonaws.com` for public listing
- **Azure blobs** — Look for `*.blob.core.windows.net` CNAMEs; test container enumeration
- **GCS objects** — `storage.googleapis.com/example-*` patterns from subdomain/cert data
- **Cloud metadata** — Internal IPs (`169.254.169.254`) in DNS records suggest cloud infrastructure → test for SSRF to metadata endpoints

### Subdomain Takeover Detection

When a subdomain's CNAME points to an unclaimed external service, anyone can claim it and serve content on the target's domain.

| Service | CNAME Pattern | Fingerprint (HTTP Response) |
|---------|--------------|---------------------------|
| GitHub Pages | `*.github.io` | "There isn't a GitHub Pages site here" |
| Heroku | `*.herokuapp.com` | "No such app" |
| AWS S3 | `*.s3.amazonaws.com` | "NoSuchBucket" |
| Azure | `*.azurewebsites.net` | "Error 404 - Web app not found" |
| Shopify | `shops.myshopify.com` | "Sorry, this shop is currently unavailable" |
| Fastly | `*.fastly.net` | "Fastly error: unknown domain" |
| Pantheon | `*.pantheonsite.io` | "404 error unknown site!" |
| Surge.sh | `*.surge.sh` | "project not found" |

**Testing procedure:**
1. During subdomain enumeration, flag any CNAME matching above patterns
2. Attempt to claim the service (e.g., create matching GitHub Pages repo)
3. Verify you can serve content on the target's subdomain
4. Severity: Medium ($500-$5K) — enables phishing, cookie theft, CSP bypass

---

## Advanced Recon Techniques

### Favicon Hash Enumeration

```
Use favicon hashes to discover related infrastructure:
1. Download favicon: curl -sL https://[target]/favicon.ico -o favicon.ico
2. Calculate Shodan/Censys hash (mmh3 or MD5)
3. Search Shodan: http.favicon.hash:[hash]
4. Search Censys: services.http.response.favicons.hashes=[hash]
```

**Why:** Shared favicons reveal infrastructure across different domains and cloud providers — staging servers, internal tools, forgotten instances.

### Origin IP Discovery via Certificate Transparency

```
Bypass WAF/CDN to find the real origin server:
1. Search crt.sh: curl -s "https://crt.sh/?q=%25.[domain]&output=json" | jq
2. Extract cert-associated IPs from historical DNS (SecurityTrails, PassiveTotal)
3. Check if origin responds: curl -sI -H "Host: [domain]" https://[origin-IP]
4. Compare responses from CDN vs direct-to-origin
```

**Why:** Direct origin access bypasses WAF rules, rate limits, and CDN-based security. Programs pay High for WAF bypass → origin access chains.

### Copyright Notice Mining

```
Find related, untested infrastructure via shared copyright notices:
1. Extract copyright text from target's footer/about/legal pages
2. Web search: "[exact copyright text]" -site:[target-domain]
3. Discover: sister companies, acquired properties, white-label deployments
```

**Why:** Related domains often share codebases but have weaker security configurations.

### Virtual Host Fuzzing

```
Discover hidden virtual hosts on the same IP:
1. Resolve target IP: dig +short [target]
2. Fuzz Host header: ffuf -u https://[IP] -H "Host: FUZZ.[domain]" -w subdomains.txt -fs [default-size]
3. Also try: internal, staging, admin, dev, test, api, beta, legacy
```

**Why:** Internal vhosts may lack authentication entirely.

### Method-Aware Route Discovery

Standard recon uses GET exclusively. Many routes only respond to POST/PUT/DELETE/PATCH and are invisible to GET-based crawling.

**Step 1: Discover methods from docs and JS (safe, no side effects):**
```
1. Extract methods from OpenAPI/Swagger spec (definitive source)
2. Grep JS bundles for fetch/axios calls with method: "POST"/"PUT"/"DELETE"
3. Check for X-HTTP-Method-Override or _method patterns in JS source
```

**Step 2: Probe with OPTIONS (safe, read-only):**
```
curl -sI -X OPTIONS https://[target]/api/[endpoint] → Check Allow header
Note: Allow header is often generic (framework default) — treat as hint, not proof
```

**Step 3: Test mutating methods safely (only against owned test data):**
```
Use YOUR OWN account's objects or clearly non-existent IDs:
1. curl -s -X POST https://[target]/api/[path] -d '{}' -H 'Content-Type: application/json'
2. curl -s -X PUT https://[target]/api/[path]/[YOUR-ID] -d '{}' -H 'Content-Type: application/json'
3. curl -s -X DELETE https://[target]/api/[path]/[YOUR-ID]
Compare: 405 (method not allowed) vs 200/403 (route exists) reveals real routing
```

**Why:** Hidden write endpoints behind GET-only routes are where BOLA/BFLA and mass assignment bugs live.

### ASN & CIDR Enumeration

```
Map the target's full network footprint:
1. Find ASN: dig TXT [target].origin.asn.cymru.com +short
2. Get CIDR ranges: whois -h whois.radb.net -- '-i origin AS[number]'
3. Web search: "[company name]" site:bgp.he.net
4. Reverse DNS on CIDR: for ip in range; dig -x $ip +short
5. Cross-reference: subdomains from crt.sh + IPs from ASN
```

---

## Recon Variations

- **Quick Fingerprint** — Headers, tech stack, security headers only
- **Full Domain Recon** — Everything: subdomains, paths, WAF, authenticated recon, full assessment
- **Subdomain Hunt** — Maximum subdomain enumeration, takeover checks
- **Pre-Hunt Recon** — Attack surface mapping tied to program scope

---

## CLI Execution Rules

Redirect large outputs to files. Use structured output flags: `nmap -oG -`, `ffuf -o results.json -of json`, `nuclei -j -o results.json -s critical,high`, `subfinder -o subs.txt`, `httpx -json -o live.json`, `dig +short`, `curl -sS -D -`. Cap brute-forcer results (`head -50`). Summarize >100-line outputs before loading into context.

---

## Related Skills

- **program-research** — Research the program before hunting | **cloud-security** — Deep cloud testing when recon reveals cloud infra
- **auth-testing** — Auth/authz testing after authenticated recon | **business-logic** — Business logic after workflow mapping
- **hunt-plan** (command) — Turn recon into a prioritized plan | Provide root domain + mention program for best results
