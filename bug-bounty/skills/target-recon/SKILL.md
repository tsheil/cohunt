---
name: target-recon
description: Recon a web target — fingerprint tech stack, enumerate subdomains, detect WAF/CDN, map the attack surface. Works with curl/dig and web search. Trigger with "recon this target", "fingerprint example.com", "enumerate subdomains for", "what tech stack does X use", "map the attack surface of", "Google dork", "OSINT", "subdomain takeover", "ASN lookup", "CIDR range", "JavaScript analysis", "endpoint discovery", "what runs on". Use proactively when user mentions a target domain and hasn't done recon yet.
---

# Target Recon

Get a complete external view of any web target before hunting using curl, dig, and web search.

---

## Output Format

```markdown
# Target Recon: [domain]

**Generated:** [Date]
**Sources:** curl + dig + Web Search

---

## Quick Take

[2-3 sentences: What this target is, notable findings, initial attack surface assessment]

---

## Target Profile

| Field | Value |
|-------|-------|
| **Domain** | [domain] |
| **IP Address** | [IP(s)] |
| **Hosting** | [Provider / ASN] |
| **CDN** | [Detected CDN or "None detected"] |
| **WAF** | [Detected WAF or "None detected"] |

---

## Technology Stack

| Layer | Technology | Version | Notes |
|-------|-----------|---------|-------|
| **Server** | [e.g. nginx] | [version] | [From Server header] |
| **Framework** | [e.g. Next.js] | [version] | [Detection method] |
| **CMS** | [e.g. WordPress] | [version] | [If applicable] |
| **Language** | [e.g. PHP, Node] | — | [Inferred from] |
| **JS Libraries** | [e.g. React, jQuery] | [version] | [From page source] |

---

## Security Headers

| Header | Status | Value |
|--------|--------|-------|
| **Strict-Transport-Security** | [Present/Missing] | [Value] |
| **Content-Security-Policy** | [Present/Missing] | [Value] |
| **X-Frame-Options** | [Present/Missing] | [Value] |
| **X-Content-Type-Options** | [Present/Missing] | [Value] |
| **X-XSS-Protection** | [Present/Missing] | [Value] |
| **Permissions-Policy** | [Present/Missing] | [Value] |
| **CORS** | [Open/Restricted/None] | [Value] |

**Assessment:** [Summary — well-hardened, some gaps, or notable misconfigurations]

---

## Subdomain Inventory

| Subdomain | IP | Status | Server | Notes |
|-----------|----|--------|--------|-------|
| [sub.domain] | [IP] | [HTTP status] | [Server] | [Takeover risk, interesting, etc.] |

**Total:** [count] subdomains found
**Live:** [count] responding
**Notable:** [Any takeover candidates, staging environments, API endpoints]

---

## Interesting Paths

| Path | Status | Notes |
|------|--------|-------|
| /robots.txt | [status] | [Disallowed paths of interest] |
| /sitemap.xml | [status] | [Interesting entries] |
| /.well-known/security.txt | [status] | [Bug bounty info if present] |
| /api/ | [status] | [API discovery] |
| /.git/ | [status] | [Exposed git repo?] |
| /wp-admin/ | [status] | [CMS admin panels] |

---

## WAF/CDN Analysis

| Check | Result |
|-------|--------|
| **CDN Provider** | [Cloudflare, Akamai, etc. or None] |
| **WAF Detected** | [Yes/No — provider if known] |
| **Rate Limiting** | [Observed behavior] |
| **Origin IP** | [If discoverable behind CDN] |

---

## Attack Surface Summary

### High Interest
- [Finding and why it matters]
- [Finding and why it matters]

### Medium Interest
- [Finding worth investigating]

### Next Steps
1. [Recommended next action]
2. [Second priority]
3. [Third priority]

---

## Sources
- [Source 1](URL)
- [Source 2](URL)
```

---

## Execution Flow

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

### Step 3: Subdomain Enumeration (Always)

```
Run these lookups:
1. Web search: "site:crt.sh [domain]" or curl crt.sh API
2. dig [domain] ANY → DNS records
3. dig ns [domain] → Name servers
4. dig mx [domain] → Mail servers
5. For each discovered subdomain: curl -sI → Status and server
6. Check for subdomain takeover indicators (CNAME to unclaimed services)
```

**Extract:**
- Subdomain inventory from certificate transparency
- DNS record types and values
- Live vs. dead subdomains
- Takeover candidates (dangling CNAMEs)
- Interesting subdomains (staging, api, admin, dev, internal)

### Step 4: WAF/CDN Detection (Always)

```
Check for WAF/CDN:
1. Inspect response headers for CDN signatures
2. Check for WAF-specific headers or cookies
3. Compare direct IP vs domain response
4. Test with common WAF trigger payloads in User-Agent
5. Note rate limiting behavior
```

