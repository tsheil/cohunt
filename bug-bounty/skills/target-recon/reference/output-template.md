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

| State ID | Role | Ready | Enter Via | Exit Via | Artifacts to Reuse | Notes |
|----------|------|-------|-----------|----------|-------------------|-------|
| **Account A** (active) | [role] | [Ready/Blocked] | — | — | session, API token | [Username, details] |
| **Account B** (active) | [role] | [Ready/Blocked] | — | — | session, API token | [Username, details] |
| `downgraded_from_paid` | free | [✅/❌/N/A] | [endpoint] | upgrade | old session, refresh token, export IDs, signed URLs | [details] |
| `pending_invite` | invited | [✅/❌/N/A] | [endpoint] | accept/revoke | invite token, accept URL | [details] |
| `suspended` | member | [✅/❌/N/A] | [endpoint] | restore | session cookie, API token, websocket | [details] |
| `expired_trial` | free | [✅/❌/N/A] | time/billing | upgrade | trial-only artifact IDs | [details] |
| `pending_approval` | user | [✅/❌/N/A] | [endpoint] | approve/reject | approval ID, linked resource | [details] |
| **OAST Collector** | — | [Ready/Not set up] | — | — | — | [Collaborator URL or interactsh ID] |
| **API Token Pair** | — | [Ready/N/A] | — | — | — | [Token A (role), Token B (role)] |

### Transition Endpoints

| Transition | Endpoint | Method | Notes |
|-----------|----------|--------|-------|
| Downgrade | [e.g. PUT /api/billing/subscription] | PUT | [Plan change details] |
| Suspend | [e.g. POST /api/admin/users/:id/suspend] | POST | [Admin action] |
| Revoke invite | [e.g. DELETE /api/invites/:id] | DELETE | [Invite lifecycle] |
| Cancel approval | [e.g. POST /api/approvals/:id/cancel] | POST | [Approval workflow] |
| Rotate API key | [e.g. POST /api/keys/rotate] | POST | [Credential lifecycle] |

### Role-Endpoint Matrix (Baseline = active state)

| Endpoint | Method | Free | Paid | Admin | Notes |
|----------|--------|------|------|-------|-------|
| /api/[path] | GET | ✓ | ✓ | ✓ | [Public data] |
| /api/[path] | DELETE | ✗ | ✗ | ✓ | [Test target: BOLA] |
| /api/[path] | PUT | ✗ | ✓ | ✓ | [Test target: privilege escalation] |

**Test targets:** [count] ✗ cells identified for auth bypass testing

### Role × State × Endpoint Delta Matrix

Only rows where state SHOULD change access — sparse overlay on the baseline matrix above:

| Card | Role | State | Endpoint | Method | Active Baseline | Expected In State | First Probe |
|------|------|-------|----------|--------|-----------------|-------------------|-------------|
| TC-01 | free | `downgraded_from_paid` | /api/exports/:id/download | GET | paid-only | deny | reuse pre-downgrade export ID with stale + fresh token |
| TC-02 | invited | `pending_invite` | /api/invites/:id/accept | POST | single-use | deny after revoke | replay invite token after admin revokes |
| TC-03 | member | `suspended` | /api/projects/:id | GET | ✓ own | deny all | retry existing session cookie after suspension |
| TC-04 | free | `expired_trial` | /api/ai/generate | POST | trial-only | deny | retry with expired-trial session |
| ... | ... | ... | ... | ... | ... | ... | ... |

### Tenant Boundary Map (if multi-tenant)

| Boundary | Isolation Method | Test Vector |
|----------|-----------------|-------------|
| [Org data] | [URL path / header / subdomain] | [Swap org_id between tenants] |
| [Shared resources] | [ID-based] | [Access cross-tenant IDs] |

---

## Proof-First Test Cards

Summary table (all 10), then expanded cards for top 3:

| # | Tuple (Role + State + Endpoint) | Bug Class | Payout Range | Source Matrix |
|---|--------------------------------|-----------|-------------|---------------|
| 1 | [role + state + endpoint] | [BOLA/BFLA/state-transition/etc.] | [$range] | [R×E / R×S×E / Tenant] |
| 2 | [role + state + endpoint] | [bug class] | [$range] | [source] |
| ... | ... | ... | ... | ... |

### Test Card TC-01: [Name]
- **Tuple:** [role + state + endpoint + method]
- **Boundary:** [What trust boundary is crossed]
- **Proof Type:** [before/after state, two-account comparison, cross-tenant]
- **Setup / Transition:** [What state to create and how]
- **Baseline:** [Expected safe behavior — capture this first]
- **Attack:** [Exact request modification]
- **Verify:** [What confirms the bug — specific response field, status code, data]
- **FP Check:** [What would make this a false positive — caching, intentional behavior, public data]
- **Evidence:** [What to capture — screenshots, response bodies, timestamps, two-account comparison]
- **Pivot If Hit:** [What to test next — sibling endpoints, escalation paths]
- **Pivot If Blocked:** [Alternative approach if initial probe fails]
- **Route To:** [auth-testing, business-logic, api-security, etc.]

### Test Card TC-02: [Name]
[Same format as TC-01]

### Test Card TC-03: [Name]
[Same format as TC-01]

---

## Sources
- [Source 1](URL)
- [Source 2](URL)
```
