---
description: Triage your findings — rank bugs by reportability, estimate severity and payout, and decide what to report first
argument-hint: "[findings summary or 'review session']"
---

# /triage-findings

Rank and prioritize your bug bounty findings before writing reports. Helps you decide what to report first, what needs more work, and what to skip.

## Usage

```
/triage-findings <describe your findings or say "review session">
```

Triage findings: $ARGUMENTS

---

## How It Works

```
┌──────────────────────────────────────────────────────────────┐
│                    TRIAGE FINDINGS                            │
├──────────────────────────────────────────────────────────────┤
│  ✓ Classify each finding by vulnerability type and CWE       │
│  ✓ Estimate severity (CVSS range) for each finding           │
│  ✓ Assess reportability (strong PoC vs needs more work)      │
│  ✓ Flag duplicate risk based on common patterns              │
│  ✓ Rank findings by expected payout ÷ report effort          │
│  ✓ Recommend report order and next steps                     │
└──────────────────────────────────────────────────────────────┘
```

---

## What I Need From You

Describe what you found. For each finding, tell me:

**Required:**
- What you found (vulnerability type, location)
- What impact you observed

**Helps a lot:**
- Target program name
- Whether you have a working PoC
- How confident you are it's exploitable
- Whether it requires chaining with another bug

**Example input:**

```
I found three things on example.com:
1. IDOR on /api/users/{id} — can read other users' profiles including email
2. Reflected XSS on /search?q= — but CSP blocks inline scripts
3. Open redirect on /login?next= — not sure if it's in scope
```

---

## Output

```markdown
## Finding Triage: [Target/Program]

**Generated:** [Date]
**Findings reviewed:** [count]

---

### Triage Summary

| # | Finding | Severity | Reportability | Duplicate Risk | Priority |
|---|---------|----------|---------------|----------------|----------|
| 1 | [Short description] | [Critical/High/Medium/Low] | [Ready/Needs Work/Weak] | [High/Medium/Low] | [Report Now/Strengthen/Skip] |
| 2 | [Short description] | [Severity] | [Reportability] | [Duplicate Risk] | [Action] |

---

### Finding 1: [Title]

**Vulnerability:** [CWE-XXX: Name]
**Location:** [URL/endpoint]
**Estimated Severity:** [CVSS range] — [Justification]
**Reportability:** [Ready / Needs Work / Weak]

**Strengths:**
- [What makes this a solid finding]

**Weaknesses:**
- [What could get this N/A'd or downgraded]

**Before reporting:**
- [Step to strengthen the finding, if needed]

**Estimated payout:** [$range based on program + severity]

---

### Recommended Report Order

1. **[Finding X]** — [Why first: highest severity, best PoC, etc.]
2. **[Finding Y]** — [Why second]
3. **[Finding Z]** — [Why last or skip: needs work, low payout, high duplicate risk]

### Findings to Skip or Shelve

- **[Finding]** — [Why: out of scope, too weak, almost certainly duplicate]

### Next Steps

1. [What to do next — strengthen PoCs, check scope, report]
2. [Second action]
```

---

## Triage Criteria

### Reportability Assessment

| Level | Meaning | Criteria |
|-------|---------|----------|
| **Ready** | Submit now | Working PoC, clear impact, in-scope, strong repro steps |
| **Needs Work** | Strengthen first | Partial PoC, unclear impact, needs second account test, or chaining required |
| **Weak** | Consider skipping | Self-only impact, theoretical, likely duplicate, or borderline out-of-scope |

### Duplicate Risk Assessment

| Level | Indicators |
|-------|-----------|
| **High** | Common vulnerability on well-known endpoint, program has many disclosed reports of this type |
| **Medium** | Known vulnerability class but on less obvious feature, some disclosed reports nearby |
| **Low** | Unusual vulnerability, obscure endpoint, novel technique, or newly added scope |

### Priority Scoring

```
Priority = (Severity × Reportability) ÷ (Duplicate Risk × Report Effort)
```

- **Report Now:** High severity + ready + low duplicate risk
- **Strengthen:** Good severity but needs stronger PoC or impact demonstration
- **Skip:** Low severity + high duplicate risk, or out of scope

---

## Tips

1. **Batch your findings** — triage everything from a session before reporting any of it
2. **Report highest severity first** — in case lower findings are related and get marked as duplicates
3. **Don't sit on criticals** — if you have a high-severity finding that's ready, report immediately
4. **Chain before reporting** — two medium findings that chain into a critical are worth more than two separate mediums. Use `/chain` to document and score the attack path
5. **Check disclosed reports** — if program-research data is available, cross-reference before reporting
