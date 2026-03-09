---
description: Write a submission-ready bug bounty report — structured, scored, and formatted for the platform
argument-hint: "<describe your finding>"
---

# /write-report

Write a complete, submission-ready bug bounty report from a finding description. Produces a structured report with clear title, reproduction steps, CVSS scoring, impact assessment, and remediation — formatted for the target platform.

## Usage

```
/write-report <describe your finding>
```

Write report for: $ARGUMENTS

---

## How It Works

```
┌──────────────────────────────────────────────────────────────┐
│                      WRITE REPORT                             │
├──────────────────────────────────────────────────────────────┤
│  ✓ Structure finding into submission-ready format            │
│  ✓ Generate specific, scannable title                        │
│  ✓ Write unambiguous step-by-step reproduction               │
│  ✓ Calculate and justify CVSS 3.1 + 4.0 scores               │
│  ✓ Assess business impact concretely                         │
│  ✓ Map to CWE identifier                                     │
│  ✓ Suggest remediation with code examples                    │
│  ✓ Format for target platform (HackerOne, Bugcrowd, etc.)   │
└──────────────────────────────────────────────────────────────┘
```

---

## What I Need From You

**Minimum:** What you found and where you found it.

**Better:** Include the HTTP request/response, the payload, what happened, and the target program.

**Best:** All of the above plus screenshots, a working PoC, and which platform to format for.

**Example inputs:**

```
/write-report IDOR on api.example.com/users/{id} — changing the ID returns other users' profiles including email and phone. For Shopify on HackerOne.
```

```
/write-report Found reflected XSS on example.com/search?q=<payload>. CSP is misconfigured — unsafe-inline is allowed. Have a working PoC that steals cookies.
```

```
/write-report I can bypass 2FA on the mobile app by intercepting the API call and replaying the OTP verification response for a different user.
```

If you give me incomplete info, I'll ask for what's missing before writing. I won't guess reproduction steps.

---

## Execution Flow

### Step 1: Understand the Finding

```
From the user, extract:
1. Vulnerability type (XSS, IDOR, SSRF, etc.)
2. Location (URL, endpoint, parameter, feature)
3. What the attacker can achieve (data access, action, escalation)
4. Reproduction details (steps, payloads, requests)
5. Target program and platform (if known)

If anything critical is missing → ASK before writing.
```

**If the user has session notes:** When the finding comes from `/session-notes` structured format (containing **Vulnerability**, **URL**, **Payload**, **Reproduction**, **Impact** fields), map those fields directly into the report:
- `Vulnerability` → CWE classification + title
- `URL` + `Parameter` → Affected asset in reproduction steps
- `Payload` → PoC section
- `Reproduction` → Reproduction steps (expand with HTTP details)
- `Impact` → Impact section (strengthen with business context)
- `Severity Estimate` → Starting point for CVSS calculation

### Step 2: Classify and Score

```
1. Map to CWE identifier
2. Calculate CVSS 3.1 with justification for each metric
3. Calculate CVSS 4.0 with justification (include Attack Requirements metric)
4. Determine severity tier — include both scores; note which the platform uses
   - HackerOne: uses CVSS 3.1 with their own severity mapping
   - Bugcrowd: uses VRT (maps loosely to CVSS)
   - Intigriti: uses CVSS 3.1
   - When in doubt, lead with 3.1 and include 4.0 as supplementary
5. Cross-reference with program's reward table (if known from program-research)
6. Assess duplicate risk (if program-research data available)
```

### Step 3: Write the Report

```
1. Title — [Vuln Type] in [Location] allows [Impact]
   - Specific and scannable, not generic

2. Summary — 2-3 sentences for a triager reading report #47 today

3. Severity — CVSS 3.1 + 4.0 scores with metric-by-metric justification
   - Lead with whichever version the platform uses
   - Include both for completeness

4. Repro steps — Numbered, exact, zero ambiguity
   - Include raw HTTP requests where applicable
   - Include exact payloads
   - Note prerequisites (accounts, state, timing)

5. Impact — Concrete business consequences
   - Not "could be bad" but "attacker reads all user emails"
   - Affected user count/scope
   - Realistic attack scenario

6. PoC — Working proof
   - curl command, script, or step-by-step with screenshots

7. Remediation — Show you understand the fix
   - Specific technical recommendation
   - Code example if applicable
```

### Step 4: Quality Check

```
Before delivering, verify:
- [ ] Title states impact, not just vuln type
- [ ] Summary readable in 10 seconds
- [ ] Repro steps work if followed blindly
- [ ] Impact is concrete, not hypothetical
- [ ] CVSS justified honestly (not inflated)
- [ ] PoC actually proves the vulnerability
- [ ] No out-of-scope items included
- [ ] Formatted for target platform
```

---

## Platform Formatting

### HackerOne
- Markdown formatting (renders well)
- Reference their severity taxonomy
- Tag the correct asset from scope
- Include structured impact section

### Bugcrowd
- Follow VRT (Vulnerability Rating Taxonomy)
- Use P1-P5 priority alongside CVSS
- Reference specific scope target

### Intigriti
- Follow their severity guidelines
- Reference program-specific rules
- Include domain from scope

### Self-Hosted / Email
- Clean markdown or plain text
- Include all context (no platform assumptions)
- Attach PoC files separately

---

## Tips

1. **Lead with impact** — the title and first sentence should make severity obvious
2. **Write for a tired triager** — they read dozens of reports daily
3. **Be honest about severity** — inflated CVSS gets recalculated and hurts reputation
4. **One bug per report** — unless reporting a chain (use `/chain` to document multi-step attacks)
5. **Include a working PoC** — a curl command beats a paragraph of explanation
6. **Don't rush criticals** — take 30 minutes to write a great report rather than submitting a messy one
