# Target Recon Output Template

Use this template when generating recon reports. Fill in each section based on findings.

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

---

## Authenticated Recon (include when accounts are available)

### State Fixtures

| Fixture | Status | Notes |
|---------|--------|-------|
| **Account A** (role: [role]) | [Ready/Blocked] | [Username, role details] |
| **Account B** (role: [role]) | [Ready/Blocked] | [Username, role details] |
| **OAST Collector** | [Ready/Not set up] | [Collaborator URL or interactsh ID] |
| **Pending invite** | [Created/N/A] | [Details] |
| **Downgraded account** | [Created/N/A] | [Was premium, now free] |
| **API tokens** | [Ready/N/A] | [Token A (role), Token B (role)] |

### Role-Endpoint Matrix

| Endpoint | Method | Free | Paid | Admin | Notes |
|----------|--------|------|------|-------|-------|
| /api/[path] | GET | ✓ | ✓ | ✓ | [Public data] |
| /api/[path] | DELETE | ✗ | ✗ | ✓ | [Test target: BOLA] |
| /api/[path] | PUT | ✗ | ✓ | ✓ | [Test target: privilege escalation] |

**Test targets:** [count] ✗ cells identified for auth bypass testing

### Tenant Boundary Map (if multi-tenant)

| Boundary | Isolation Method | Test Vector |
|----------|-----------------|-------------|
| [Org data] | [URL path / header / subdomain] | [Swap org_id between tenants] |
| [Shared resources] | [ID-based] | [Access cross-tenant IDs] |

---

## First 10 Test Cases

| # | Target | Test | Expected Vuln | Route To |
|---|--------|------|--------------|----------|
| 1 | [endpoint] | [what to do] | [vulnerable response] | [skill] |
| 2 | [endpoint] | [what to do] | [vulnerable response] | [skill] |
| ... | ... | ... | ... | ... |

---

## Sources
- [Source 1](URL)
- [Source 2](URL)
```