### Step 5: Synthesize

```
1. Combine all sources
2. Map technology stack
3. Assess security posture from headers
4. Identify attack surface areas
5. Prioritize targets by interest level
6. Generate next steps
```

### Step 6: Google Dorking & OSINT

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
9. site:github.com "[target]" ".env" OR "mcp.json" OR ".cursorrules" → AI config + secret leaks in repos
```

### Step 6b: AI/MCP Configuration Artifact Discovery

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

**Why:** AI config files are the new `.env` — developers commit them with embedded API keys, internal URLs, MCP server credentials, and system prompts that reveal architecture. MCP configs often contain OAuth tokens or PATs with broad scopes. `.cursorrules` and `CLAUDE.md` files expose internal business logic that aids further exploitation.

### Step 7: JavaScript Analysis (Always for SPAs)

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

**Why this matters:** JavaScript recon is yielding $5K-$25K payouts for hidden endpoints and exposed credentials. Every SPA ships its attack surface to the browser.

> **Full JavaScript analysis methodology, tools, secret patterns, and framework techniques:** See [reference/javascript-analysis.md](reference/javascript-analysis.md)

---

## Reference Files

This skill uses progressive disclosure. Detailed reference material is available on demand:

| File | Contents | Lines |
|------|----------|-------|
| [reference/javascript-analysis.md](reference/javascript-analysis.md) | JS bundle analysis, source map exploitation, secret hunting (14 patterns), SPA route extraction, webpack chunk enumeration, framework-specific techniques | ~200 |

**Quick search** — find specific JS analysis patterns:
```
grep -n "source map\|sourceMappingURL\|\.map" reference/javascript-analysis.md
grep -n "secret\|API_KEY\|token\|credential\|password" reference/javascript-analysis.md
grep -n "webpack\|chunk\|bundle\|build" reference/javascript-analysis.md
grep -n "React\|Angular\|Vue\|Next.js" reference/javascript-analysis.md
grep -n "endpoint\|route\|path\|/api/" reference/javascript-analysis.md
```

---

## Advanced Recon Techniques

### Favicon Hash Enumeration

```
Use favicon hashes to discover related infrastructure hidden behind different domains:
1. Download favicon: curl -sL https://[target]/favicon.ico -o favicon.ico
2. Calculate Shodan/Censys hash (mmh3 or MD5)
3. Search Shodan: http.favicon.hash:[hash]
4. Search Censys: services.http.response.favicons.hashes=[hash]
5. Discover: staging servers, internal tools, acquisition targets, forgotten instances
```

**Why:** Shared favicons reveal infrastructure belonging to the same organization across different domains, IPs, and cloud providers. Often exposes forgotten or unprotected instances running the same software.

### Origin IP Discovery via Certificate Transparency

```
Bypass WAF/CDN to find the real origin server:
1. Search crt.sh: curl -s "https://crt.sh/?q=%25.[domain]&output=json" | jq
2. Extract all cert-associated IPs from historical DNS (SecurityTrails, PassiveTotal)
3. Check if origin responds directly: curl -sI -H "Host: [domain]" https://[origin-IP]
4. Compare responses from CDN vs direct-to-origin
5. Look for: staging certs, internal hostnames, pre-CDN DNS records
```

**Why:** Direct origin access bypasses WAF rules, rate limits, and CDN-based security. Many programs pay High for WAF bypass → origin access chains.

### Copyright Notice Mining

```
Find related, untested infrastructure via shared copyright notices:
1. Extract copyright text from target's footer/about/legal pages
2. Web search: "[exact copyright text]" -site:[target-domain]
3. Discover: sister companies, acquired properties, white-label deployments
4. Check if discovered hosts share the same tech stack and vulnerabilities
```

**Why:** Related domains often share codebases but have weaker security configurations. White-label deployments frequently lack the hardening of the primary product.

### Virtual Host Fuzzing

```
Discover hidden virtual hosts on the same IP:
1. Resolve target IP: dig +short [target]
2. Fuzz Host header: ffuf -u https://[IP] -H "Host: FUZZ.[domain]" -w subdomains.txt -fs [default-size]
3. Also try: internal, staging, admin, dev, test, api, beta, legacy
4. Check for: different responses, authentication prompts, admin panels
```

**Why:** Virtual hosts on the same IP often share the same server but have different access controls. Internal vhosts may lack authentication entirely.

### ASN & CIDR Enumeration

```
Map the target's full network footprint:
1. Find ASN: dig TXT [target].origin.asn.cymru.com +short → Get ASN number
2. Get CIDR ranges: whois -h whois.radb.net -- '-i origin AS[number]' → All IP ranges
3. Web search: "[company name]" site:bgp.he.net → BGP routing info
4. Reverse DNS on CIDR: for ip in range; dig -x $ip +short → Find hostnames
5. Cross-reference: subdomains from crt.sh + IPs from ASN → Complete IP-to-domain mapping
```

**Why:** Organizations often have assets across multiple CIDR ranges that aren't discoverable via DNS alone. ASN enumeration reveals forgotten infrastructure, acquired company assets, and internal systems with public IPs.

---

## Recon Variations

### Quick Fingerprint
Focus on: Headers, tech stack, security headers only

### Full Domain Recon
Focus on: Everything — subdomains, paths, WAF, full assessment

### Subdomain Hunt
Focus on: Maximum subdomain enumeration, takeover checks

### Pre-Hunt Recon
Focus on: Attack surface mapping tied to program scope

---

## CLI Execution Rules

When running external recon tools via shell, follow these rules to avoid blowing up the context window with raw output:

| Tool | Use This Flag | Why |
|------|--------------|-----|
| `nmap` | `-oG -` (greppable) or `-oX -` (XML) | Raw stdout is verbose; greppable output is parseable |
| `ffuf` | `-o results.json -of json` | Parse JSON file instead of streaming stdout |
| `nuclei` | `-j -o results.json` | JSON output; filter by severity with `-s critical,high` |
| `subfinder` | `-o subs.txt` | Write to file, read selectively |
| `httpx` | `-json -o live.json` | Structured output for tech fingerprinting |
| `amass` | `-json output.json` | Structured enumeration results |
| `dig` | `+short` | Minimal output for DNS lookups |
| `curl` | `-sS -D -` | Silent with headers; pipe through `head -100` for large responses |

**General rules:**
- Always redirect large outputs to files and read selectively — never dump raw stdout into context
- For brute-forcers, cap results: `head -50` or use tool-native limits
- Filter by severity/confidence before loading results — not everything is relevant
- When a tool returns >100 lines, summarize findings instead of including raw output

---

## Tips for Better Recon

1. **Provide the root domain** — "recon example.com" casts a wider net than "recon www.example.com"
2. **Mention the program** — "recon example.com for their HackerOne program" helps scope the work
3. **Ask for depth** — "deep recon" vs "quick fingerprint" adjusts thoroughness
4. **Chain with hunt-plan** — Run recon first, then `/hunt-plan` to turn findings into action

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

## Authenticated Recon

External recon finds the perimeter. Authenticated recon maps the real attack surface — the endpoints, roles, and data flows that only exist behind login. Most high-severity bugs (IDOR, privilege escalation, business logic) live here.

### Per-Role Endpoint Mapping

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

5. Test every ✗ cell — can the blocked role actually access it?
```

