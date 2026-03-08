---
name: target-recon
description: Recon a web target — fingerprint tech stack, enumerate subdomains, detect WAF/CDN, map the attack surface. Works standalone with curl/dig and web search, supercharged with asset discovery connectors. Trigger with "recon this target", "fingerprint example.com", "enumerate subdomains for", "what tech stack does X use", "map the attack surface of".
---

# Target Recon

Get a complete external view of any web target before hunting. This skill always works with curl, dig, and web search, and gets significantly better with asset discovery connectors.

## Connectors (Optional)

Connect your tools to supercharge this skill:

| Connector | What It Adds |
|-----------|--------------|
| **Asset discovery** | Full subdomain inventory, open ports, service banners, historical data |
| **DNS intelligence** | DNS history, zone data, related domains, passive DNS |
| **Vulnerability database** | Known CVEs mapped to detected technology stack |

> **No connectors?** No problem. curl, dig, and web search provide solid recon for any target.

---

## Output Format

```markdown
# Target Recon: [domain]

**Generated:** [Date]
**Sources:** curl + dig + Web Search [+ Asset Discovery] [+ DNS Intel]

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

### Step 5: Asset Discovery (If Connected)

```
If asset discovery tools available:
1. Full subdomain enumeration → Comprehensive inventory
2. Port scanning → Open services beyond HTTP
3. Service detection → Banner grabbing
4. Historical data → Previously seen subdomains/services
```

### Step 6: Synthesize

```
1. Combine all sources
2. Map technology stack
3. Assess security posture from headers
4. Identify attack surface areas
5. Prioritize targets by interest level
6. Generate next steps
```

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

## Tips for Better Recon

1. **Provide the root domain** — "recon example.com" casts a wider net than "recon www.example.com"
2. **Mention the program** — "recon example.com for their HackerOne program" helps scope the work
3. **Ask for depth** — "deep recon" vs "quick fingerprint" adjusts thoroughness
4. **Chain with hunt-plan** — Run recon first, then `/hunt-plan` to turn findings into action

---

## Related Skills

- **program-research** — Research the bug bounty program before hunting
- **hunt-plan** (command) — Turn recon into a prioritized hunting plan
