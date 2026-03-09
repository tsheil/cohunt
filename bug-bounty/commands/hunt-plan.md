---
name: hunt-plan
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
│  ✓ Gather or reuse target recon data                         │
│  ✓ Gather or reuse program research data                     │
│  ✓ Map scope boundaries (in/out)                             │
│  ✓ Prioritize by reward × likelihood ÷ competition           │
│  ✓ Generate specific test cases to run first                 │
│  ✓ Allocate time budget across targets                       │
└──────────────────────────────────────────────────────────────┘
```

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

Ranked by: (Reward Potential × Vulnerability Likelihood) ÷ Competition Level

| Priority | Target | Vuln Type | Reward | Likelihood | Competition | Rationale |
|----------|--------|-----------|--------|------------|-------------|-----------|
| 1 | [endpoint/feature] | [vuln class] | [$range] | [H/M/L] | [H/M/L] | [Why this first] |
| 2 | [endpoint/feature] | [vuln class] | [$range] | [H/M/L] | [H/M/L] | [Why] |
| 3 | [endpoint/feature] | [vuln class] | [$range] | [H/M/L] | [H/M/L] | [Why] |

---

### First Test Cases

Start with these specific tests:

**Test 1: [Name]**
- Target: [URL or feature]
- Vuln type: [CWE]
- Steps: [Concrete testing steps]
- What to look for: [Expected behavior if vulnerable]
- Time estimate: [minutes]

**Test 2: [Name]**
- Target: [URL or feature]
- Vuln type: [CWE]
- Steps: [Concrete testing steps]
- What to look for: [Expected behavior if vulnerable]
- Time estimate: [minutes]

**Test 3: [Name]**
- Target: [URL or feature]
- Vuln type: [CWE]
- Steps: [Concrete testing steps]
- What to look for: [Expected behavior if vulnerable]
- Time estimate: [minutes]

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
