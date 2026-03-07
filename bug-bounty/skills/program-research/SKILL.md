---
name: program-research
description: Research a bug bounty program — scope, rewards, response times, disclosed reports, and what gets paid. Works standalone with web search, supercharged with bug bounty platform connectors. Trigger with "research the X bug bounty program", "what does X pay for bugs", "what's in scope for", "program intel on", "should I hunt on X".
---

# Program Research

Get complete intelligence on any bug bounty program before you start hunting. This skill always works with web search, and gets significantly better with bug bounty platform connectors.

## How It Works

```
┌─────────────────────────────────────────────────────────────────┐
│                    PROGRAM RESEARCH                               │
├─────────────────────────────────────────────────────────────────┤
│  ALWAYS (works standalone via web search)                        │
│  ✓ Program profile: platform, scope, policy, safe harbor        │
│  ✓ Reward structure: bounty table, bonus programs, swag         │
│  ✓ Scope analysis: in-scope domains/apps, out-of-scope items    │
│  ✓ Disclosed reports: top paid vulns, common CWEs, N/A patterns │
│  ✓ Response metrics: time to first response, triage, bounty     │
│  ✓ Hunt readiness: go/no-go recommendation with rationale       │
├─────────────────────────────────────────────────────────────────┤
│  SUPERCHARGED (when you connect your tools)                      │
│  + Platform API: live scope, real-time stats, report history     │
│  + Vulnerability DB: CVE mapping for in-scope technologies       │
│  + Asset discovery: pre-mapped attack surface for scope items    │
└─────────────────────────────────────────────────────────────────┘
```

---

## Getting Started

Just tell me what program to research:

- "Research the Shopify bug bounty program"
- "What does GitHub pay for bugs?"
- "What's in scope for the Tesla program?"
- "Program intel on HackerOne's own program"
- "Should I hunt on Uber's bug bounty?"

I'll search the web for program details, disclosed reports, and hunter experiences. If you have platform connectors, I'll pull live data too.

---

## Connectors (Optional)

Connect your tools to supercharge this skill:

| Connector | What It Adds |
|-----------|--------------|
| **Bug bounty platform** | Live scope, real-time reward tables, report statistics, response metrics |
| **Vulnerability database** | CVE data mapped to program's technology stack |
| **Asset discovery** | Pre-enumerated subdomains and services for in-scope targets |

> **No connectors?** No problem. Web search provides solid program intelligence from public disclosures, write-ups, and platform pages.

---

## Output Format

