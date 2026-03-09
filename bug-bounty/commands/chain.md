---
name: chain
argument-hint: "[finding1 + finding2 + ...]"
description: Document and score a vulnerability chain — combine multiple findings into a higher-severity attack path with proper CVSS scoring and a single cohesive report
arguments:
  - name: findings
    description: "Describe the individual vulnerabilities in your chain (e.g., 'open redirect on /login + SSRF on /preview + IMDS access')"
    required: true
---

# /chain

Document a multi-step attack chain — map individual findings into a cohesive attack path, score the chain correctly, and produce a single report that maximizes payout.

## Usage

```
/chain <describe your individual findings and how they connect>
```

Chain: $ARGUMENTS

---

## Why Chain

Chaining is how medium-severity findings become criticals. Two individually weak bugs that combine into account takeover or data exfiltration earn dramatically more than reporting them separately. But chains are also where reports fail hardest — a confusing chain gets N/A'd even when the bugs are real.

---

## How It Works

```
┌──────────────────────────────────────────────────────────────┐
│                     VULNERABILITY CHAIN                       │
├──────────────────────────────────────────────────────────────┤
│  ✓ Map individual findings into attack flow                  │
│  ✓ Identify the critical path (minimal steps to max impact)  │
│  ✓ Score the chain as a single finding (CVSS 3.1 + 4.0)     │
│  ✓ Score each link independently for comparison              │
│  ✓ Produce chain diagram showing attack flow                 │
│  ✓ Generate a single cohesive report                         │
│  ✓ Flag weak links that could break the chain                │
└──────────────────────────────────────────────────────────────┘
```

---

## What I Need From You

**Minimum:** Describe each vulnerability and how they connect.

**Better:** Include endpoints, payloads, and the order of exploitation.

**Best:** Full reproduction details for each link plus the end-to-end attack scenario.

**Example inputs:**

```
/chain Open redirect on /oauth/authorize redirects to attacker domain → steals OAuth code → exchanges for access token → full account takeover
```

```
/chain I found three things: 1) SSRF on /api/preview?url= 2) Cloud metadata accessible at 169.254.169.254 3) AWS credentials in metadata response. Together: SSRF → metadata → AWS key exfil
```

```
/chain Self-XSS in profile bio + CSRF on profile update = stored XSS affecting any user who views the profile
```

---

## Execution Flow

### Step 1: Parse the Chain

```
For each finding the user describes:
1. Identify vulnerability type and CWE
2. Identify location (endpoint, parameter, feature)
3. Identify what it produces (output/artifact used by next step)
4. Identify what it requires (input/precondition from prior step)

Map the dependency graph — which finding feeds into which.
```

### Step 2: Map the Attack Flow

```
Produce an attack flow showing:
1. Entry point — where the attack begins
2. Each link — what the attacker does at each step
3. Transitions — what output from step N feeds step N+1
4. Terminal impact — what the attacker achieves at the end

Format as a numbered chain:
  [Entry] → [Link 1: vuln type] → [Link 2: vuln type] → ... → [Impact]
```

### Step 3: Identify the Critical Path

```
If there are multiple possible paths:
1. Find the shortest path to maximum impact
2. Find the path with the most reliable links (least likely to fail)
3. Recommend which path to report (usually: shortest + most reliable)

Flag any optional links that increase impact but aren't required.
```

### Step 4: Score the Chain

```
Score the CHAIN as a single finding (not individual links):

CVSS 3.1:
- Attack Vector: Based on the entry point (usually Network)
- Attack Complexity: Based on the HARDEST step in the chain
  - Multiple steps = at least "High" unless each step is trivial
- Privileges Required: Based on what's needed to START the chain
- User Interaction: Based on whether any step requires victim action
- Scope: Changed if the chain crosses security boundaries
- CIA: Based on the TERMINAL impact, not intermediate steps

CVSS 4.0:
- Follow CVSS 4.0 scoring with Attack Requirements metric
- Consider the full chain for Subsequent system impact

Also score each link independently for comparison.
```

### Step 5: Assess Chain Reliability

```
For each link, assess:
- Reliability: Does it work every time or depend on timing/conditions?
- Detectability: Would WAF/IDS/monitoring catch this step?
- Prerequisites: What must be true for this step to work?
- Alternatives: If this link breaks, is there another path?

Flag weak links — the chain is only as strong as its weakest link.
```

### Step 6: Generate Chain Report

```
Produce a single report with:
1. Title that captures the full chain impact
2. Executive summary of the chain (3 sentences max)
3. Chain diagram (text-based flow)
4. Individual link documentation
5. End-to-end reproduction steps
6. Combined CVSS scoring with justification
7. Impact assessment based on terminal state
8. Why it's a chain (not just multiple separate bugs)
```

---

## Output Format

