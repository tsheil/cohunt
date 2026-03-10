# Market Context (2025-2026)

*Last updated: March 2026 (v0.75.0)*

Reference data for program evaluation. Loaded on demand when program-research needs market statistics, reward benchmarks, notable programs, competition landscape, or disclosed vulnerability examples.

## Table of Contents

- [Key Market Metrics](#key-market-metrics)
- [AI & Automation Market Data](#ai--automation-market-data)
- [MCP Security Attack Surface](#mcp-security-attack-surface)
- [Platform & Vendor Statistics](#platform--vendor-statistics)
- [Notable Programs & Expansions](#notable-programs--expansions)
- [Notable Disclosed Vulnerabilities](#notable-disclosed-vulnerabilities)
- [Hacker Demographics](#hacker-demographics)
- [Competition Landscape](#competition-landscape)
- [AI Security Tools & Startups](#ai-security-tools--startups)
- [Autonomous Pentesting Tools](#autonomous-pentesting-tools)
- [Industry Trends & Predictions](#industry-trends--predictions)

---

## Key Market Metrics

| Metric | Value | Source |
|--------|-------|--------|
| HackerOne total annual payouts | $81M (13% YoY increase) | HackerOne 9th Annual Report (Oct 2025) |
| Top 10 programs paid | $21.6M combined | HackerOne 2025 |
| Top 100 programs paid | $51M (Jul 2024 - Jun 2025) | HackerOne 2025 |
| Top 100 all-time earners | $31.8M total | HackerOne 2025 |
| HackerOne validated vulns | 580,000+ across 1,950 enterprise programs | HackerOne 2025 |
| HackerOne all-time payouts | $300M+ (30 hackers earned $1M+); Top 100 all-time earners $31.8M | HackerOne 2025 |
| Bug bounty market valuation | **$2.06B** (2026), projected **$7.74B by 2035** (CAGR 15.94%); 63% of Fortune 500 run bug bounty programs | Verified Market Research 2026 |
| Bug bounty ROI | For every $1 spent on bounties, companies save $15 ($3B in mitigated losses) | Industry analysis 2026 |
| Critical vuln payout increase | 32% average increase for critical findings | Bugcrowd CISO Report 2025 |
| Microsoft annual bounty payouts | $17M to 344 researchers | Microsoft 2025 |
| Meta annual bounty payouts | $4M in 2025; $25M all-time | Meta 2025 |
| Google record Chrome bounty | $250,000 for CVE-2025-4609 (sandbox escape) | Google 2025 |
| Immunefi total payouts | $100M+ total; largest single payout $10M (Wormhole) | Immunefi 2025 |
| Web3 losses H1 2025 | $3B+; $1.83B from access control exploits alone | Immunefi 2025 |
| Smart contract damages (H1 2025) | $263M across Web3 | CoinLaw 2026 |

---

## AI & Automation Market Data

| Metric | Value | Source |
|--------|-------|--------|
| AI vulnerability reports | 210% increase YoY | HackerOne 2025 |
| Prompt injection reports | 540% increase YoY | HackerOne 2025 |
| Programs with AI in scope | 1,121 (270% YoY increase) | HackerOne 2025 |
| Autonomous AI valid reports | 560+ submitted on HackerOne | HackerOne 2025 |
| Hacker AI adoption | 82% (up from 64% in 2023) | Bugcrowd 2026 Report |
| AI security startup funding Q4 2025 | $2.17B across 28 deals — **8x growth** over two years | Industry analysis 2026 |
| Enterprise AI readiness | Only 34% have AI-specific security controls; <40% conduct regular AI testing | Help Net Security 2026 |
| Shadow AI breach cost | $670,000 additional cost vs standard breach; 1 in 5 orgs breached via unauthorized AI | Help Net Security 2026 |
| Prompt injection in production AI | 73%+ of deployments affected; only 34.7% have defenses | Security audits 2025 |
| AI coding tool CVEs | GitHub Copilot RCE (9.6), Cursor IDE (9.8), MS Copilot (9.3); IDEsaster campaign: 30+ vulns, 24 CVEs across 10+ AI IDEs | 2025-2026 |
| Cisco vs DeepSeek R1 | 50/50 jailbreak prompts succeeded (100% bypass rate) | Cisco Q1 2025 |
| Enterprise agentic AI deployment | 79% of companies deployed; only 34% have AI-specific security controls | PwC AI Agent Survey 2025 |
| Cisco AI Security readiness gap | 83% plan agentic AI, only **29% ready** to secure; autonomous agents in OODA loops create lateral movement risk | Cisco State of AI Security 2026 |
| Agentic AI as #1 threat | 48% of cybersecurity professionals rank agentic AI as top attack vector for 2026, outranking deepfakes, ransomware, supply chain | Dark Reading poll 2026 |
| CrowdStrike AI-enabled attacks | 89% YoY surge; avg eCrime breakout time now 29 minutes; 90+ orgs compromised via prompt injection in 2025 | CrowdStrike 2026 Global Threat Report |
| Multi-turn prompt injection success | Up to 92% success rates across 8 open-weight models | Academic research 2026 |
| Prompt injection meta-analysis | Attack success rates exceed 85% vs state-of-the-art defenses; 18 defense mechanisms analyzed — most achieve <50% mitigation | arXiv:2601.17548 Jan 2026 |
| Ransomware AI involvement | ~80% of ransomware attacks now involve AI | MIT Sloan 2026 |
| Tenable AI/MCP risk | 70% of orgs integrated AI/MCP packages without centralized security oversight; 18% granted AI admin permissions rarely audited | Tenable Cloud & AI Risk Report 2026 |
| Non-human identity risk | AI agents/service accounts represent higher risk (52%) than human users (37%) in cloud environments | Tenable 2026 |
| Enterprise CART adoption | 60% of large enterprises using continuous automated red teaming by 2026 | Industry 2026 |
| Enterprise agentic AI failure costs | 64% of $1B+ companies lost >$1M from AI failures | Help Net Security 2026 |
| Gravitee AI agent security | **3+ million AI agents** in corporations; **88% reported security incidents**; **47% of agents not monitored** (~1.5M at risk); only 14.4% have full security approval | Gravitee 2026 |

---

## MCP Security Attack Surface

| Metric | Value | Source |
|--------|-------|--------|
| MCP CVEs filed | 50+ CVEs by March 2026 (30 in 60 days) — AI's fastest-growing attack surface | MCP Security Research 2026 |
| MCP servers exposed | **42,665 exposed instances** (5,194 actively vulnerable); 8,000+ publicly exposed (Feb 2026) | Security research 2026 |
| MCP servers lacking auth | 38% of 500+ scanned servers completely lack authentication | MCP Security Research 2026 |
| MCP auth security | 88% of MCP servers require credentials; 53% rely on insecure long-lived static secrets; only 8.5% use modern OAuth | Astrix State of MCP Security 2025 |
| MCP eval() epidemic | 7 RCE CVEs in February 2026 alone — all from unsanitized user input to eval()/exec() | DEV Community 2026 |
| MCP SDK-level flaws | CVE-2026-25536 (TypeScript SDK cross-client data leak), CVE-2026-29787 (memory service info disclosure) | Security advisories 2026 |
| MCP classic vuln patterns | 82% Path Traversal, 67% Code Injection, 34% Command Injection across 2,614 MCP implementations | Endor Labs 2026 |
| Enkrypt AI MCP scan results | Top 1,000 MCP servers: **33% critical vulns**, averaging **5.2 vulns per server** | Enkrypt AI 2026 |
| MCPTox benchmark | Tool poisoning attack success rates exceed **60%** across 1,312 malicious test cases on GPT-4o-mini, o1-mini, DeepSeek-R1, Phi-4 | arXiv 2026 |
| ClawJacked WebSocket hijack | Malicious websites hijack local AI agents via WebSocket — full remote control of localhost agents | The Hacker News Feb 2026 |
| OpenClaw supply chain crisis | 1,184+ malicious skills across ClawHub; 8 critical CVEs in 6 weeks; 42,665 exposed instances; 824+ actively malicious skills out of 10,700+; infostealers now target OpenClaw file paths | Security research 2025-2026 |
| ToxicSkills ecosystem audit | 3,984 agent skills audited: 36% contain prompt injection, 1,467 malicious payloads, 13.4% critical-level issues; SKILL.md to shell access in 3 lines of markdown | Snyk Feb 2026 |
| CoSAI MCP Security Whitepaper | 12 core threat categories, ~40 distinct threats; developed by EY, Google, IBM, Meta, Microsoft, NVIDIA, PayPal, Snyk, Trend Micro, Zscaler | CoSAI Jan 2026 |
| OWASP Agentic Top 10 2026 | ASI01-ASI10: Goal Hijacking, Tool Misuse, Identity/Privilege Abuse, Supply Chain, Code Execution, Memory Poisoning, Inter-Agent Comms, Cascading Failures, Trust Exploitation, Rogue Agents | OWASP December 2025 |
| OWASP MCP Top 10 | MCP01-MCP10: Token Mismanagement, Privilege Escalation, Command Injection, Supply Chain, Auth, Tool Poisoning, Shadow Servers, Insecure Data, Input Validation, Logging | OWASP 2026 |
| OWASP AIVSS | AI Vulnerability Scoring System v0.5 — extends CVSS for AI risks (autonomy, non-determinism, tool use); v1.0 at RSA March 2026 | OWASP 2025-2026 |
| OWASP AI Testing Guide | v1.0 — first comprehensive standard for AI Trustworthiness testing | OWASP November 2025 |

---

## Platform & Vendor Statistics

| Metric | Value | Source |
|--------|-------|--------|
| Avg pentest findings | 12 vulns, 16% high/critical | HackerOne 2025 |
| Bug bounty high/critical rate | 25% of submissions | HackerOne 2025 |
| Top vulnerability type (bounty) | XSS (but declining) | HackerOne 2025 |
| Fastest growing vuln class | Authorization flaws (IDOR, BOLA) | HackerOne 2025 |
| Declining vuln classes | XSS, SQLi | HackerOne 2025 |
| Hacker team collaboration | 72% say teams yield better results | Bugcrowd 2026 |
| Hardware vulns | 88% increase | Bugcrowd CISO Report 2025 |
| Network vulns | 2x spike | Bugcrowd CISO Report 2025 |
| Intigriti CVSS V4 adoption | All new submissions paid using CVSS V4 starting 2026; enhanced bonuses for quality/variant research | Intigriti 2026 |
| HackerOne Hai Triage adoption | 90% of HackerOne customers by end of 2025 | HackerOne 2025 |
| Bugcrowd AI Triage Assistant | 98% P1 accuracy, 98% duplicate detection confidence | Bugcrowd Dec 2025 |
| huntr AI/ML fix rate | 50.5% of AI/ML OSS vulnerabilities found through huntr have been fixed; 49.5% remain unpatched | huntr 2025 |
| Exploitation speed | 28.96% of KEVs exploited on or before CVE publication day (up from 23.6% in 2024); 884 KEVs in 2025; vuln exploitation was #1 cause of incidents (40%) | VulnCheck / IBM X-Force 2026 |
| CISA KEV catalog additions | 245 vulnerabilities in 2025 (30%+ increase over 2023-2024) | CISA 2025 |
| API-related CVEs | 17% of all 2025 published security bulletins (11,053 of 67,058) | Wallarm 2025 |
| API-related CISA KEVs | 43% of newly added CISA KEVs in 2025 were API-related | Wallarm 2025 |
| API security detection gap | Only 21% of orgs detect attacks at API layer; only 13% can prevent >50% of API attacks; 97% of API vulns exploitable with single request | 42Crunch 2026 |
| CVEs disclosed in H1 2026 | 21,500+ (16-18% increase over 2024) — unprecedented vulnerability volume | NVD H1 2026 |
| FIRST CVE forecast 2026 | Predicted median 59,427 new CVEs for 2026 (first year >50K); realistic scenarios suggest 70-100K possible | FIRST Feb 2026 |
| IBM X-Force 2026 | 44% increase in public-facing app attacks; vuln exploitation now leading attack cause (40%); 300K+ ChatGPT creds on dark web; ~4x supply chain compromise increase since 2020 | IBM X-Force Feb 2026 |
| MITRE CWE Top 25 (2025) | CWE-79 (XSS) still #1 but CWE-862 (Missing Authorization) climbed 5 positions — biggest mover. New entries include CWE-770 (Allocation of Resources Without Limits), CWE-639 (Authorization Bypass Through User-Controlled Key) | MITRE 2025 |
| Salami slicing attacks | Gradual constraint bypass via incremental interactions over 1-3 weeks; procurement agent manipulated to approve $5M fraud | Repello AI / Palo Alto 2026 |
| LPCI (novel vuln class) | Logic-Layer Prompt Control Injection — persistent, conditionally-triggered payloads in agent memory; QSAF reduces success from 43% to 5.3% | CSA Feb 2026, arXiv:2507.10457 |
| NIST/ISO AI gaps | AI RMF and ISO 42001 do not yet address technical controls for agentic deployments | Standards analysis 2026 |

---

## Notable Programs & Expansions

Notable new programs and expansions (2025-2026):
- **Apple Security Bounty Evolved** (November 2025): max reward doubled to **$2M** for zero-click remote exploits (from $1M), with bonuses potentially exceeding $5M. New categories: WebKit sandbox escapes (up to $300K), wireless proximity exploits over any radio (up to $1M). "Target Flags" system for accelerated payouts processed before a fix is available. $35M total paid to date
- Samsung: up to $1M for critical mobile vulns (Knox Vault, TEEGRIS OS bypasses)
- Crypto.com: up to $2M for critical security vulns
- **Microsoft Zero Day Quest expanded**: up to **$5M total bounty pool** for 2026 live hacking event at Redmond campus (Feb-Mar 2026), targeting Azure, Copilot, M365, Identity. 50% bounty multiplier for critical AI/cloud vulnerabilities. Paid $1.6M in inaugural 2025 event. Microsoft "In Scope by Default" — all products/services eligible even without formal bounty program. $17M total to 344 researchers in 2025
- **Google AI Vulnerability Reward Program** (October 2025): dedicated AI VRP covering Google Search, Gemini Apps, and Workspace — up to $30K per finding. Chrome vulnerability rewards increased to $250,000 for highest category
- **Nvidia + Intigriti** — Bug bounty + VDP partnership; private program covers core AI assets; 125K+ ethical hackers
- **OpenAI increased to $100K** — 5x increase for critical infrastructure flaws. Time-limited IDOR bonus: doubled payouts up to $13K through April 30. Specialized Bio Bug Bounty: $25K for universal jailbreaks of bio/chem safety filters, $10K for multiple jailbreak prompts
- Anthropic on HackerOne: up to $15K for universal jailbreaks on Constitutional Classifiers; 405 participants, 3,000+ hours of red teaming
- **Bugcrowd Hacker Showdown** (2nd annual, 2025): "The Mind Cathedral" themed, $30K grand prize for teams of 2-3 hackers
- **Usual (crypto)** partnered with Sherlock for $16M stablecoin codebase audit — **largest single bug bounty prize in history**
- **Intigriti** won Security Innovation of the Year at the 2025 UK IT Industry Awards; adopted **CVSS V4 for all new submissions** starting 2026; enhanced bounty/bonus criteria evaluating quality, variant research, and exceptional PoCs; Google Cloud Apigee cross-tenant vulnerability (CVE-2025-13292) disclosed through Intigriti; hosts Nvidia bug bounty
- **Jenkins** (December 2025): launched new bug bounty program on YesWeHack, backed by the European Commission
- Czech Republic government: Public sector bug bounty via Hackrate
- huntr: world's first bug bounty platform specifically for AI/ML vulnerabilities
- **Immunefi**: surpassed **$100M in total payouts**. Smart contract bugs represent 77.5% ($78M) of total payouts. Largest single payout: $10M (Wormhole critical bug). Undisputed leader in Web3/blockchain bounties
- **curl bug bounty shutdown** (January 2026): Daniel Stenberg ended program after AI-generated submissions overwhelmed team — only 5% of 2025 submissions were genuine vulnerabilities. First major program shutdown attributed to AI slop. Paid $90K+ for 81 genuine vulnerabilities before closing. 20 submissions in just the first weeks of 2026 (7 in a single 16-hour period), none identifying real vulnerabilities. Reports via GitHub directly now, no monetary rewards
- **HackerOne Good Faith AI Research Safe Harbor** (January 20, 2026): clarifies legal protections for researchers conducting authorized AI security testing
- **HackerOne AI Policy Update** (February 2026): researcher submissions are NOT used to train AI models — addresses researcher concerns about data usage
- **HackerOne Agentic PTaaS** (January 2026): combines autonomous agent execution with human expertise for continuous testing; **88% accurate**, fix-verified findings with low false positives; trained on proprietary exploit intelligence
- **IBM Granite AI Bug Bounty** (HackerOne, 2026): up to **$100K in bounties** to test IBM Granite AI models; focus on jailbreaks and flaws in enterprise environments with Granite Guardian guardrails active; select researchers invited
- **0din** (Mozilla): GenAI-focused bug bounty covering GPT-4, Gemini, LLaMa, Claude. Bounties $500-$15,000. Launched **Agent 0DIN** gamified CTF for prompt injection/jailbreaking training
- **OpenAI Codex Security** launched publicly March 6, 2026 (formerly Aardvark) — 14 CVEs discovered across OpenSSH, GnuTLS, GOGS, Chromium; available to ChatGPT Enterprise/Business/Edu
- **Amazon Nova** — Private AI bug bounty; prompt injection, jailbreaks, exploitable model behavior; $55K+ paid, 30+ validated findings (Amazon Science, early 2026)
- **Microsoft Copilot added moderate payouts** — up to $5K (previously $0 for moderate). Range now $250-$30K. Expanded to Copilot for Telegram, WhatsApp. 2026 Research Challenge and Live Hacking Event announced
- **Microsoft "In Scope by Default" expansion**: All online services now in scope; new services auto-included on launch; **third-party and open source code** impacting Microsoft services now eligible for bounties; any critical vulnerability with demonstrable impact qualifies regardless of code origin; $17M+ paid to 344 researchers in 2025
- **Cloudflare public bug bounty launch** (2026): Cloudflare launched its paid public bug bounty program after years of operating a private program — high-value target covering CDN, DNS, Workers, R2, and zero trust products
- **NEAR Intents bridge bug bounty** (March 3, 2026): covers critical cross-chain infrastructure including MPC network for chain signatures and bridge protocols; cross-chain bridges are historically high-value targets (Wormhole $10M, Ronin $600M incident)
- **Bugcrowd FedRAMP Authorization**: FedRAMP Moderate Authorization sponsored by CISA — federal agencies can now deploy Bugcrowd at scale
- **Salesforce Bug Bounty 10th Anniversary** (March 4, 2026): $30.4M invested since 2015
- **Google AI VRP** (Oct 2025): dedicated AI bug bounty covering Search, Gemini Apps, Workspace core apps; up to $30K per report with novelty bonuses; $430K+ paid for AI bugs; prompt injection/jailbreaks excluded from scope but encouraged as research
- **HackerOne Hai AI validation agent** (Feb 2026): 56% reduction in vulnerability validation time; updated AI policy confirms no training on researcher submissions
- **Intigriti retesting feature**: New submission retesting for validating fixes across all program types with one click
- **Vercel React2Shell WAF Bypass program**: **$1M total bounty** ($50K per unique bypass technique); 116 researchers, 156 reports, 20 unique bypass techniques validated; Vercel blocked 6M+ exploit attempts. One of the fastest public HackerOne programs ever launched
- **Intigriti hourly payout model** (2026): new hybrid compensation paying hunters based on hours (like pentesting) in addition to per-vulnerability rewards — signals industry shift toward valuing researcher time, not just findings
- **HackerOne Hai Replay** (2026): dashboard feature turning historical vulnerability data into actionable insights for future testing
- **Apple Security Research Device 2026**: iPhone 17 with Memory Integrity Enforcement; expanded bounty categories; "Target Flags" for objective exploitability demonstration; max $5M+ with bonuses
- **NVIDIA private AI bounty on Intigriti**: explicit AI-focused private program + public VDP for all NVIDIA assets; high-value AI infrastructure target

---

## Notable Disclosed Vulnerabilities

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
- **HackerOne Export Feature Information Disclosure ($10K)**: "Export as .zip" feature allowed downloading private comments from partially disclosed reports — new feature deployed Monday, patched Tuesday. Pattern: test every new feature against existing access control models
- **SmarterMail Authentication Bypass (WT-2026-0001)**: WatchTowr Labs — admin password reset chain to OS command execution; in-the-wild exploitation triggered emergency patch. Pattern: password reset flows remain high-value targets
- **Instagram Private Post Data Leak**: authorization failure returned private post data when specific mobile headers were used — pattern: test API endpoints with mobile-specific headers to bypass web access controls
- **Argo CD DoS (CVE-2025-59538/59531, $8,500)**: unauthenticated DoS in Kubernetes GitOps tool via HackerOne IBB; pattern: K8s ecosystem infrastructure components are underexplored
- Anthropic Filesystem-MCP: sandbox escape + symlink bypass enabling arbitrary file access and code execution
- Microsoft MarkItDown MCP server: unpatched SSRF that can compromise AWS EC2 instances via metadata service exploitation
- OpenSSL: 12 zero-day vulnerabilities discovered by AISLE AI system, including CVE-2025-15467 (stack buffer overflow, remotely exploitable); three bugs dated back to 1998-2000
- SSRF protection bypass in AutoGPT reported through huntr platform
- Multiple Salesforce Tableau Server vulns including unrestricted file upload -> RCE (CVE-2025-52449)
- WhatsApp MCP tool poisoning: Invariant Labs demonstrated malicious MCP server silently exfiltrating user's entire WhatsApp history via tool poisoning combined with legitimate whatsapp-mcp server
- CVE-2025-37899: Linux kernel SMB use-after-free zero-day found with OpenAI o3 assistance by Sean Heelan
- CVE-2025-43714: ChatGPT SVG vulnerability — crafted SVG executed arbitrary HTML/JS in user's browser within ChatGPT's preview window; formal CVE assignment coordinated with MITRE/NVD
- CVE-2025-68664: LangChain "LangGrinch" — prompt injection vulnerability in LangChain Core; $4,000 bounty (maximum ever awarded in the project)
- CVE-2025-49596: Critical RCE in Anthropic's MCP Inspector (CVSS 9.4) — one of the first critical RCEs in Anthropic's MCP ecosystem (Oligo Security)
- CVE-2025-53967: Figma MCP server RCE — command injection via unvalidated user input in shell commands
- CVE-2026-2256: ModelScope MS-Agent command injection (CVSS 9.8) — regex denylist bypass via command obfuscation; prompt injection -> full system compromise; public PoC on GitHub
- CVE-2026-22812: OpenCode unauthenticated RCE (CVSS 8.8) — AI coding agent HTTP server with permissive CORS allows any website to execute shell commands
- CVE-2026-25049: n8n expression sandbox escape (CVSS 9.4) — TypeScript type confusion; destructuring bypasses sanitization for system command execution
- CVE-2026-21877: n8n Git Node RCE affecting self-hosted and Cloud instances; part of 6-CVE batch disclosure
- CVE-2026-27001/27002: OpenClaw sandbox escape + Docker container escape; workspace path injection and dangerous Docker config options
- Cursor Workspace Trust disabled by default — silent code execution via `.vscode/tasks.json` (Oasis Security)
- ChatGPT message limit bypass: business-logic vulnerability allowed free-tier users to bypass GPT-5 message limits by editing messages in older conversations; silently patched 69 days after submission
- Google Cloud Apigee CVE-2025-13292: cross-tenant vulnerability providing read/write access to analytics data across thousands of organizations (Focal Security)
- CVE-2026-25536: MCP TypeScript SDK cross-client data leak — responses leak across client boundaries when McpServer instance is reused
- CVE-2026-29787: MCP Memory Service info disclosure — `/api/health/detailed` leaks OS, CPU, memory, database path without auth
- CVE-2026-27896: MCP Go SDK vulnerability — JSON parser handles field names case-insensitively, enabling crafted malicious MCP responses
- CVE-2025-53109: critical symlink bypass in MCP file operations — system takeover if server runs with elevated privileges
- ClawJacked (February 2026): WebSocket hijacking of local OpenClaw AI agent instances — malicious websites gain full remote control via localhost WebSocket
- Persistent prompt injection in procurement agent (Palo Alto Unit42, 2026): manufacturing company's agent manipulated over 3 weeks to approve $5M in false purchase orders
- OmniGPT breach (February 2026): threat actor breached AI aggregator exposing **34 million lines of conversations**, 30,000 user credentials, and uploaded business documents
- CVE-2025-32711 (EchoLeak, CVSS 9.3): First real-world zero-click prompt injection in production LLM — attacker email -> Copilot retrieves -> exfiltrates internal files with no user interaction; bypassed XPIA classifier, link redaction, and CSP via Teams proxy
- CVE-2025-34291: Langflow permissive CORS + missing CSRF enabling session hijack -> RCE via code validation endpoint
- CVE-2025-64106: Cursor IDE MCP installation flow vulnerability (CVSS 8.8) — arbitrary command execution via trust assumption abuse (Check Point)
- CVE-2025-54135/54136: Cursor MCPoison — persistent code execution via MCP trust bypass (Check Point)
- ContextCrush (Noma Labs, Feb 2026): Context7 MCP server supply chain — malicious "Custom Rules" served as trusted documentation to AI coding assistants, enabling env file theft and file deletion
- Clawdbot/Moltbot (Jan 2026): viral AI agent framework exposed 2,000+ MCP gateways on Shodan; infostealers added to target lists within 72 hours; exposed API keys, OAuth tokens, conversation histories
- PerplexedBrowser / PleaseFix (Zenity Labs, Mar 2026): zero-click agentic browser vulnerabilities in Perplexity Comet — file system exfiltration and 1Password vault credential theft via calendar invite -> autonomous agent manipulation
- Anthropic AI Espionage Disruption (Feb 2026): first documented large-scale AI-orchestrated cyberattack — Chinese state actors jailbroke Claude Code as autonomous pentest orchestrator; 150GB of Mexican government data (195M taxpayer records); AI executed 80-90% of operations; 30+ global targets autonomously discovered and exploited
- ToxicSkills (Snyk, Feb 2026): comprehensive audit of 3,984 ClawHub skills found 36% with prompt injection, 1,467 malicious payloads; SKILL.md to shell access in 3 lines of markdown
- CVE-2025-55241 (Microsoft Entra ID, CVSS 10.0): Global Administrator privilege escalation via Actor Tokens authentication mechanism — cloud identity total compromise
- CVE-2026-0830 (Kiro/AWS): command injection leading to RCE in AWS AI coding IDE
- IDEsaster campaign: 30+ vulnerabilities across 10+ AI coding tools (Claude Code, Cursor, Kiro, Windsurf), 24 CVEs by researcher Ari Marzouk
- CVE-2025-7656 (Chromium in Cursor/Windsurf): 94+ Chromium vulnerabilities in AI IDEs due to legacy Chromium builds, affecting 1.8M developers
- Extension Recommendation Attacks: AI coding IDEs (Cursor, Windsurf, Trae, Google Antigravity) recommend non-existent extensions from OpenVSX — threat actors claim namespaces and serve malware
- CVE-2026-29783 (Copilot CLI, March 2026): shell expansion RCE via `env` allowlist bypass — malware download with zero approval via poisoned README prompt injection (PromptArmor)
- CVE-2026-0628 (GlicJack, Unit42): Chrome Gemini Live panel hijack — malicious extensions gain camera/microphone/file access (CVSS 8.8)
- CVE-2026-22218/22219 (ChainLeak, Zafran Labs): Chainlit file read + SSRF -> full cloud compromise with no user interaction
- CVE-2026-27795 (LangChain): RecursiveUrlLoader SSRF via HTTP redirect to AWS metadata (169.254.169.254)
- CVE-2026-25253 (OpenClaw): 1-click RCE via `gatewayUrl` query parameter WebSocket credential relay (CVSS 8.8)
- CVE-2026-0621 (MCP TypeScript SDK): ReDoS via URI template regex catastrophic backtracking (CVSS 8.7)
- SANDWORM_MODE (Socket, Feb 2026): npm supply chain worm with McpInject module — deploys rogue MCP servers into AI coding assistant configs; steals credentials for 9 LLM providers
- ForcedLeak (Varonis, March 2026): Salesforce Agentforce CRM data exfiltration via Web-to-Lead form prompt injection (CVSS 9.4)
- DockerDash (Noma Security, Feb 2026): Docker image label injection -> Ask Gordon AI -> MCP Gateway compromise; new "Meta-Context Injection" vulnerability class
- PleaseFix/PerplexedBrowser (Zenity Labs, March 2026): zero-click agentic browser hijack — file system exfil + 1Password credential theft via calendar invite
- Notion AI Data Exfiltration (PromptArmor, Jan 2026): white text prompt injection exfiltrates salary data via invisible image requests; initially closed as N/A
- Uncrew (Noma Security): CrewAI CVSS 9.2 — leaked GitHub token with full admin privileges across entire GitHub org during provisioning failure
- Operation Bizarre Bazaar (Pillar Security, Jan 2026): first attributed LLMjacking campaign; 35,000 sessions; commercial marketplace monetizing unauthorized LLM/MCP access
- Shai-Hulud supply chain worm: multi-wave npm campaign; 454,648 malicious npm packages in 2025; s1ngularity campaign harvested 2,349 credentials from 1,079 dev systems
- AWS CodeBuild vulnerability (Wiz): critical flaw allowing hijacking of official AWS GitHub repositories and leaking secrets from build logs
- ServiceNow Now Assist second-order prompt injection: first documented cross-agent privilege escalation in production multi-agent system
- AI Recommendation Poisoning (Microsoft, Feb 2026): 50+ poisoning prompts from 31 companies embedded in web content to manipulate AI assistants on health, finance, security topics
- FortiGate AI-Augmented Breach (Jan-Feb 2026): 600+ devices across 55+ countries compromised using multiple commercial GenAI services; data exfiltration within 4 minutes of initial access
- React2Shell (CVE-2025-55182): CVSS 10.0: pre-auth RCE in React Server Components; #1 most exploited CVE on HackerOne; exploited in-the-wild within hours
- Microsoft Entra ID (CVE-2025-55241): CVSS 10.0: Global Admin takeover via Actor Tokens; cloud identity total compromise
- n8n Ni8mare (CVE-2026-21858): CVSS 10.0: unauthenticated RCE in ~100K workflow automation servers via Content-Type confusion
- pac4j-jwt JWE-wrapped PlainJWT bypass (CVE-2026-29000): CVSS 10.0: complete authentication bypass
- Reprompt (Varonis -> Microsoft, patched Jan 13, 2026): single-click Microsoft Copilot data exfiltration via URL parameter injection
- OpenClaw Browser Relay (CVE-2026-28458, CVSS 7.5) and Sandbox Bridge (CVE-2026-28468): unauthenticated WebSocket/HTTP endpoints enabling session theft; total OpenClaw CVE count now exceeds 12
- AI app breach epidemic (Barrack.ai): 20+ breaches since Jan 2025 from misconfigured Firebase, missing Supabase RLS, hardcoded keys; vibe-coded AI wrappers represent a systemic, structural security crisis with hundreds of millions of records exposed
- CVE-2026-30861 (WeKnora MCP Stdio RCE, CVSS 9.9): blacklisted argument bypass via `-p` flag enabling arbitrary code execution
- CVE-2026-30860 (WeKnora Database Query RCE, CVSS 9.9): PostgreSQL expression bypass enabling remote code execution
- CVE-2026-23744 (MCPJam Inspector RCE, CVSS 9.8): unauthenticated endpoint on 0.0.0.0 installs arbitrary MCP servers
- CVE-2025-59536 (Claude Code config injection, CVSS 8.7): project file manipulation for RCE and API token exfiltration (Check Point Research)
- CVE-2026-21516/21523/21256 (GitHub Copilot, Feb 2026 Patch Tuesday): prompt injection -> command injection chain across Copilot IDE extensions
- CVE-2026-1470 (n8n `with` statement, CVSS 9.9): deprecated JavaScript `with` feature enables sandbox escape and arbitrary code execution
- ContextCrush (Noma Labs, Feb 2026): Context7 MCP server custom rules served verbatim to AI coding assistants; environment file theft via supply chain poisoning
- CVE-2026-1731 (BeyondTrust Remote Support, CVSS 9.9): pre-auth RCE discovered by Hacktron AI — AI-augmented research team model for critical findings
- CVE-2025-68145/68143/68144 (Anthropic mcp-server-git): three chained CVEs enabling RCE via indirect prompt injection through flag bypass, unvalidated paths, and argument injection
- CVE-2026-25536 (MCP TypeScript SDK, CVSS 7.1): cross-client data leak when McpServer instance reused across multiple clients; affects SDK v1.10.0-1.25.3; fixed v1.26.0
- MCP ecosystem crisis confirmed (March 2026): **30 CVEs in 60 days**; Enkrypt AI scan of 1,000 servers found **33% critically vulnerable** averaging **5.2 vulns per server**; compound exploit probability hits 92% at 10 plugins
- CVE-2026-30855 (WeKnora Multi-Tenant Auth Bypass, CVSS 8.8): broken access control in Tencent's LLM framework; open registration → cross-tenant ATO + API key leakage; 5 CVEs total including DNS rebinding SSRF and cross-tenant KB cloning
- RustDesk 7+ CVE cluster (March 5, 2026): CVE-2026-30789 (session replay auth bypass), CVE-2026-30784 (missing authorization), CVE-2026-30797 (MitM API manipulation), plus pre-auth SSRF; self-hosted remote desktop tool with many Shodan-exposed instances
- CVE-2026-20079 + CVE-2026-20131 (Cisco Secure Firewall Management Center, both **CVSS 10.0**, March 4 2026): authentication bypass → root + Java deserialization RCE → root; affects on-premises FMC and Cisco Security Cloud Control; no workarounds — patching only
- CVE-2026-22719 (VMware Aria Operations, CVSS 8.1): command injection during support-assisted migration; actively exploited in the wild; added to CISA KEV March 3 2026; root access → full virtual infrastructure compromise
- CVE-2026-21385 (Qualcomm Android, CVSS 7.8): integer overflow in graphics/display component affecting 230+ chipset models; limited targeted exploitation (likely spyware/nation-state); part of Google March 2026 Android update patching 129 vulnerabilities including critical System RCE (CVE-2026-0006)
- Image-based prompt injection (arXiv:2603.03637, March 2026): black-box attack embeds instructions in natural images invisible to humans; 64% success rate against GPT-4-turbo — new multimodal attack vector
- Trail of Bits Comet audit (February 20, 2026): 4 prompt injection techniques against Perplexity's agentic browser exfiltrating private Gmail data; TRAIL trust zone taxonomy defines INJECTION/CTX_IN/CTX_OUT/REV_CTX_IN violation categories
- CVE-2026-22807 (vLLM, Critical): AI inference engine loads Hugging Face `auto_map` dynamic modules without `trust_remote_code` gating — attacker-controlled Python executes at server startup. Pattern: model metadata as executable code. Fixed v0.14.0
- CVE-2026-23947 (Orval OpenAPI codegen): `x-enumDescriptions` in untrusted OpenAPI specs inject arbitrary TypeScript/JavaScript into generated clients via JSDoc closer technique. Pattern: code generators processing untrusted API specs
- CVE-2026-27826 (mcp-atlassian SSRF): high-severity SSRF via header-controlled Atlassian base URLs; companion to RCE CVE-2026-27825
- CVE-2026-24457 (OpenMQ, CVSS 9.1): arbitrary file read + potential RCE via unsafe parsing in OpenMQ Broker; message broker as file system access vector
- Endor Labs "classic vulns meet AI infra" (March 2026): MCP servers inherit CWE-22/CWE-77 at scale; 82% of 2,614 implementations vulnerable to path traversal; root cause is architectural assumption that LLM inputs are trusted
- VDP as compliance requirement (March 2026): NIST CSF 2.0, ISO 27001, EU CRA all reference coordinated vulnerability disclosure; not having VDP now equivalent to missing privacy policy
- CVE-2026-28514 (Rocket.Chat, CVSS 9.3): critical authentication bypass — missing `await` on async `bcrypt.compare()` in microservices account-service; un-awaited Promise evaluates truthy → any password accepted for any user with bcrypt hash set; discovered by GitHub Security Lab Taskflow Agent. Pattern: async/await bugs in auth paths
- EU Cyber Resilience Act enforcement (September 11, 2026): mandatory vulnerability reporting for all products with digital elements sold in EU; actively exploited vulnerabilities must be reported to ENISA within 24 hours; manufacturers must implement coordinated vulnerability disclosure processes

---

## Hacker Demographics

Hacker demographics (Bugcrowd Inside the Mind of a Hacker 2026, 2,000+ surveyed):
- 92% of hackers are 34 or younger; 69% hold a college degree or higher
- 20% identify as neurodivergent
- 74% are motivated by financial gain, but 85% say reporting a critical vulnerability matters more than money
- 65% have chosen NOT to disclose a vulnerability because the company lacked a clear, safe reporting pathway
- Continuous assurance testing is replacing annual pentests as the industry standard
- 98% proud of their work; 85% say reporting critical vulns > money
- 56% say geopolitics outweighs curiosity as driving factor
- 75% say hacking is becoming more about money than curiosity
- AI-induced job scarcity driving new influx into security freelancing and bug bounty

---

## Competition Landscape

Competition awareness:
- XBOW reached #1 on both US and global HackerOne leaderboards with 1,400+ zero-day submissions, running 80x faster than manual teams; raised **$117M total** ($75M Series B led by Altimeter/Sequoia); expanding APAC presence in 2026
- XBOW benchmark: **75% of standard web security challenges** solved autonomously, **85% of custom-built challenges** — uses automated "validator" peer reviewers to confirm each finding
- HackerOne **split leaderboards** to separate individuals from companies/agents like XBOW
- XBOW launched **Pentest On-Demand** — fully automated pentest service delivering results within 5 business days
- 82% of researchers now use AI tools in hunting (Bugcrowd 2026)
- **Shannon** emerged as GitHub's fastest-rising security project (96.15% on XBOW benchmark, fully autonomous white-box pentesting; subscription pricing $0-$49.99/mo, not per-run)
- **Strix** emerged as leading open-source autonomous AI pentest tool (~2K GitHub stars, used by Fortune 500 security teams and top 1% HackerOne hunters)
- **NeuroSploit v3** introduced autonomous pentesting with exploit chaining and anti-hallucination safeguards
- Programs with fewer researchers (avg 56 vs 97) tend to have higher-impact submissions
- Bugcrowd 2026 prediction: high-end vulnerability research will become more valuable as AI handles low-hanging fruit
- **Novee Security** emerged from stealth ($51.5M, Jan 2026) — proprietary offensive AI model for continuous pentesting; Unit 8200/Talpiot founders
- **Maze** ($31M funded) — AI agents for vuln management proving 80-90% of cloud findings are false positives
- **Bugcrowd achieved FedRAMP** — federal agencies now deploying crowdsourced security at scale
- **Apple doubled max bounty to $2M+** (with bonuses exceeding $5M) — highest in the industry; $35M total paid
- **Microsoft "In Scope by Default"** — all products/services eligible even without formal bounty program
- **IBM X-Force**: vulnerability exploitation now #1 cause of attacks (40%); supply chain compromises quadrupled since 2020
- **CAI** reached top-30 in Spain, top-500 worldwide on HTB within a week; **156x cost reduction** vs traditional testing; enables non-professionals to find CVSS 4.3-7.5 bugs at expert rates
- **Apiiro research**: AI coding assistants produce 10x more security findings per month; 322% increase in privilege escalation bugs; 2x credential exposure — AI-generated code is expanding the attack surface for human hunters
- AI coding agent bug submission prediction: 2x bug submissions in 2026 vs prior year driven by coding agents (Joseph Thacker)
- curl bug bounty shutdown (Jan 2026) demonstrates AI slop risk — first major program closed due to AI-generated garbage flooding; confirmed-rate dropped below 5% from 15%+ due to AI slop
- **AISLE** found 100+ CVEs across 30+ projects including all 12 OpenSSL zero-days; demonstrated full-loop (discover, validate, patch) — sets new bar for AI-discovered bugs
- **OpenAI Codex Security**: 1.2M commits scanned, 792 critical findings; 84% noise reduction; discovered 14 CVEs across OpenSSH, GnuTLS, GOGS, Chromium, PHP, libssh
- **Claude + Mozilla Firefox**: 14 high-severity bugs, 22 CVEs; fixes in Firefox 148 (Feb 24, 2026); 500+ total across OSS codebases
- **Hacktron AI**: $10K bounty for Google Antigravity RCE; also hacked Cursor, Windsurf, Perplexity Comet, OpenAI Atlas — emerging AI bug bounty research team

---

## AI Security Tools & Startups

- **AISLE** found all 12 OpenSSL zero-days (Jan 2026), 100+ CVEs across 30+ projects including all 14 OpenSSL CVEs in 2025; demonstrated full-loop capability (discover, validate, patch) — sets new bar for AI-discovered bugs
- **Codex Security** (OpenAI, formerly Aardvark) — launched publicly **March 6, 2026**; **92% recall** on golden benchmarks; 1.2M commits scanned, 792 critical findings; 84% noise reduction; reported **14 CVEs** across OpenSSH, GnuTLS, GOGS, Chromium, PHP, libssh; **$10M in API credits** committed for open-source security; available to ChatGPT Enterprise/Business/Edu
- **Claude Code Security** launched Feb 20, 2026 — found 14 high-severity bugs and 22 CVEs in Firefox (fixes in Firefox 148); 500+ vulnerabilities total in production open-source codebases; Mozilla Firefox partnership; bugs undetected for decades
- **ZAST.AI** raised $10M total, 119 CVEs assigned, "zero false positive" approach — new competitor for automated code scanning
- **Trend Micro AESIR** — AI-powered zero-day discovery (21 CVEs across NVIDIA, Tencent, MLflow, MCP tooling)
- **Semgrep AI-Powered Detection** — multimodal AppSec engine: deterministic + LLM reasoning; detects IDORs, broken auth; filters 1 in 5 false positives
- **Novee Security** — **$51.5M** Seed + Series A; AI penetration testing platform from Unit 8200/Talpiot veterans; proprietary model achieves **90% accuracy** on constrained web exploitation (55%+ better than frontier LLMs at ~65%)
- **Maze** — **$31M** funded; AI agents model analyst workflows; proves 80-90% of findings are false positives
- **Terra Security** raised **$38M total** (Series A $30M led by Felicis, Sep 2025) — first agentic-AI continuous pentesting platform, Fortune 100 clients
- **DeepKeep** launched AI agent attack surface scanner (March 2026) — maps enterprise risk across Microsoft, Agentforce, OpenAI Agents, CrewAI, Amazon Bedrock AgentCore, n8n, and Make frameworks
- **Enkrypt AI MCP Scanner** — top 1,000 MCP servers: 33% critical vulns, averaging 5.2 vulns per server; 43% high-risk rate
- **Cisco MCP Scanner** — open source, 3 scanning engines
- **Snyk Agent Scan** — scans AI agents, MCP servers, Claude Skills; auto-discovers Claude Code/Desktop, Cursor, Gemini CLI, Windsurf configs; background mode for company-wide monitoring
- **Akto** — AI agent security platform with 1,000+ real-world agent exploits; continuous red teaming + runtime guardrails; recognized by Gartner
- **Repello SkillCheck** — browser-based skill scanner; upload skill zip, get security score (0-100); catches prompt injection, env var exfiltration, payload delivery
- **mcp-scan -> Snyk**: Invariant Labs mcp-scan (standard MCP security scanner) acquired by Snyk; v0.4.2 released Feb 2026
- **MCPScan.ai** — MCP server security scanning
- **AgentShield Benchmark** — first open benchmark for 6 AI agent security tools; 537 test cases; 95%+ PI detection but weak tool abuse detection
- **Endor Labs AURI** — free MCP-integrated security intelligence for agentic coding; found 7 OpenClaw vulns; only 10% of AI-generated code is secure
- **HexStrike AI MCP server** — enables AI agents to autonomously run 150+ security tools

---

## Autonomous Pentesting Tools

- **AWS Security Agent** (preview re:Invent Dec 2025) — multi-agent continuous on-demand pentesting
- **Hadrian** — attack-surface-driven 24/7 autonomous pentesting; real-time testing triggered by attack surface changes
- **BlacksmithAI** (March 2026) — open-source multi-agent pentesting framework; each agent specializes in different assessment phases
- **PentAGI** (Feb 2026) — Go-based multi-agent pentesting with PostgreSQL + Neo4j; 20+ tools in isolated Docker environments
- **Assail Ares** launched from stealth (Jan 2026) — autonomous API/web/mobile pentesting
- **Zen-AI-Pentest** — wraps 20+ tools (Nmap, SQLMap, Metasploit, Burp, Nuclei, BloodHound) with AI orchestration; CI/CD integration
- **Penligent.ai** — "first true end-to-end AI pentest agent" with natural language interface and OODA loop feedback; autonomous payload generation
- **RunSybil** — AI-driven pentesting with coordinated autonomous agents; built by OpenAI/Bishop Fox/CrowdStrike alumni
- **Shannon** — fully autonomous white-box pentesting; 96.15% on XBOW benchmark; GitHub's fastest-rising security project
- **Aikido Infinite/Attack** — autonomous agents pentest every deployment, validate, remediate, retest; found 7 CVEs in Coolify including RCE as root across 52,000+ instances
- **Shannon AI bug bounty service** — AI-powered recon, exploitation, and report writing for HackerOne, Bugcrowd, Intigriti programs
- **Burp Suite with Burp AI** (Professional 2025.2+) — Explore Issue autonomously investigates scanner findings; Explainer generates AI explanations in Repeater; BAC false positive reduction
- **OWASP ZAP 2.17.0** (Dec 2025) — MCP integration for AI-powered web security testing; free/open-source AI-augmented scanning
- **Shift for Caido** — AI plugin for Caido proxy: English-command control, context-aware wordlists, AI-powered HTTP modification
- **Cecuro AI** — DeFi security agent (92% detection in exploited contracts)
- By 2027, "manual pentesting" predicted to become a boutique service; 99% of vulnerability assessments will be agentic

---

## Industry Trends & Predictions

- As XSS and SQLi become easier to mitigate, organizations shift rewards toward identity, access, and business logic flaws
- Continuous assurance testing is replacing annual pentests as the industry standard
- **EU AI Act compliance deadline August 2, 2026** — driving mandatory AI red teaming and creating new consulting/testing demand
- **NIST CAISI** launched formal AI agent security standards; RFI on AI Agent Security (Mar 2026); Agent Identity and Authorization concept paper (Apr 2026)
- **Google 2025 zero-day review**: 90 zero-days tracked in 2025; 48% targeted enterprise technology (new high)
- **Vibe Hacking**: attackers use AI to lower hacking barrier; low-effort AI-built attacks beating enterprise defenses
- **A2A Protocol vulnerabilities**: Google Agent-to-Agent protocol: identity spoofing, capability forgery, task chain poisoning, trust graph attacks
- **GRP-Obliteration**: single unlabeled prompt removes LLM safety alignment via inverted GRPO training; GPT-OSS-20B attack success rate: 13% -> 93%
- **Autonomous jailbreak agents**: large reasoning models as autonomous adversaries achieve 97.14% jailbreak success rate across 9 target models
- **ZombieAgent**: zero-click ChatGPT memory poisoning via malicious email -> long-term memory corruption -> self-propagation
- **Side-channel timing attacks**: Whisper Leak achieves >98% AUPRC classification across 28 LLMs via packet timing analysis
- **Inter-agent trust exploitation**: 82.4% of LLMs execute malicious payloads from peer agents they'd refuse from users
- **Cascading failure propagation**: single compromised agent poisons 87% of downstream decisions within 4 hours in multi-agent systems
- **Autonomous ransomware**: first confirmed AI-orchestrated attacks in 2025; fully autonomous pipelines predicted for 2026
- **MCP security is AI's fastest-growing attack surface** — 30+ CVEs in 60 days; 7 RCE CVEs in February 2026 alone
- **Supply chain compromise growth**: nearly quadrupled since 2020 (IBM X-Force 2026)
- **Ransomware group surge**: 49% YoY increase (IBM X-Force 2026)
- **OpenAI Lockdown Mode**: system prompt protection for enterprise deployments
- **Docker MCP Horror Stories**: 5-part series documenting real MCP attacks: WhatsApp exfiltration, GitHub injection, localhost breach, supply chain attack
- **Log-To-Leak attack class**: new MCP privacy attack silently exfiltrates via malicious logging tool; preserves task quality
- **PoisonedRAG**: first knowledge corruption attack on RAG systems; semantic poisoning of retrieval databases
- **MINJA memory poisoning**: 95%+ injection success against production AI agents with persistent memory; temporally decoupled attacks
- **GTG-1002 incident**: first documented state-sponsored espionage primarily orchestrated by AI agent; autonomous Claude Code executed 80-90% of intrusion lifecycle
- **PromptPwnd CI/CD injection**: AI agents in GitHub Actions/GitLab CI leak secrets via prompt injection; 5+ Fortune 500 confirmed affected
- **Rules File Backdoor**: invisible Unicode in `.cursorrules`/`.github/copilot-instructions.md` weaponizes AI IDE config files
- **RoguePilot**: passive prompt injection via GitHub issues -> Copilot GITHUB_TOKEN theft in Codespaces
- **Clinejection**: prompt injection turning AI coding bots into supply chain vectors via GitHub Actions pipelines
- **Semantic chaining jailbreak**: individually benign instructions chained to produce forbidden output; bypasses input AND output filtering
- **H-CoT chain-of-thought hijacking**: OpenAI o1 rejection drops from 99% to <2%; affects o1/o3, DeepSeek-R1, Gemini 2.0 Flash Thinking
- **AgentLeak multi-agent privacy**: multi-agent systems introduce unmonitored inter-agent channels; 27.2% vs 43.2% output leakage
- **OMNI-LEAK cascade injection**: single injection cascades through multi-agent orchestrator pattern; 1/500 success rate leaks data in 5 days
- **Kali MCP server command injection**: Official Kali Linux MCP server has `subprocess` with `shell=True`; textbook command injection
- **FailSafe 2025**: access control flaws caused $953.2M in losses; logic flaws another $63M
- **Shai-Hulud supply chain worm**: 454,648 malicious npm packages in 2025; 99% of all open-source malware
- **IDEsaster campaign**: 30+ vulns across 10+ AI coding tools, 24 CVEs; extension recommendation attacks via OpenVSX
- **Denial-of-wallet via MCP**: 142.4x token amplification via MCP overthinking loops; severe financial risk
- **Invisible Unicode prompt injection**: Unicode tag chars (E0000-E007F) encode hidden instructions in normal text; exploited HackerOne Hai
- **AI Recommendation Poisoning**: 50+ unique poisoning prompts from 31 companies across 14 industries
- **IBM X-Force 2026**: 300K+ ChatGPT credentials exposed via infostealer malware; 44% increase in public-facing app exploitation
- **AISLE AI CVE discoveries**: 100+ CVEs across 30+ projects; 13/14 OpenSSL CVEs in 2025
- **Codex Security refined performance**: discovered 14 CVEs across OpenSSH, GnuTLS, GOGS, Chromium, PHP, libssh
- **Agent skill supply chain threat**: ToxicSkills study found 36% of ClawHub skills contain prompt injection
- **Bugcrowd FedRAMP Moderate** (March 2026): first crowdsourced security platform cleared for U.S. federal agencies, sponsored by CISA
- **HackerOne AI Research Safe Harbor** (January 2026): platform-wide opt-in for AI system testing, separate from Gold Standard Safe Harbor
- **Intigriti Global Hacker Ambassador Program** (March 4, 2026): early access to private programs; 125,000+ ethical hackers
- **NVIDIA private AI bounty on Intigriti**: explicit AI-focused private program + public VDP for all NVIDIA assets
- **Zscaler 2026 AI Security Report**: 100% enterprise failure rate; median 16 min to first critical; AI/ML activity +91% YoY
- **Unit42 in-the-wild IDPI catalog**: 22 distinct payload techniques in production telemetry; first documented ad review evasion
- **Langflow CSV Agent RCE** (CVE-2026-27966, CVSS 9.8): hardcoded `allow_dangerous_code=True` → prompt injection to RCE
- **Joseph Thacker (rez0) critical window**: 2026 is "massively important" — 2x submissions predicted; internal AI code review will commoditize external bug bounty in 1-2 years
- **curl ended bug bounty** (January 2026): first shutdown from AI slop — 8x submission spike, 5% genuine, 0 AI-only real findings in 6 years
- **Wallarm API ThreatStats 2026**: AI-related API threats +400% YoY; 36% of AI vulns are also API vulns
- **Cisco State of AI Security 2026**: AI vulnerabilities from research now materializing in production incidents
- **OpenClaw privacy breach scale**: 42K+ exposed instances, 1.5M leaked API tokens; 93% had critical auth bypass
- **Penetrify autonomous red team** (March 3, 2026): first fully autonomous AI red team; CI/CD integration; from $50/mo
- **GHOSTCREW MCP-based toolkit** (January 2026): open-source, integrates Metasploit/Nmap/SQLMap; 450+ GitHub stars
- **Pynt MCP exploit probability** (March 2026): 10 MCP plugins = 92% exploit probability; 72% of MCPs expose sensitive capabilities; 281 configs analyzed; real exploits observed: HTML→shell, email→code exec, Slack→terminal (VentureBeat/Pynt)
- **Enkrypt AI 1,000 MCP server scan** (March 2026): 33% critical vulns, 5.2 avg per server; K8s MCP server: 26 vulns (6 CVSS 9.8); malicious Postmark MCP exfiltrated emails
- **ChatGPT MCP integration** (March 2026): ChatGPT connects to any remote MCP server; malicious servers are most immediate threat for Enterprise/Business users
- **CVE-2026-23864** (React Server Components DoS, CVSS 7.5): crafted HTTP requests crash Next.js servers via OOM/CPU exhaustion
- **RondoDox botnet**: mass exploitation of React2Shell CVE-2025-55182 against IoT and web servers
- **Apple SRD 2026**: iPhone 17 with Memory Integrity Enforcement; "Target Flags" for objective exploitability; expanded categories
- **Noma x OWASP Red Team Playbook** (March 2026): agentic AI red team framework with dynamic attack generation
- **March 2026 Patch Tuesday**: 2 confirmed zero-days (Windows SmartScreen bypass, Outlook flaw); follows February's 6 actively exploited zero-days
- **Android March 2026**: 129 vulnerabilities patched; Qualcomm zero-day CVE-2026-21385 under active exploitation (234 chipsets)
- **HackerOne $25K self-vuln** (Report #3000510): `.json` endpoint leaked reporters' emails, OTP codes, phone numbers, graphql tokens; fixed in 2 days
- **Dell RecoverPoint CVE-2026-22769** (CVSS 10.0): hardcoded credentials exploited by UNC6201 (China-nexus) since mid-2024
- **BlacksmithAI** (March 2, 2026): open-source multi-agent AI pentesting framework (GPL-3.0); Docker-based with mini-kali container
- **CyberStrikeAI threat intel**: AI tool weaponized to breach 600+ FortiGate devices across 55 countries; CrowdStrike reports 89% increase in AI-enabled attacks
- **Healthcare AI $14M breach**: patient records exfiltrated via prompt injection over 3 months (Moltwire); only 17% of enterprises adopted agentic AI
- **Apple bounty evolution**: max payout now $5M+ (base $2M + Lockdown Mode bonus); iPhone 17 Security Research Device with Memory Integrity Enforcement
- **HackerOne hackbot policy** (Feb 2026): mandated "hacker-in-the-loop" for all AI tools; split leaderboard into humans vs AI collectives; HackerOne does not train models on researcher submissions
- **XBOW enterprise pivot**: completed HackerOne mission; GA product now working with large banks and tech firms on pre-production scanning; $75M Series B ($117M total)
- **30 MCP CVEs in 60 days** (Feb-Mar 2026): MCP is AI's fastest-growing attack surface; 38% of 500+ scanned servers lack auth entirely
- **Claude DXT zero-click RCE** (LayerX, CVSS 10.0): 50+ DXT extensions run unsandboxed; Anthropic declined to fix; PromptJacking (Koi AI, CVSS 8.9) found AppleScript injection in 3 official extensions
- **Claude Cowork shipped with known vuln**: file exfiltration via Anthropic API whitelist bypass; disclosed Oct 2025, launched unpatched Jan 2026
- **Cisco SD-WAN CVE-2026-20127** (CVSS 10.0): auth bypass exploited since 2023 by UAT-8616 APT; novel software downgrade chaining technique
- **Microsoft March 2026**: 4 zero-days (kernel UAF, NTFS USB, FAT VHD, MMC .msc); follows February's 6 zero-days — 10 actively exploited zero-days in 2 months
- **Android/Qualcomm CVE-2026-21385**: integer overflow affecting 200+ chipsets; targeted exploitation; CISA KEV
- **D-Link CVE-2026-0625** (CVSS 9.3): DNSChanger zero-day on EoL routers; no patch; exploited since Nov 2025
- **Cisco FMC dual CVSS 10.0** (CVE-2026-20079 + CVE-2026-20131): unauthenticated RCE as root via Java deserialization; affects FMC 6.4.0-10.0.0; no workaround
- **ingress-nginx EOL RCE** (CVE-2026-3288, CVSS 8.8): annotation injection at project end-of-life; 50%+ of K8s clusters affected; no patch at advisory
- **Brave unseeable prompt injection**: hidden near-transparent text passes OCR in AI browsers; hijacks authenticated sessions; novel agentic browser attack class
- **48% of respondents** believe agentic AI will be top attack vector by end of 2026 (DarkReading); only 29% report readiness to secure agentic AI
- **Unit 42 IDPI delivery breakdown**: in-the-wild delivery methods — visible plaintext 37.8%, HTML attribute cloaking 19.8%, CSS rendering suppression 16.9%; autonomous ransomware demo completed in ~25 minutes
- **Smithery.ai path traversal**: fly.io token leaked granting access to 3,000+ MCP server apps; API keys/auth tokens captured from thousands of clients (GitGuardian)
- **Unit 42 2026 Global IR Report**: attacks accelerated 4x YoY; identity weaknesses exploited in 89% of incidents; 87% of attacks involved multiple attack surfaces