### Hidden Route Discovery

```
□ Predictable admin paths — /admin, /internal, /debug, /status, /_admin
□ API version exploration — if /api/v2/users exists, try /api/v1/users, /api/v3/users
□ Framework-specific routes — /actuator (Spring Boot), /__debug__ (Django), /elmah (ASP.NET)
□ GraphQL introspection — query { __schema { types { name fields { name } } } }
□ Swagger/OpenAPI endpoints — /swagger.json, /openapi.yaml, /api-docs
□ Sitemap and robots.txt — may reference internal paths
□ JavaScript bundle analysis — see reference/javascript-analysis.md
□ Error-based discovery — send invalid requests to trigger stack traces with route info
□ AI/MCP config files — /.mcp.json, /.claude/, /.cursor/mcp.json, /.amazonq/, /CLAUDE.md, /AGENTS.md
```

### Multi-Tenant Isolation Probing

```
□ Tenant ID in URL — /orgs/{tenant_id}/resource → swap tenant IDs
□ Tenant ID in headers — X-Tenant-ID, X-Organization-ID → swap values
□ Subdomain-based tenants — company1.app.com vs company2.app.com → cross-tenant requests
□ Shared resources — /shared/files/{id} → IDs may span tenants
□ Admin endpoints — tenant admin vs platform admin authorization boundaries
□ API key scoping — does API key for tenant A work against tenant B endpoints?
```

### Webhook & Integration Mapping

```
□ Outgoing webhooks — where does the app send data? Test for SSRF via webhook URL
□ Incoming webhooks — /webhooks/*, /callbacks/* — test for signature bypass
□ OAuth integrations — which third-party apps are connected? Test token scope
□ Email notifications — do they include sensitive data? HTML injection?
□ Export features — CSV/PDF/Excel export — test for formula injection, IDOR on exports
□ Import features — CSV/file upload import — test for XXE, SSRF, path traversal
```

## Related Skills

- **program-research** — Research the bug bounty program before hunting
- **cloud-security** — Deep cloud misconfig testing when recon reveals cloud infrastructure
- **hunt-plan** (command) — Turn recon into a prioritized hunting plan
- **auth-testing** — Deep dive on authentication and authorization testing after authenticated recon