```markdown
# Program Research: [Company/Program Name]

**Generated:** [Date]
**Sources:** Web Search [+ Platform API] [+ Vuln DB]

---

## Quick Take

[2-3 sentences: Is this worth hunting? Key highlights and concerns.]

---

## Program Profile

| Field | Value |
|-------|-------|
| **Program** | [Name] |
| **Platform** | [HackerOne / Bugcrowd / Intigriti / Self-hosted] |
| **Type** | [Public / Private / VDP] |
| **Launched** | [Date or approximate] |
| **Safe Harbor** | [Yes / Partial / No] |
| **Disclosure** | [Allowed / Case-by-case / Not allowed] |
| **Managed** | [Yes (triaged by platform) / No (triaged by company)] |

---

## Scope

### In Scope

| Asset | Type | Tier | Notes |
|-------|------|------|-------|
| [*.example.com] | [Wildcard domain] | [Critical] | [Main web app] |
| [api.example.com] | [API] | [Critical] | [REST API] |
| [mobile app] | [iOS/Android] | [Standard] | [App store link] |

### Out of Scope

- [asset or category]
- [asset or category]
- [specific exclusions — e.g., DoS, social engineering, phishing]

### Scope Notes
[Any important nuances — recently added/removed assets, specific testing restrictions, required accounts]

---

## Rewards

| Severity | Bounty Range | Notes |
|----------|-------------|-------|
| **Critical** | $[min] – $[max] | [Examples: RCE, auth bypass] |
| **High** | $[min] – $[max] | [Examples: SQLi, stored XSS] |
| **Medium** | $[min] – $[max] | [Examples: IDOR, CSRF] |
| **Low** | $[min] – $[max] | [Examples: info disclosure] |

**Bonus Programs:** [Any bonus multipliers, special events, or seasonal programs]
**Average Payout:** [If known from public data]

---

## Vulnerability Intelligence

### Top Paid Vulnerability Types
1. [Vuln type] — [typical bounty] — [frequency]
2. [Vuln type] — [typical bounty] — [frequency]
3. [Vuln type] — [typical bounty] — [frequency]

### Common CWEs in Disclosed Reports
| CWE | Description | Count | Avg Bounty |
|-----|-------------|-------|------------|
| [CWE-XXX] | [Name] | [count] | [$amount] |

### What Gets N/A or Informational
- [Common N/A patterns — helps avoid wasted time]
- [Known duplicates — heavily tested areas]
- [Out-of-scope vuln types they explicitly reject]

---

## Disclosed Reports

[Overview of disclosed reports accessible through program, including counts by category, notable high-impact findings, and any public write-ups]

---

## AI & Automation Landscape

| Field | Value |
|-------|-------|
| **AI in Scope** | [Yes/No — specific AI features?] |
| **Prompt Injection Reports** | [Any disclosed? Accepted/rejected?] |
| **AI-Specific Bounty Table** | [Different payouts for AI vulns?] |
| **XBOW/Bot Activity** | [Signs of automated hunting? Duplicate risk from AI tools?] |
| **OWASP LLM Scope** | [Does program reference OWASP LLM Top 10?] |

---

## Response Metrics

| Metric | Value |
|--------|-------|
| **Time to First Response** | [hours/days] |
| **Time to Triage** | [hours/days] |
| **Time to Bounty** | [days] |
| **Time to Resolution** | [days] |
| **Response Efficiency** | [percentage if known] |

**Reputation:** [Hunter community sentiment — responsive, slow, fair, etc.]

---

## Hunt Readiness Assessment

### Go / No-Go: [GO / CAUTION / NO-GO]

**Rationale:**
- [Key factor 1]
- [Key factor 2]
- [Key factor 3]

### Strengths
- [Why this program is attractive]

### Concerns
- [Potential issues — competition, low payouts, slow triage]

### Recommended Approach
- **Focus areas:** [Where to look based on disclosed reports and scope]
- **Vuln types to target:** [Most likely to pay based on history]
- **Time investment:** [Estimated effort vs. expected return]
- **Competition level:** [High / Medium / Low — based on program age, payouts, hunter activity]

---

## Sources
- [Source 1](URL)
- [Source 2](URL)
```

---

## Execution Flow

### Step 1: Parse Request

```
Identify what to research:
- "Research the Shopify program" → Full program research
- "What does GitHub pay?" → Reward-focused
- "Should I hunt on Uber?" → Go/no-go assessment
- "What's in scope for Tesla?" → Scope-focused
```

### Step 2: Web Search (Always)

```
Run these searches:
1. "[Company] bug bounty program" → Program page
2. "[Company] hackerone OR bugcrowd OR intigriti" → Platform page
3. "[Company] bug bounty disclosed reports" → Public disclosures
4. "[Company] bug bounty writeup" → Hunter write-ups and experiences
5. "[Company] vulnerability disclosure policy" → VDP details
6. "[Company] bug bounty rewards" → Payout information
7. "site:hackerone.com/[company]" → HackerOne program page
```

**Extract:**
- Program platform and type
- Scope (domains, apps, APIs)
- Reward structure
- Policy details (safe harbor, disclosure)
- Disclosed report details
- Hunter community feedback

### Step 3: Disclosed Reports Analysis (Always)

```
From public disclosures:
1. Categorize by vulnerability type
2. Note bounty amounts where public
3. Identify most common CWEs
4. Track N/A and informational patterns
5. Note what gets duplicated frequently
6. Identify which scope areas get most reports
```

### Step 4: Platform Data (If Connected)

