---
name: hunt-plan
argument-hint: "[target-domain]"
description: Build a hunting plan for a target — combine recon and program intel into a prioritized action plan
arguments:
  - name: target
    description: "Target domain or program name (e.g., 'example.com', 'Shopify')"
    required: true
---

# /hunt-plan

Build a prioritized hunting plan by combining target recon and program intelligence into actionable test cases.

## Usage

```
/hunt-plan <target or program>
```

Build a hunting plan for: $ARGUMENTS

---

## How It Works

```
┌──────────────────────────────────────────────────────────────┐
│                        HUNT PLAN                             │
├──────────────────────────────────────────────────────────────┤
│  ✓ Step 0: What changed? (new features, patches, scope)      │
│  ✓ Gather or reuse target recon data                         │
│  ✓ Gather or reuse program research data                     │
│  ✓ Map scope boundaries (in/out)                             │
│  ✓ Prioritize by reward × likelihood ÷ competition           │
│  ✓ Generate specific test cases to run first                 │
│  ✓ Allocate time budget across targets                       │
└──────────────────────────────────────────────────────────────┘
```

> **Always start here:** Before planning tests, check what changed recently on the target — new features, patches, scope additions, API version bumps, acquisitions, infrastructure migrations. Fresh attack surface has the lowest duplicate risk and highest signal. Use program changelogs, blog posts, release notes, git commits, and Wayback Machine diffs.

---

## What I Need From You

**Option 1: Just a target**
Give me a domain or program name. I'll run recon and program research fresh, then build the plan.

**Option 2: After recon and research**
If you've already run `target-recon` and `program-research`, I'll use that data and go straight to plan generation.

**Option 3: Describe your situation**
Tell me what you know: "I want to hunt on Shopify. I'm good at API bugs and XSS. I have 10 hours this week."

---

## Output

