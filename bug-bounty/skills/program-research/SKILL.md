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

Context: As of 2025, 1,121 HackerOne programs include AI in scope (270% YoY increase). Programs paying for AI vulns saw 339% increase in bounties YoY. Prompt injection reports surged 540%. HackerOne Hai Triage adopted by 90% of customers. Bugcrowd AI Triage Assistant (Dec 2025) achieves 98% P1 accuracy. HackerOne clarified (Feb 2026) that researcher submissions are NOT used to train AI models. OWASP Top 10 for Agentic Applications released December 2025.

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
| HackerOne all-time payouts | $300M+ (30 hackers earned $1M+); Top 100 all-time earners $31.8M | HackerOne 2025 |
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
| Meta annual bounty payouts | $4M in 2025; $25M all-time | Meta 2025 |
| Google record Chrome bounty | $250,000 for CVE-2025-4609 (sandbox escape) | Google 2025 |
| Prompt injection in production AI | 73%+ of deployments affected; only 34.7% have defenses | Security audits 2025 |
| AI coding tool CVEs | GitHub Copilot RCE (9.6), Cursor IDE (9.8), MS Copilot (9.3) | 2025 |
| Smart contract damages (H1 2025) | $263M across Web3 | CoinLaw 2026 |
| Bug bounty market valuation | $1.19B (2024), projected $3.98B by 2032 (16.3% CAGR) | Market research 2025 |
| Critical vuln payout increase | 32% average increase for critical findings | Bugcrowd CISO Report 2025 |
| Exploitation speed | 32.1% exploited on/before CVE disclosure day | VulnCheck State of Exploitation 2026 |
| CISA KEV catalog additions | 245 vulnerabilities in 2025 (30%+ increase over 2023-2024) | CISA 2025 |
| API-related CVEs | 17% of all 2025 published security bulletins (11,053 of 67,058) | Wallarm 2025 |
| API-related CISA KEVs | 43% of newly added CISA KEVs in 2025 were API-related | Wallarm 2025 |
| AISLE AI CVE discoveries | 100+ CVEs across 30+ projects; 13/14 OpenSSL CVEs in 2025 | AISLE Jan 2026 |
| Claude Code Security vulns found | 500+ in production open-source codebases (launched Feb 20, 2026) | Anthropic 2026 |
| ZAST.AI CVE discoveries | 119 CVEs across Microsoft Azure SDK, Apache Struts, WordPress, Langfuse | ZAST.AI Feb 2026 |
| Trend Micro AESIR CVEs | 21 CVEs across NVIDIA, Tencent, MLflow, MCP tooling | Trend Micro 2025-2026 |
| Immunefi total payouts | $100M+ total; largest single payout $10M (Wormhole) | Immunefi 2025 |
| Web3 losses H1 2025 | $3B+; $1.83B from access control exploits alone | Immunefi 2025 |
| HackerOne Hai Triage adoption | 90% of HackerOne customers by end of 2025 | HackerOne 2025 |
| Bugcrowd AI Triage Assistant | 98% P1 accuracy, 98% duplicate detection confidence | Bugcrowd Dec 2025 |
| MCP servers exposed | 8,000+ publicly exposed (Feb 2026), 492 vulnerable | Security research 2026 |
| Enterprise CART adoption | 60% of large enterprises using continuous automated red teaming by 2026 | Industry 2026 |
| EU AI Act compliance deadline | August 2, 2026; penalties up to 35M EUR or 7% of global turnover | EU 2026 |
| OpenAI o3 zero-day | CVE-2025-37899 Linux kernel SMB use-after-free found with o3 assistance | Security research 2025 |