```
If bug bounty platform connected:
1. Get live program scope → Current in/out of scope
2. Get reward table → Real-time bounty ranges
3. Get program statistics → Response times, resolved count
4. Get recent activity → Trending focus areas
```

### Step 5: Vulnerability Mapping (If Connected)

```
If vulnerability database connected:
1. Map detected technologies to known CVEs
2. Check for recently disclosed vulns in scope tech
3. Identify unpatched or commonly misconfigured components
```

### Step 6: Assess AI & Automation Landscape

Check for AI-specific program features:
1. Search for "AI" or "LLM" in program scope and policy
2. Check if program has specific AI vulnerability categories
3. Search for disclosed AI/prompt injection reports on the program
4. Assess XBOW/automation duplicate risk — look for high volume of recent reports
5. Note if program uses HackerOne's AI Bug Bounty framework
6. Check for different bounty tables for AI vs traditional vulns

Context: As of 2025-2026, 1,121 HackerOne programs include AI in scope (270% YoY increase). Programs paying for AI vulns saw 339% increase in bounties YoY. Prompt injection reports surged 540%. HackerOne Hai Triage adopted by 90% of customers. Bugcrowd AI Triage Assistant (Dec 2025) achieves 98% P1 accuracy. HackerOne clarified (Feb 2026) that researcher submissions are NOT used to train AI models. OWASP Top 10 for Agentic Applications 2026 (ASI01-ASI10) released December 2025 covering agent-specific risks. OWASP AIVSS (AI Vulnerability Scoring System v0.5) extends CVSS for AI-specific severity — use for AI/agent vulnerability reports. OWASP AI Testing Guide v1 (November 2025) provides first comprehensive AI testing methodology. 79% of companies have deployed agentic AI but only 34% have AI-specific security controls (PwC 2025). EchoLeak (CVE-2025-32711) demonstrated first zero-click prompt injection in production (MS 365 Copilot). CyberStrikeAI weaponized across 55 countries (Jan-Feb 2026) — AI offensive tool proliferation is real.

### Step 7: Synthesize

```
1. Combine all sources
2. Build complete program profile
3. Analyze reward vs. effort ratio
4. Assess competition level
5. Generate go/no-go recommendation
6. Suggest focus areas and approach
```

---

## Research Variations

### Quick Overview
Focus on: Program basics, scope, reward ranges

### Deep Dive
Focus on: Everything — full disclosed report analysis, competition assessment, approach strategy

### Reward Analysis
Focus on: What pays, what doesn't, ROI estimate

### Go/No-Go Decision
Focus on: Should I invest time here? Quick assessment with recommendation

---

## Reference Files

This skill uses progressive disclosure. Core research methodology is above. For detailed market data, read the reference file:

**Market context, notable programs, disclosed vulns, competition landscape** → See [reference/market-context.md](reference/market-context.md)
Read this when: assessing program competition level, comparing payout benchmarks, evaluating AI-specific program features, checking for autonomous tool activity (XBOW, Shannon, Strix), or researching specific disclosed vulnerabilities and notable program expansions.

Key quick-reference stats (full data in reference file):
- HackerOne annual payouts: $81M; 580K+ validated vulns; 1,121 programs with AI in scope
- Fastest growing vuln class: authorization flaws (IDOR, BOLA) — up 5 positions in CWE Top 25
- XBOW reached #1 on HackerOne with 1,400+ zero-days; 75-85% of standard web vulns solved autonomously
- Bug bounty market: $2.06B (2026), projected $7.74B by 2035
- MCP: 30+ CVEs in 60 days; 38% of servers lack auth — AI's fastest-growing attack surface

---

## Tips for Better Research

1. **Name the platform** — "research Shopify on HackerOne" is more targeted
2. **State your skill level** — helps calibrate the go/no-go assessment
3. **Mention your focus** — "I specialize in API bugs" shapes the recommendation
4. **Chain with recon** — Research the program first, then recon their in-scope targets

---

## Related Skills

- **target-recon** — Recon in-scope targets after researching the program
- **hunt-plan** (command) — Combine program research with recon into an action plan