```markdown
## Hunt Plan: [Target/Program]

**Generated:** [Date]
**Hunter Profile:** [Skills/focus if provided]
**Time Budget:** [If provided, otherwise "flexible"]

---

### Target Overview

| Field | Value |
|-------|-------|
| **Program** | [Name + Platform] |
| **Primary Domain** | [domain] |
| **Tech Stack** | [Key technologies] |
| **WAF/CDN** | [Protection layer] |
| **Attack Surface** | [# subdomains, APIs, apps] |

---

### Target Archetype

| Archetype | Signals | Primary Skills | Top Vuln Classes |
|-----------|---------|---------------|-----------------|
| **B2B SaaS** | Multi-tenant, org/team model, SCIM/SSO, seat limits | business-logic, auth-testing | Tenant isolation, IDOR, privilege escalation |
| **Consumer App** | User profiles, social features, payments, mobile | vuln-patterns, client-side-security, file-processing | XSS, IDOR, payment bypass, upload bugs |
| **API-First** | Developer docs, API keys, webhooks, rate limits | api-security, auth-testing | BOLA/BFLA, broken auth, rate limiting |
| **AI Product** | Chatbot, copilot, agent, MCP integrations | ai-hunting, vuln-patterns | Prompt injection, tool abuse, memory poisoning |
| **Infrastructure** | Network appliances, management consoles, VPN | vuln-patterns (infra ref) | Auth bypass, command injection, default creds |
| **Mobile-First** | iOS/Android apps, deep links, cert pinning | mobile-security, api-security | API backend vulns, deep link hijacking |

**This target's archetype:** [archetype] → drives skill selection and test priorities below.

---

### Setup & State Fixtures (REQUIRED before testing)

| Fixture | Status | Notes |
|---------|--------|-------|
| 2+ accounts (different roles) | □ | [e.g., free + paid, user + admin] |
| Webhook receiver | □ | [Burp Collaborator / interactsh / webhook.site] |
| Multi-tenant accounts | □ / N/A | [accounts in 2+ tenants if applicable] |
| Pending invite (not accepted) | □ | |
| Downgraded plan (premium→free) | □ | [check retained access] |
| API token pair (2 users) | □ | |
| Tools ready (Burp/proxy) | □ | |

**Blockers:** [List anything the hunter can't set up — adjust plan accordingly]

---

### Role-Endpoint Matrix

| Endpoint | Unauth | Free | Paid | Admin |
|----------|--------|------|------|-------|
| [endpoint 1] | [✓/✗] | [✓/✗] | [✓/✗] | [✓/✗] |
| [endpoint 2] | [✓/✗] | [✓/✗] | [✓/✗] | [✓/✗] |

Every ✗ cell is a test target — can the blocked role actually access it?

---

### Scope Boundaries

**In Scope — Hunt These:**
- [Asset 1] — [type] — [why it's interesting]
- [Asset 2] — [type] — [why it's interesting]

**Out of Scope — Avoid:**
- [Asset or vuln type]
- [Asset or vuln type]

**Gray Areas — Proceed With Caution:**
- [Anything ambiguous in the scope]

---

### Prioritized Hunt Targets

Ranked by: (Reward × Likelihood) ÷ (Automation Pressure × Duplicate Risk)

| Priority | Target | Vuln Type | Reward | Likelihood | Auto Pressure | Dup Risk | Rationale |
|----------|--------|-----------|--------|------------|---------------|----------|-----------|
| 1 | [endpoint/feature] | [vuln class] | [$range] | [H/M/L] | [H/M/L] | [H/M/L] | [Why this first] |
| 2 | [endpoint/feature] | [vuln class] | [$range] | [H/M/L] | [H/M/L] | [H/M/L] | [Why] |
| 3 | [endpoint/feature] | [vuln class] | [$range] | [H/M/L] | [H/M/L] | [H/M/L] | [Why] |

**Automation Pressure key:** HIGH = XBOW/Shannon/Codex Security can find this; MEDIUM = partially automatable; LOW = requires business context, multi-step chains, or human judgment. Deprioritize HIGH automation pressure targets — focus where human hunters have the edge.

---

### Proof-First Test Cards

#### Test Card 1: [Name]
- **Target:** [Exact URL/endpoint]
- **Baseline:** [Expected behavior for authorized user — capture this first]
- **Attack:** [Exact request with payload — what to change and why]
- **Verify:** [What confirms the bug — specific response field, status code, data]
- **FP Check:** [What would make this a false positive]
- **Evidence:** [What to capture — screenshot, response body, two-account comparison]
- **Pivot If Hit:** [What to test next — escalation, chain, variant]
- **Pivot If Blocked:** [Alternative approach or next test area]

#### Test Card 2: [Name]
[Same format]

#### Test Card 3: [Name]
[Same format]

---

### Time Budget

| Block | Focus | Time | Expected Outcome |
|-------|-------|------|-----------------|
| 1 | [High-priority targets] | [hours] | [What you should find/learn] |
| 2 | [Medium-priority exploration] | [hours] | [What you should find/learn] |
| 3 | [Deep dive on findings] | [hours] | [What you should find/learn] |

---

### Avoid These (Known Duplicates / N/A Patterns)
- [Common duplicate area]
- [Known N/A vuln type for this program]
- [Heavily tested feature]

---

### Tools Needed
- [Tool 1] — [what for]
- [Tool 2] — [what for]
```

---

## If Connectors Available

**Bug bounty platform connected:**
- I'll pull live scope (catches recent changes)
- Real-time reward tables
- Recent report activity to gauge competition

**Asset discovery connected:**
- Comprehensive subdomain and service inventory
- Port scan data for non-HTTP services
- Historical asset changes

**Vulnerability database connected:**
- CVE mapping for detected tech stack
- Recently disclosed vulns to test for
- Known exploit availability

---

## Tips

1. **Run recon first** — "recon example.com" then "/hunt-plan" gives better results than cold-starting
2. **Share your strengths** — "I'm good at XSS and IDOR" helps me prioritize relevant targets
3. **Set a time budget** — "I have 4 hours" forces realistic prioritization
4. **Mention past experience** — "I've hunted on this program before" avoids redundant basics