Notable new programs and expansions (2025-2026):
- **Apple Security Bounty Evolved** (November 2025): max reward doubled to **$2M** for zero-click remote exploits (from $1M), with bonuses potentially exceeding $5M. New categories: WebKit sandbox escapes (up to $300K), wireless proximity exploits over any radio (up to $1M). "Target Flags" system for accelerated payouts processed before a fix is available. $35M total paid to date
- Samsung: up to $1M for critical mobile vulns (Knox Vault, TEEGRIS OS bypasses)
- Crypto.com: up to $2M for critical security vulns
- **Microsoft Zero Day Quest expanded**: up to **$5M total bounty pool** for 2026 live hacking event at Redmond campus (Feb-Mar 2026), targeting Azure, Copilot, M365, Identity. 50% bounty multiplier for critical AI/cloud vulnerabilities. Paid $1.6M in inaugural 2025 event. Microsoft "In Scope by Default" — all products/services eligible even without formal bounty program. $17M total to 344 researchers in 2025
- **Google AI Vulnerability Reward Program** (October 2025): dedicated AI VRP covering Google Search, Gemini Apps, and Workspace — up to $30K per finding. Chrome vulnerability rewards increased to $250,000 for highest category
- Nvidia + Intigriti: Bug bounty covering core AI assets + public VDP for all Nvidia properties
- OpenAI on Bugcrowd: increased critical payouts from $20K to **$100K** for critical infrastructure flaws. Specialized Bio Bug Bounty: $25K for universal jailbreaks of bio/chem safety filters, $10K for multiple jailbreak prompts
- Anthropic on HackerOne: up to $15K for universal jailbreaks on Constitutional Classifiers; 405 participants, 3,000+ hours of red teaming
- **Jenkins** (December 2025): launched new bug bounty program on YesWeHack, backed by the European Commission
- Czech Republic government: Public sector bug bounty via Hackrate
- huntr: world's first bug bounty platform specifically for AI/ML vulnerabilities
- **Immunefi**: surpassed **$100M in total payouts**. Smart contract bugs represent 77.5% ($78M) of total payouts. Largest single payout: $10M (Wormhole critical bug). Undisputed leader in Web3/blockchain bounties
- **curl bug bounty shutdown** (January 2026): Daniel Stenberg ended program after AI-generated submissions overwhelmed team — only 5% of 2025 submissions were genuine vulnerabilities. First major program shutdown attributed to AI slop. Paid $90K+ for 81 genuine vulnerabilities before closing. 20 submissions in just the first weeks of 2026 (7 in a single 16-hour period), none identifying real vulnerabilities. Reports via GitHub directly now, no monetary rewards
- **HackerOne Good Faith AI Research Safe Harbor** (January 20, 2026): clarifies legal protections for researchers conducting authorized AI security testing
- **HackerOne AI Policy Update** (February 2026): researcher submissions are NOT used to train AI models — addresses researcher concerns about data usage

Notable disclosed vulnerabilities (2025-2026):
- CVE-2025-4609: Google Chrome sandbox escape — record-breaking $250,000 bounty payout
- CVE-2025-55315: ASP.NET Core Kestrel HTTP request smuggling — CVSS 9.9, highest ever for ASP.NET Core ($10K bounty)
- CVE-2025-6965: Critical SQLite memory corruption (CVSS 7.2) — first AI agent (Google Big Sleep) to foil active exploitation in the wild
- CVE-2025-53773: GitHub Copilot RCE via prompt injection (CVSS 9.6) — potentially compromising millions of developer machines
- CVE-2025-6514: Critical OS command injection in mcp-remote (437K+ downloads) — malicious MCP servers achieve RCE via crafted auth endpoints
- CVE-2026-27825: Critical unauthenticated RCE and SSRF in mcp-atlassian (MCP server vulnerability)
- CVE-2025-55182: Critical RCE in React Server Components via insecure deserialization in Flight protocol (disclosed via Meta Bug Bounty)
- CVE-2025-59536 / CVE-2026-21852: RCE and API token exfiltration through Claude Code project files (Check Point Research)
- CVE-2025-68145/68143/68144: Three CVEs in Anthropic's Git MCP server enabling RCE via prompt injection
- CVE-2025-67511: Critical command injection in CAI cybersecurity framework (CVSS 9.6-9.7) — AI security tools themselves vulnerable
- CVE-2025-35027: Command injection in Unitree robotic products allowing root access through Bluetooth Low Energy
- CVE-2025-9074: Docker Desktop access control failure — locally running Linux containers access Docker Engine API without authentication
- GitHub MCP server prompt injection breach: public GitHub issues used to hijack AI assistants and exfiltrate private repos
- Supabase Cursor agent breach: privileged agent processed support tickets as commands, attackers exfiltrated integration tokens via SQL injection
- Anthropic Filesystem-MCP: sandbox escape + symlink bypass enabling arbitrary file access and code execution
- Microsoft MarkItDown MCP server: unpatched SSRF that can compromise AWS EC2 instances via metadata service exploitation
- OpenSSL: 12 zero-day vulnerabilities discovered by AISLE AI system, including CVE-2025-15467 (stack buffer overflow, remotely exploitable); three bugs dated back to 1998-2000
- SSRF protection bypass in AutoGPT reported through huntr platform
- Multiple Salesforce Tableau Server vulns including unrestricted file upload → RCE (CVE-2025-52449)
- WhatsApp MCP tool poisoning: Invariant Labs demonstrated malicious MCP server silently exfiltrating user's entire WhatsApp history via tool poisoning combined with legitimate whatsapp-mcp server
- CVE-2025-37899: Linux kernel SMB use-after-free zero-day found with OpenAI o3 assistance by Sean Heelan

