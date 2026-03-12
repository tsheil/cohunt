---
name: program-research
description: Researches bug bounty programs before hunting — scope, rewards, disclosed reports, duplicate patterns, and competition assessment. Provides a go/no-go recommendation with rationale. ALWAYS use this skill when evaluating whether a program is worth hunting on, or when the user asks about any bug bounty platform or program. Trigger on "which program", "what pays", "program scope", "bug bounty program", "HackerOne program", "Bugcrowd program", "Intigriti", "Synack", "is it worth hunting on", "duplicate risk", "program rewards", "bounty table", "how much does X pay", "payout range", "response time", "triage time", "who else is hunting", "should I hunt this", "compare programs", or when evaluating whether to hunt on a program. Also use when comparing programs or checking what gets marked as duplicate. For market statistics and trends, see reference/market-context.md.
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
| **Access** | [Public / Private / Invite-only / Application-only] |
| **Requirements** | [Test accounts? Paid product? VPN? KYC/ID? NDA? Region limits?] |

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

### Hard Vetoes (Any = NO-GO)
- [ ] Invite-only with no application path
- [ ] No safe harbor and no VDP
- [ ] No matching asset class for your skills
- [ ] High-friction access (KYC, paid product, NDA) without proportional rewards
- [ ] Trust history: pattern of N/A on valid bugs, non-payment, or slow resolution (>90 days)

### Go / No-Go Scorecard

| Factor | Score (1-5) | Weight | Notes |
|--------|------------|--------|-------|
| Reward potential | _ | 3x | Critical payout ÷ your time estimate |
| Asset match | _ | 3x | Your skills vs. scope assets |
| Competition density | _ | 2x | Active researchers, duplicate risk |
| AI competition risk | _ | 2x | XBOW-dominated vuln classes in scope? |
| Response quality | _ | 2x | Time to triage, payment consistency |
| Scope freshness | _ | 1x | New assets, recent feature launches |

**Score ≥50 = GO, 35-49 = CAUTION (timebox to 4hrs), <35 = NO-GO**

### Recommended Approach
- **Focus areas:** [Where to look based on disclosed reports and scope]
- **Vuln types to target:** [Highest-ROI classes — see Competitive Calibration]
- **Time investment:** [Stop-loss timebox: 4hrs for CAUTION, 8hrs for GO before reassessing]
- **Competition level:** [High / Medium / Low — based on program age, payouts, researcher count]

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

### Step 5: Synthesize

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
- **HackerOne payouts**: $81M annual (+13% YoY), $300M+ all-time; top 10 programs = $21.6M
- **AI programs**: 1,121 with AI in scope (270% YoY increase), 560+ valid AI agent reports, 339% jump in AI bounties
- **Top bounties**: Apple $2M base (up to $5M+ with bonuses), Microsoft $5M pool (Zero Day Quest), Google $250K Chrome
- **Competition**: XBOW #1 on HackerOne (1,060 submissions, 41% acceptance rate, 28-min turnaround); 82% of hunters use AI tools
- **Best ROI vuln classes**: IDOR/BOLA (+29% YoY), business logic (45% of awards), auth bypass (HackerOne #2) — low AI competition, highest payouts
- **Avoid**: commodity XSS/SQLi (XBOW-dominated, declining payouts)
- **Fastest-growing attack surface**: MCP (50+ CVEs by March 2026), AI coding tools (IDEsaster: 30+ vulns), React RSC
- **New programs**: Cloudflare public launch, Nvidia on Intigriti, IBM Granite ($100K), NEAR Intents bridge, Ethereum $1M max

---

## Competitive Calibration (XBOW Era)

XBOW reached #1 on HackerOne with 1,060 submissions. Understanding its strengths and weaknesses informs where to hunt.

### Where AI Dominates (Avoid Head-to-Head)

| Vuln Class | XBOW Capability | Human Strategy |
|-----------|----------------|---------------|
| SQLi | 28-min turnaround, automated payload generation | Don't hunt basic SQLi; focus on second-order or blind SQLi in complex workflows |
| XSS (reflected) | Mass-scanned at scale, 0% FP with headless browser validation | Only hunt stored XSS → account takeover chains |
| RCE (direct) | 48-step exploit chains, Python script generation | Focus on RCE via business logic (e.g., template injection, deserialization in custom code) |

### Where Humans Win (Target These)

| Vuln Class | Why AI Struggles | Payout Trend |
|-----------|-----------------|-------------|
| **IDOR/BOLA** | Requires understanding business objects, tenant boundaries, role hierarchies | ↑↑ +29% YoY, $4M+ IAC awards |
| **Business Logic** | State machines, payment flows, subscription bypass, race conditions | 45% of all bounty awards (Intigriti) |
| **Auth Bypass** | SSO chains, MFA bypass, session fixation across domains | HackerOne #2 vuln type |
| **Multi-tenant isolation** | Cross-tenant data access requires understanding data models | Highest per-report payouts in SaaS programs |
| **AI/LLM vulns** | Prompt injection impact assessment, agent abuse chains | ↑↑↑ 270% YoY programs in scope |

### XBOW Outcome Data (Calibration)

XBOW's 1,060 HackerOne submissions broke down as:
- **12% resolved** (130) — programs confirmed and fixed
- **29% triaged** (303) — mostly VDP, acknowledged but not bounty-eligible
- **20% duplicate** (208) — even AI with SimHash dedup hits duplicates at scale
- **20% informative** (209) — finding was real but impact insufficient for bounty
- **3% N/A** (36) — rejected outright
- **15% pending** (158) — still in review

**Takeaways:** (1) Impact articulation matters — 20% informative means the bug was real but the report didn't demonstrate sufficient business impact. Always chain findings to show real-world harm. (2) Duplicate risk is real even for AI — target programs with fewer researchers (avg 56 vs 97 for higher-impact submissions). (3) VDP programs accept but don't pay — focus on paid programs unless building reputation.

### Go/No-Go AI Competition Check

Add to Step 5 (Synthesize):
1. **Is this program in XBOW's target list?** Programs with high-volume, low-complexity scope (public web apps with standard auth) are XBOW territory
2. **Does the program have AI/LLM features?** If yes, human advantage — XBOW doesn't hunt prompt injection or agent abuse
3. **Does the program reward business logic?** If top disclosed reports are IDOR/auth/logic bugs, human hunters have the edge
4. **How many researchers?** Programs with <60 active researchers have higher per-report impact
5. **What's the duplicate rate?** Check disclosed reports — if 50%+ of recent reports are XSS/SQLi, AI competition is saturating those classes

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

**Platform AI Submission Rules (2026):**
- **HackerOne:** all automated/AI findings require human expert review before submission; fully autonomous submission prohibited; hackbot policy requires operating within published VDP + Code of Conduct; AI Research Safe Harbor opt-in available
- **Bugcrowd:** AI tools permitted as research aids; no prohibition on AI-assisted submissions; policy prohibits third-party AI training on researcher data
- **Intigriti:** CVSS V4 for all new submissions; enhanced bonuses for quality/variant research; hourly payout model available on some programs

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
| [reference/market-context.md](reference/market-context.md) | Market metrics, vuln class ROI matrix, AI automation data, MCP attack surface stats, platform statistics, notable programs, disclosed vulnerabilities, competition landscape, autonomous pentesting tools, industry trends (150+ data points) | ~367 |

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
