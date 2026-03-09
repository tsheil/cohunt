---
name: report-writing
description: Write high-quality bug bounty reports that get triaged fast and paid well. Structures findings with clear impact, solid reproduction steps, and severity justification. Trigger with "write a report for", "draft a bug report", "help me report this vuln", "write up this finding", "report this bug to".
---

# Report Writing

Write bug bounty reports that get triaged fast, avoid N/A, and maximize payout. Good reporting is the difference between a bounty and a wontfix.

## What I Need From You

**Required:**
- What vulnerability you found (type, location)
- How to reproduce it (steps, payloads, requests)

**Helps a lot:**
- Target program and platform
- Impact you observed (what data was exposed, what action was possible)
- Screenshots or HTTP request/response pairs
- Your CVSS estimate (I'll validate or suggest adjustments)

**Optional:**
- Proof of concept code or script
- Video recording reference
- Related vulnerabilities or chaining potential

---

## Output Format

```markdown
# [Vulnerability Type] in [Feature/Endpoint] allows [Impact]

## Summary

[2-3 sentences: What the vulnerability is, where it exists, and what an attacker can achieve. Written for a triager who reads 50 reports a day.]

## Severity

**CVSS 3.1:** [score] ([vector string])
**CVSS 4.0:** [score] ([vector string]) *(include when program accepts CVSS 4.0)*
**Suggested Severity:** [Critical / High / Medium / Low]

### CVSS 3.1 Breakdown

| Metric | Value | Justification |
|--------|-------|---------------|
| Attack Vector | [Network/Adjacent/Local/Physical] | [Why] |
| Attack Complexity | [Low/High] | [Why] |
| Privileges Required | [None/Low/High] | [Why] |
| User Interaction | [None/Required] | [Why] |
| Scope | [Unchanged/Changed] | [Why] |
| Confidentiality | [None/Low/High] | [Why] |
| Integrity | [None/Low/High] | [Why] |
| Availability | [None/Low/High] | [Why] |

### CVSS 4.0 Breakdown *(when program accepts CVSS 4.0)*

CVSS 4.0 replaces Scope with Attack Requirements and splits impact into vulnerable system vs. subsequent systems:

| Metric | Value | Justification |
|--------|-------|---------------|
| Attack Vector (AV) | [Network/Adjacent/Local/Physical] | [Why] |
| Attack Complexity (AC) | [Low/High] | [Why] |
| Attack Requirements (AT) | [None/Present] | [Preconditions needed?] |
| Privileges Required (PR) | [None/Low/High] | [Why] |
| User Interaction (UI) | [None/Passive/Active] | [Why — Passive=click, Active=input] |
| Vuln Conf Impact (VC) | [None/Low/High] | [Confidentiality impact on the vulnerable system] |
| Vuln Integ Impact (VI) | [None/Low/High] | [Integrity impact on the vulnerable system] |
| Vuln Avail Impact (VA) | [None/Low/High] | [Availability impact on the vulnerable system] |
| Sub Conf Impact (SC) | [None/Low/High] | [Confidentiality impact on subsequent systems] |
| Sub Integ Impact (SI) | [None/Low/High] | [Integrity impact on subsequent systems] |
| Sub Avail Impact (SA) | [None/Low/High] | [Availability impact on subsequent systems] |

## Vulnerability Details

**Type:** [CWE-XXX: Name]
**Location:** [URL, endpoint, or feature]
**Parameter:** [Affected parameter, header, or input]
**Authentication:** [Required / Not required]

## Steps to Reproduce

1. [Exact step with specific URLs, parameters, values]
2. [Next step — no ambiguity, a triager should be able to follow blindly]
3. [Continue until vulnerability is demonstrated]

**HTTP Request:**
```
[Raw HTTP request if applicable]
```

**Response showing vulnerability:**
```
[Relevant response excerpt]
```

## Impact

[Concrete business impact. Not "an attacker could do bad things" but specifically what data is exposed, what actions become possible, how many users are affected, and what the business consequence is.]

**Affected Users:** [Scope of impact — all users, specific role, etc.]
**Data at Risk:** [What specifically can be accessed or modified]
**Attack Scenario:** [Realistic attack narrative — how would this be exploited in practice]

## Proof of Concept

[PoC script, curl command, or step-by-step with screenshots]

```bash
# Example PoC command
curl -X POST https://api.example.com/endpoint \
  -H "Authorization: Bearer ATTACKER_TOKEN" \
  -d '{"user_id": "VICTIM_ID"}'
```

## Remediation

**Recommended Fix:**
- [Specific technical recommendation]
- [Implementation guidance]

**Quick Mitigation:**
- [Temporary measure if full fix takes time]

## References

- [CWE Reference](https://cwe.mitre.org/data/definitions/XXX.html)
- [OWASP Reference](https://owasp.org/...)
- [Relevant advisory or write-up]
```

---

## Execution Flow

### Step 1: Gather Finding Details

```
From the user, collect:
1. Vulnerability type (XSS, IDOR, SSRF, SQLi, etc.)
2. Exact location (URL, endpoint, parameter)
3. Reproduction steps (what they did)
4. What they observed (response, behavior change)
5. Target program (if known)
```

If the user provides incomplete info, ask specifically for what's missing. Don't guess reproduction steps.

### Step 2: Classify and Score

```
1. Map to CWE identifier
2. Calculate CVSS 3.1 score with justification for each metric
3. If program accepts CVSS 4.0, also calculate CVSS 4.0:
   - Replace "Scope" with "Attack Requirements" (AT)
   - Split "User Interaction" into None/Passive/Active
   - Split impact into Vulnerable system (VC/VI/VA) and Subsequent (SC/SI/SA)
4. Determine severity tier
5. Check if severity matches program's bounty table (if known)
```

### Step 3: Write the Report

```
1. Title: [Vuln Type] in [Location] allows [Impact] — specific, scannable
2. Summary: 2-3 sentences a tired triager can understand immediately
3. Repro steps: Exact, numbered, no ambiguity
4. Impact: Business consequences, not technical jargon
5. PoC: Working proof, ideally a curl command or script
6. Remediation: Show you understand the fix, builds credibility
```

### Step 4: Review and Polish

```
1. Read the report as a triager — can I reproduce in under 5 minutes?
2. Check title isn't generic ("XSS found" → bad, "Stored XSS in comment field allows session hijacking via SVG upload" → good)
3. Verify CVSS justification makes sense
4. Ensure impact is concrete, not hypothetical hand-waving
5. Confirm CWE mapping is accurate
6. Check for common N/A triggers and address them preemptively
```

---

## Report Quality Checklist

Before submitting, verify:

- [ ] **Title** is specific and states impact (not just vuln type)
- [ ] **Summary** can be understood in 10 seconds
- [ ] **Repro steps** work if followed exactly by someone with no context
- [ ] **Impact** describes business consequences, not just technical possibility
- [ ] **CVSS** is justified metric-by-metric, not inflated
- [ ] **PoC** actually proves the vulnerability (not just "trust me")
- [ ] **No out-of-scope** items included (checked program scope)
- [ ] **No duplicated effort** signals (checked disclosed reports if available)

---

## Common Pitfalls (and How to Avoid Them)

### Reports That Get N/A'd

| Pitfall | Fix |
|---------|-----|
| "An attacker could theoretically..." | Show concrete impact with proof |
| Generic title ("XSS on website") | Specific: "Reflected XSS in /search via `q` parameter bypasses CSP" |
| Missing reproduction steps | Number every step, include exact URLs and payloads |
| Inflated severity | Justify every CVSS metric honestly |
| Out-of-scope vulnerability | Check scope before writing |
| Self-XSS reported as stored XSS | Demonstrate actual attack scenario |
| Missing impact context | Tie to user data, business operations, or compliance |

### Reports That Get Downgraded

| Pitfall | Fix |
|---------|-----|
| CVSS inflation | Be honest — reviewers recalculate anyway |
| "Critical" for info disclosure | Match severity to actual exploitability |
| No proof of cross-user impact | Demonstrate with two accounts if IDOR/access control |
| Theoretical chaining | Only claim chained impact if you've proven the chain |

---

## Platform-Specific Tips

### HackerOne
- Use markdown formatting — it renders well
- Reference their severity taxonomy in your CVSS justification
- Tag the correct asset from their scope
- Mention if you've read their policy on disclosure

### Bugcrowd
- Follow their VRT (Vulnerability Rating Taxonomy) for severity
- Use their priority scale (P1-P5) alongside CVSS
- Note the target from their scope list specifically

### Intigriti
- Follow their severity guidelines
- Reference the specific program rules
- Include the domain from their scope

---

## Report Variations

### Quick Report
For straightforward vulns with clear impact — concise, focused, no fluff.

### Detailed Report
For complex vulns, chains, or anything that needs extra context — full write-up with extensive proof.

### Chain Report
For vulnerabilities that combine — document each link in the chain separately, then show the combined impact.

### Regression Report
For previously fixed vulns that reappeared — reference the original report if possible.

---

## Quality Differentiation (The Anti-Slop Imperative)

The curl project shut down its HackerOne bug bounty in January 2026 — the first shutdown attributed to AI-generated report spam. Submission volume spiked 8x while confirmed-vulnerability rate dropped below 5%. Zero AI-only submissions found real vulnerabilities in 6 years. Programs are increasingly hostile toward low-quality reports. Quality is now a competitive moat.

**Self-Triage Before Submission:**

| Question | If No → |
|----------|---------|
| Can I reproduce this from my steps alone? | Don't submit — revise steps |
| Does the PoC actually demonstrate the vulnerability? | Don't submit — build a working PoC |
| Would the fix require a code change (not just config)? | Reconsider severity — config issues pay less |
| Is the impact concrete and measurable? | Rewrite impact with specific data/users/scope |
| Have I checked disclosed reports for this program? | Check first — duplicates waste everyone's time |
| Is the affected asset clearly in scope? | Verify scope — out-of-scope = instant N/A |

**AI Slop Red Flags (What Triagers Look For):**

Triagers now specifically watch for these AI-generated report patterns:
- Citing non-existent functions, endpoints, or CVEs
- Generic "impact assessment" paragraphs that could apply to any vulnerability
- Copy-paste from scanner output without manual verification
- Referencing code or behavior that doesn't match the target
- Overly polished language with no actual technical depth
- Vague reproduction steps that can't be followed
- Claims of "potential" impact without demonstrated proof

**How to Stand Out:**
- Include raw HTTP requests/responses (not paraphrased descriptions)
- Show you understand the application's business context
- Reference specific code paths or application behavior you observed
- Include a suggested fix that demonstrates you understand the root cause
- Keep the writing natural and direct — over-produced reports raise suspicion

---

## Tips for Better Reports

1. **Write for a tired triager** — they read dozens of reports daily, make yours easy
2. **Lead with impact** — title and summary should make severity obvious
3. **Be honest about severity** — inflated reports get downgraded and hurt reputation
4. **Include a working PoC** — a curl command beats a paragraph of explanation
5. **Check scope first** — don't waste your time or theirs on out-of-scope findings
6. **One vulnerability per report** — unless you're reporting a chain
7. **Disclose AI assistance** — if you used AI tools for discovery, note it in the methodology section; transparency builds trust and avoids "AI slop" suspicion
8. **For AI/LLM vulns** — clearly distinguish between prompt injection that stays in-session (often not reportable) vs injection that escapes context or affects other users (reportable)

---

## Related Skills

- **program-research** — Research the program before reporting (know the scope, rewards, and what gets N/A'd)
- **target-recon** — Gather technical context about the target for your report
- **hunt-plan** (command) — Plan your hunting session before finding bugs to report