Hacker demographics (Bugcrowd Inside the Mind of a Hacker 2026, 2,000+ surveyed):
- 92% of hackers are 34 or younger; 69% hold a college degree or higher
- 20% identify as neurodivergent
- 74% are motivated by financial gain, but 85% say reporting a critical vulnerability matters more than money
- 65% have chosen NOT to disclose a vulnerability because the company lacked a clear, safe reporting pathway
- Continuous assurance testing is replacing annual pentests as the industry standard

Competition awareness:
- XBOW reached #1 on both US and global HackerOne leaderboards with 1,400+ zero-day submissions, running 80x faster than manual teams; raised $75M in funding; expanding APAC presence in 2026
- HackerOne **split leaderboards** to separate individuals from companies/agents like XBOW
- XBOW launched **Pentest On-Demand** — fully automated pentest service delivering results within 5 business days
- 82% of researchers now use AI tools in hunting (Bugcrowd 2026)
- **Shannon** emerged as GitHub's fastest-rising security project (96.15% on XBOW benchmark, fully autonomous white-box pentesting)
- **Strix** emerged as leading open-source autonomous AI pentest tool (~2K GitHub stars, used by Fortune 500 security teams and top 1% HackerOne hunters)
- **NeuroSploit v3** introduced autonomous pentesting with exploit chaining and anti-hallucination safeguards
- Programs with fewer researchers (avg 56 vs 97) tend to have higher-impact submissions
- 75% of hackers say hacking is becoming more about money than curiosity (Bugcrowd 2026)
- Bugcrowd 2026 prediction: high-end vulnerability research will become more valuable as AI handles low-hanging fruit
- HexStrike AI MCP server enables AI agents to autonomously run 150+ security tools — increasing automation of recon and scanning
- **AISLE** found all 12 OpenSSL zero-days (Jan 2026), 100+ CVEs across 30+ projects — sets new bar for AI-discovered bugs
- **Codex Security** (OpenAI, formerly Aardvark) — rebranded; available to ChatGPT Enterprise/Business/Edu via Codex web; continuous source code analysis, 92% recall, 10 CVEs assigned
- **Claude Code Security** launched Feb 20, 2026 — found 500+ vulnerabilities in production open-source codebases; bugs undetected for decades
- **ZAST.AI** raised $10M total, 119 CVEs assigned, "zero false positive" approach — new competitor for automated code scanning
- **Trend Micro AESIR** — AI-powered zero-day discovery (21 CVEs across NVIDIA, Tencent, MLflow, MCP tooling)
- **Terra Security** raised **$38M total** (Series A $30M led by Felicis, Sep 2025) — first agentic-AI continuous pentesting platform, Fortune 100 clients
- **AWS Security Agent** (preview re:Invent Dec 2025) — multi-agent continuous on-demand pentesting
- **Hadrian** — attack-surface-driven 24/7 autonomous pentesting; real-time testing triggered by attack surface changes
- **Escape** (Y Combinator) — API-native DAST with agentic crawling; 140+ security tests; now discovers unauthenticated MCP endpoints; 2,000+ security teams
- **PentAGI** (Feb 2026) — Go-based multi-agent pentesting with PostgreSQL + Neo4j; 20+ tools in isolated Docker environments
- **Assail Ares** launched from stealth (Jan 2026) — autonomous API/web/mobile pentesting
- **BlacksmithAI** and **Zen-AI-Pentest** emerged as new open-source AI pentesting frameworks (Feb-Mar 2026)
- **MCP security is a major attack surface** — 8,000+ servers exposed (Feb 2026), 492 vulnerable; CVE-2025-6514 (CVSS 10.0, mcp-remote RCE); Adversa AI published MCP Security TOP 25; CVE-2026-27825 (mcp-atlassian), CVE-2025-59536/CVE-2026-21852 (Claude Code RCE), CVE-2025-68145/68143/68144 (Git MCP server RCE); VulnerableMCP.info tracks the growing CVE database
- Web3 losses exceeded **$3 billion in H1 2025** alone; $1.83B from access control exploits; Immunefi surpassed $100M in total payouts
- FailSafe 2025: access control flaws caused $953.2M in losses; logic flaws another $63M
- **MITRE CWE Top 25 (2025):** CWE-79 (XSS) still #1 but CWE-862 (Missing Authorization) climbed 5 positions — biggest mover. New entries include CWE-770 (Allocation of Resources Without Limits), CWE-639 (Authorization Bypass Through User-Controlled Key)
- As XSS and SQLi become easier to mitigate, organizations shift rewards toward identity, access, and business logic flaws
- **curl bug bounty shutdown** (Jan 2026) demonstrates AI slop risk — first major program closed due to AI-generated garbage flooding; 8x normal submission volume, only 5% genuine
- **By 2027**, "manual pentesting" predicted to become a boutique service; 99% of vulnerability assessments will be agentic (industry prediction)
- **EU AI Act compliance deadline August 2, 2026** — driving mandatory AI red teaming and creating new consulting/testing demand

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