```markdown
## Vulnerability Chain: [Terminal Impact] via [Key Technique]

**Chain Length:** [N] links
**Entry Point:** [Where it starts]
**Terminal Impact:** [What the attacker achieves]

---

### Chain Diagram

```
[Attacker]
  → [Link 1: Vuln Type on Location]
    → produces: [what it gives the attacker]
  → [Link 2: Vuln Type on Location]
    → produces: [what it gives the attacker]
  → [Terminal: Impact Description]
```

---

### Chain Scoring

| Metric | Chain Score | Justification |
|--------|------------|---------------|
| **CVSS 3.1** | [Score] | [Vector string] |
| **CVSS 4.0** | [Score] | [Vector string] |
| **Estimated Severity** | [Critical/High/Medium/Low] | Based on terminal impact |

**Individual Link Scores (for reference):**

| Link | Vuln Type | CVSS 3.1 | Alone vs. In Chain |
|------|-----------|----------|-------------------|
| 1 | [Type] | [Score] | [Lower alone — explain why chain elevates it] |
| 2 | [Type] | [Score] | [Same] |

**Chain Multiplier:** Individual links score [X] alone but [Y] together because [reason].

---

### Link 1: [Vulnerability Type]

**CWE:** CWE-XXX
**Location:** [endpoint/feature]
**What it does:** [description]
**What it produces:** [output that feeds next link]
**Reliability:** [High/Medium/Low]
**Evidence:** [HTTP request/response or PoC]

### Link 2: [Vulnerability Type]

**CWE:** CWE-XXX
**Location:** [endpoint/feature]
**Requires from Link 1:** [what input it needs]
**What it does:** [description]
**What it produces:** [output that feeds next link]
**Reliability:** [High/Medium/Low]
**Evidence:** [HTTP request/response or PoC]

[... additional links ...]

---

### End-to-End Reproduction

**Prerequisites:**
- [Account types needed]
- [Tools needed]
- [Environmental conditions]

**Steps:**

1. [First action — link 1 entry point]
   - Request: [exact HTTP request]
   - Response: [what to observe]
   - Output: [what this gives you for the next step]

2. [Second action — using output from step 1]
   - Request: [exact HTTP request]
   - Response: [what to observe]
   - Output: [what this gives you]

[... continue for each link ...]

N. [Final action — demonstrating terminal impact]
   - Result: [what the attacker has achieved]

---

### Impact Assessment

**Terminal impact:** [Concrete description — "attacker reads all user emails" not "data breach possible"]
**Affected scope:** [How many users/resources]
**Attack scenario:** [Realistic attack narrative]
**Why it's a chain:** [Why these bugs must be reported together — the individual links don't achieve this impact alone]

---

### Weak Links & Risk Factors

| Link | Risk | Mitigation |
|------|------|-----------|
| [Link N] | [What could break it] | [How to address in report] |

---

### Remediation

**Fix the chain at the weakest point:**
1. [Most impactful fix — which link to break]
2. [Defense in depth — additional fixes]
```

---

## Common Chain Patterns

| Pattern | Links | Typical Impact |
|---------|-------|----------------|
| **SSRF → Cloud metadata → Key exfil** | SSRF + cloud misconfiguration | Critical — AWS/GCP credential theft |
| **Open redirect → OAuth token theft** | Open redirect + OAuth misconfiguration | High/Critical — account takeover |
| **Self-XSS + CSRF → Stored XSS** | Self-only XSS + missing CSRF token | High — victim-triggered stored XSS |
| **IDOR + info disclosure → targeted attack** | IDOR + PII exposure | High — targeted user compromise |
| **Race condition → duplicate resource** | TOCTOU + business logic | Medium-High — financial impact |
| **XXE → SSRF → internal access** | XXE + network access | Critical — internal network pivot |
| **SQL injection → file read → RCE** | SQLi + FILE privilege | Critical — remote code execution |
| **Subdomain takeover → cookie theft** | Dangling DNS + cookie scope | High — session hijacking |

---

## When NOT to Chain

- **Each bug has high severity alone** → Report separately, earn two bounties
- **The connection is speculative** → "Could potentially lead to..." is not a chain
- **Bugs are on different assets** → Unless the chain requires both, report separately
- **One link is out of scope** → The chain is only valid if all links are in scope

---

## Tips

1. **Score the chain, not the links** — A chain of three lows that results in account takeover is a critical, not three lows
2. **Document every transition** — The handoff between links is where triagers get confused. Be explicit about what each step produces and what the next step consumes
3. **Show it's not theoretical** — Every link must have a working PoC. A chain with one unproven link is speculative
4. **Lead with the end result** — Title: "Account Takeover via OAuth + Open Redirect" not "Open Redirect on /login"
5. **Flag what breaks it** — Being upfront about weak links builds trust and prevents triager skepticism
6. **Consider reporting order** — If in doubt whether to chain, ask: would each link alone get a bounty? If yes, consider reporting separately
