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

### Step 5b: Assess AI & Automation Landscape

Check for AI-specific program features:
1. Search for "AI" or "LLM" in program scope and policy
2. Check if program has specific AI vulnerability categories
3. Search for disclosed AI/prompt injection reports on the program
4. Assess XBOW/automation duplicate risk — look for high volume of recent reports
5. Note if program uses HackerOne's AI Bug Bounty framework
6. Check for different bounty tables for AI vs traditional vulns

Context: As of 2025, 1,121 HackerOne programs include AI in scope (270% YoY increase). Programs paying for AI vulns saw 339% increase in bounties YoY. Prompt injection reports surged 540%.

### Step 6: Synthesize

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

## Market Context (2025-2026)

Key data points for program evaluation:

| Metric | Value | Source |
|--------|-------|--------|
| HackerOne total annual payouts | $81M (13% YoY increase) | HackerOne 9th Annual Report (Oct 2025) |
| Top 10 programs paid | $21.6M combined | HackerOne 2025 |
| Top 100 programs paid | $51M (Jul 2024 - Jun 2025) | HackerOne 2025 |
| Top 100 all-time earners | $31.8M total | HackerOne 2025 |
| HackerOne validated vulns | 580,000+ across 1,950 enterprise programs | HackerOne 2025 |
| HackerOne all-time payouts | $300M+ (30 hackers earned $1M+) | HackerOne 2023 milestone |
| AI vulnerability reports | 210% increase YoY | HackerOne 2025 |
| Prompt injection reports | 540% increase YoY | HackerOne 2025 |
| Programs with AI in scope | 1,121 (270% YoY increase) | HackerOne 2025 |
| Autonomous AI valid reports | 560+ submitted on HackerOne | HackerOne 2025 |
| Avg pentest findings | 12 vulns, 16% high/critical | HackerOne 2025 |
| Bug bounty high/critical rate | 25% of submissions | HackerOne 2025 |
| Top vulnerability type (bounty) | XSS (but declining) | HackerOne 2025 |
| Fastest growing vuln class | Authorization flaws (IDOR, BOLA) | HackerOne 2025 |
| Declining vuln classes | XSS, SQLi | HackerOne 2025 |
| Hacker AI adoption | 82% (up from 64% in 2023) | Bugcrowd 2026 Report |
| Hacker team collaboration | 72% say teams yield better results | Bugcrowd 2026 |
| Hardware vulns | 88% increase | Bugcrowd CISO Report 2025 |
| Network vulns | 2x spike | Bugcrowd CISO Report 2025 |
| Microsoft annual bounty payouts | $17M to 344 researchers | Microsoft 2025 |
| Smart contract damages (H1 2025) | $263M across Web3 | CoinLaw 2026 |

Notable new programs (2025-2026):
- Samsung: up to $1M for critical mobile vulns (Knox Vault, TEEGRIS OS bypasses)
- Crypto.com: up to $2M for critical security vulns
- Microsoft Zero Day Quest: $1.6M+ paid in inaugural event; spring 2026 event announced (Azure, Copilot, Identity, M365). Microsoft paid $17M total to 344 researchers in 2025
- Nvidia + Intigriti: Bug bounty covering core AI assets + public VDP for all Nvidia properties
- OpenAI on Bugcrowd: increased critical payouts from $20K to $100K (ChatGPT, APIs in scope)
- Anthropic on HackerOne: up to $15K for universal jailbreaks on Constitutional Classifiers; 405 participants, 3,000+ hours of red teaming
- Czech Republic government: Public sector bug bounty via Hackrate
- huntr: world's first bug bounty platform specifically for AI/ML vulnerabilities
- Immunefi: undisputed leader in Web3/blockchain bounties, with some of the largest single payouts in the industry

Hacker demographics (Bugcrowd Inside the Mind of a Hacker 2026, 2,000+ surveyed):
- 92% of hackers are 34 or younger; 69% hold a college degree or higher
- 20% identify as neurodivergent
- 74% are motivated by financial gain, but 85% say reporting a critical vulnerability matters more than money
- 65% have chosen NOT to disclose a vulnerability because the company lacked a clear, safe reporting pathway
- Continuous assurance testing is replacing annual pentests as the industry standard

Competition awareness:
- XBOW reached #1 on HackerOne US leaderboard with ~1,060 submissions, running 80x faster than manual teams; raised $75M in funding
- 82% of researchers now use AI tools in hunting (Bugcrowd 2026)
- Programs with fewer researchers (avg 56 vs 97) tend to have higher-impact submissions
- 75% of hackers say hacking is becoming more about money than curiosity (Bugcrowd 2026)
- Bugcrowd 2026 prediction: high-end vulnerability research will become more valuable as AI handles low-hanging fruit
- HexStrike AI MCP server enables AI agents to autonomously run 150+ security tools — increasing automation of recon and scanning
- Web3 losses exceeded $3 billion in H1 2025 alone; Immunefi dominates blockchain bug bounties
- FailSafe 2025: access control flaws caused $953.2M in losses; logic flaws another $63M
- As XSS and SQLi become easier to mitigate, organizations shift rewards toward identity, access, and business logic flaws

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
