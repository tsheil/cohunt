---
name: program-research
description: Researches bug bounty programs before hunting — scope, rewards, disclosed reports, duplicate patterns, and competition assessment. Provides a go/no-go recommendation with rationale. Trigger on "which program", "what pays", "program scope", "bug bounty program", "HackerOne program", "Bugcrowd program", "is it worth hunting on", "duplicate risk", "program rewards", "bounty table", or when evaluating whether to hunt on a program. Also use when comparing programs or checking what gets marked as duplicate. For market statistics and trends, see reference/market-context.md.
---

# Program Research

Get complete intelligence on any bug bounty program before you start hunting using web search.

---

## Output Format

```markdown
# Program Research: [Company/Program Name]

**Generated:** [Date]
**Sources:** Web Search

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

### Step 4: Assess AI & Automation Landscape

Check for AI-specific program features:
1. Search for "AI" or "LLM" in program scope and policy
2. Check if program has specific AI vulnerability categories
3. Search for disclosed AI/prompt injection reports on the program
4. Assess XBOW/automation duplicate risk — look for high volume of recent reports
5. Note if program uses HackerOne's AI Bug Bounty framework
6. Check for different bounty tables for AI vs traditional vulns

For full AI market context and statistics, see [reference/market-context.md](reference/market-context.md).

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

## Market Context

For full market statistics, reward benchmarks, notable programs, competition landscape, disclosed vulnerability examples, and industry trends, see **[reference/market-context.md](reference/market-context.md)**.

Key highlights for quick program evaluation:
- **Market size**: $2.06B (2026), projected $7.74B by 2035
- **HackerOne payouts**: $81M annual, $300M+ all-time
- **AI programs**: 1,121 with AI in scope (270% YoY increase), prompt injection reports up 540%
- **Top bounties**: Apple $2M base (up to $5M+ with bonuses), Microsoft $5M pool (Zero Day Quest), Google $250K Chrome
- **Competition**: XBOW declared mission completed (March 2026, pivoting to pre-production scanning); 82% of hunters use AI tools
- **New programs**: Nvidia on Intigriti (AI assets), IBM Granite ($100K), Amazon Nova; HackerOne IBB controversy signals trust risks in intermediary programs
- **Fastest-growing attack surface**: MCP (30+ CVEs in 60 days), AI coding tools (IDEsaster: 30+ vulns)
- **Shifting vuln rewards**: authorization flaws rising, XSS/SQLi declining

---

## Duplicate & N/A Avoidance

Before submitting, check:

**High-Duplicate Patterns** (likely already reported):
- Simple XSS on main domain (especially reflected XSS in search)
- Missing rate limiting on login (unless account takeover chain)
- Missing security headers (HSTS, X-Frame-Options) without exploit chain
- Self-XSS without social engineering chain
- Open redirect without OAuth/auth chain
- CSRF on non-sensitive actions (profile update, preferences)

**What Gets N/A or Informational:**
- Theoretical vulnerabilities without PoC
- Scanner output without manual verification
- Findings on out-of-scope assets (always check scope first)
- "Best practice" recommendations (no security headers, missing CSP)
- Bugs requiring victim to have malware/rooted device
- Social engineering without technical component

**Differentiating from Duplicates:**
- Show a different root cause (not just a different parameter)
- Demonstrate higher impact (P4 IDOR to P2 PII leak chain)
- Find the same bug class in a new scope area (different subdomain, API version)
- Provide variant analysis: if you find bug X, search for the same pattern in related endpoints, other API versions, mobile APIs, and internal tools

---

## Tips for Better Research

1. **Name the platform** — "research Shopify on HackerOne" is more targeted
2. **State your skill level** — helps calibrate the go/no-go assessment
3. **Mention your focus** — "I specialize in API bugs" shapes the recommendation
4. **Chain with recon** — Research the program first, then recon their in-scope targets

---

## Reference Files

This skill uses progressive disclosure. Detailed reference material is available on demand:

| File | Contents | Lines |
|------|----------|-------|
| [reference/market-context.md](reference/market-context.md) | Market metrics, AI automation data, MCP attack surface stats, platform statistics, notable programs, disclosed vulnerabilities, competition landscape, autonomous pentesting tools, industry trends (140+ data points) | ~459 |

**Quick search** — find specific market data without loading the full file:
```
grep -n "IDOR\|BOLA\|payout\|reward\|bounty" ${CLAUDE_SKILL_DIR}/reference/market-context.md
grep -n "XBOW\|Shannon\|Codex Security\|autonomous" ${CLAUDE_SKILL_DIR}/reference/market-context.md
grep -n "HackerOne\|Bugcrowd\|Intigriti" ${CLAUDE_SKILL_DIR}/reference/market-context.md
grep -n "MCP\|agentic\|AI vuln\|prompt injection" ${CLAUDE_SKILL_DIR}/reference/market-context.md
grep -n "Apple\|Google\|Microsoft\|Samsung" ${CLAUDE_SKILL_DIR}/reference/market-context.md
```

---

## Related Skills

- **target-recon** — After researching the program, recon the actual targets
- **hunt-plan** command — Combine program research + recon into a prioritized action plan
- **vuln-patterns** — Get testing patterns for vulnerability classes likely to pay on this program
- **ai-hunting** — If program has AI/LLM features in scope
