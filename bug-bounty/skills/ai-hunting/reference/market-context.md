# Bug Bounty & AI Security Market Context (2025-2026)

Market statistics, program developments, industry metrics, and competitive intelligence. Reference file for the ai-hunting skill.

> **Related files:** [tools-landscape.md](tools-landscape.md) for AI security tools | [ai-case-studies.md](ai-case-studies.md) for real-world incidents

---

## Table of Contents

- [Market Size & Growth](#market-size--growth)
- [Platform & Program Updates](#platform--program-updates)
- [AI Vulnerability Trends](#ai-vulnerability-trends)
- [Autonomous Agents & Competition](#autonomous-agents--competition)
- [Enterprise AI Security Gap](#enterprise-ai-security-gap)
- [Standards & Regulation](#standards--regulation)
- [Threat Landscape & CVE Trends](#threat-landscape--cve-trends)
- [MCP & Agent Infrastructure](#mcp--agent-infrastructure)
- [Notable Incidents & Disclosures](#notable-incidents--disclosures)

---

## Market Size & Growth

- **Bug bounty market valued at $2.06B** (2026), projected to reach **$7.74B by 2035** (CAGR 15.94%; alternative: $5.7B by 2033)
- **Q4 2025 AI security startup funding**: $2.17B across 28 deals — **8x growth** over two years
- **83% of organizations** now use bug bounties (HackerOne 2025)
- **CISA FY2025 federal bug bounty stats**: 12,800+ vulnerability reports received via VDP Platform, 1,200+ valid (90% remediated); 7 bug bounty programs across 4 agencies; 28 critical vulnerabilities identified, $345K+ awarded; 238 high-risk vulnerabilities added to KEV catalog (CISA 2025 Year in Review)
- **63% of Fortune 500** run bug bounty programs; for every $1 spent on bounties, companies save $15 ($3B in mitigated losses)
- **82% of hackers** now use AI in their workflows (Bugcrowd 2026, up from 64% in 2023)
- **Nearly 10% of researchers** now specialize in AI/LLM security testing
- **60% of large enterprises** using continuous automated red teaming (CART) by 2026; manual pentesting predicted to become boutique service by 2027

---

## Platform & Program Updates

- **HackerOne total annual payouts**: $81M (13% YoY increase), top 10 programs paid $21.6M
- **1,121 programs on HackerOne** now include AI in scope (270% YoY increase)
- **560+ valid reports** submitted by autonomous AI agents on HackerOne
- **HackerOne Hai Triage** adopted by 90% of customers; **Bugcrowd AI Triage Assistant** achieves 98% P1 accuracy
- **HackerOne Hai** AI validation agent launched February 2026 — 56% reduction in vulnerability validation time
- **HackerOne leaderboard split** (August 2025): separated individual researchers vs AI-powered collectives; transparency framework distinguishes human vs machine contributions
- **HackerOne Hai Insight Agent** (March 2026): launched to combat hackbot hallucination — filters false/hallucinated vulnerabilities that appear legitimate because LLMs generate them; positions validation as core platform capability
- **HackerOne AI policy** (Feb 2026): researcher submissions NOT used to train AI models; **Good Faith AI Research Safe Harbor** (Jan 2026). Background: HackerOne's AI service launch triggered researcher concerns about zero-day reports being used as LLM training data — studies show models can memorize and regurgitate sensitive fragments, creating risk if hostile actors gain indirect access. Policy clarification followed community pushback
- **HackerOne Agentic PTaaS** (January 2026): continuous testing with **88% accuracy**, fix-verified findings; trained on proprietary exploit intelligence
- **IBM Granite AI Bug Bounty** (HackerOne): up to **$100K** to test Granite models with Granite Guardian guardrails; focus on jailbreaks in enterprise environments
- **Intigriti adopted CVSS V4** for all new submissions starting 2026
- **Salesforce Bug Bounty 10th Anniversary** — $30.4M invested since 2015; **$6.2M paid in 2025** (program's most productive year), 696 ethical hackers participated, 4,000+ vulnerability reports disclosed, individual payouts up to $60K (Salesforce March 2026)
- **Google awarded $250,000** for a single Chrome flaw (CVE-2025-4609) — record-breaking payout
- **Google launched dedicated AI VRP** (October 2025) covering Search, Gemini Apps, Workspace — up to $30K per finding; $430K+ already paid
- **Apple doubled max payout to $2M** for zero-click remote exploits, with bonuses potentially exceeding $5M; launched "Target Flags"
- **Microsoft Zero Day Quest expanded to $5M** total bounty pool for 2026; paid $1.6M in inaugural 2025 event; **paid $17M** in 2025 to 344 researchers
- **Samsung launched $1M mobile bounty** for critical mobile security architecture flaws
- **OpenAI raised max bounty to $100K**; added Bio Bug Bounty ($25K for universal jailbreaks); **Lockdown Mode** (Feb 2026) acknowledged prompt injection "may never be fully patched"
- **Amazon launched private AI Bug Bounty** for Nova models ($200-$25,000)
- **Nvidia launched bug bounty on Intigriti** (2025): separate private program for core AI assets + public VDP for all other Nvidia properties; covers Nvidia products across hardware and software
- **HackerOne IBB controversy** (Jan 2026): researcher reported two high-severity DoS flaws in Argo CD (CVE-2025-59538, CVE-2025-59531) then was "ghosted" over $8,500 bounty from the Internet Bug Bounty program; highlights trust risks when programs pause/defund without communication — "Bug bounty programs run on trust and clarity"
- **Meta paid $4M** in 2025 ($25M lifetime) across ~13,000 reports
- **Usual crypto bounty**: $16M — largest single bug bounty prize in tech history
- **curl shut down its bug bounty** (January 2026) — first program shutdown attributed to AI slop
- **65% of hackers chose NOT to disclose** a vulnerability because the company lacked a clear reporting pathway (Bugcrowd 2026)
- **Bugcrowd 2026**: 98% of hackers proud of their work; 56% say geopolitics outweighs curiosity; AI-induced job scarcity driving new influx
- **Bugcrowd payout economics**: 100% increase in payouts led to only 20% more total submissions, but a **tripling of critical reports** — higher rewards effectively incentivize researchers to focus on serious vulnerabilities (eWeek 2026)
- **Bugcrowd team collaboration**: 72% believe teams yield better results; **61% find more critical vulnerabilities** when collaborating; 40% currently work in teams, 44% want to but haven't found partners
- **Microsoft expanded bounty to third-party/OSS code** impacting Microsoft services; AI Copilot bounty up to **$30K**
- **Microsoft Zero Day Quest live event**: spring 2026 at Redmond campus (invite-only); flash challenges running through **March 18, 2026**; +50% bounty multiplier for critical-severity findings in Azure, Copilot, Dynamics 365, M365
- **Agentic security gap**: no single tool category (EDR, WAF, SIEM, AI-SPM) provides complete coverage against agentic threats; **48% of cybersecurity professionals** identify agentic AI as the top attack vector heading into 2026 (Dark Reading poll)
- **Vercel React2Shell WAF Bypass program**: **$1M total bounty** — $50K per unique bypass technique; 116 researchers, 156 reports, 20 unique techniques validated; Vercel blocked 6M+ exploit attempts
- **Intigriti hourly payout model**: new feature paying hunters based on hours (like pentesting) in addition to per-vulnerability rewards — hybrid model
- **HackerOne Hai Replay**: dashboard feature turning historical vulnerability data into actionable insights for future testing
- **Microsoft "In Scope by Default"**: all online services now covered including third-party and OSS components; new services in scope from day one; up to $100K for third-party/OSS vulns impacting Microsoft
- **HackerOne top 100 all-time earners**: $31.8M total; top hunters report $300K+ annually
- **XBOW mission completed** (March 2026): declared HackerOne mission concluded; pivoting to pre-production scanning — reduces program competition but also reduces externally-available attack surface for bounty hunters
- **Vibe-coded app attack surface** (2026): Escape.tech found 2,000+ vulns in 5,600 vibe-coded apps (Lovable, Bolt.new, Replit); Moltbook breach exposed 1.5M API keys via misconfigured Supabase; 10% of Lovable apps leaked PII — new class of systematically vulnerable targets for bounty programs
- **Wiz Cyber Model Arena** (March 2026): 257 offensive security challenges, 25 agent-model combinations tested; Claude Code + Opus 4.6 #1; AI solved 9/10 directed challenges at under $50 but fails at broad enumeration; human direction + AI execution confirmed as winning model
- **DJI ROMO bug bounty** (February 2026): $30K awarded for BOLA + MQTT wildcard subscription flaw exposing 7,000+ robot vacuums across 20+ countries; live camera feeds, audio streams, and 2D floor plans accessible; researcher originally building PS5 controller interface; DJI patched Feb 8-10 (DroneDJ, March 2026)
- **Google AI VRP**: dedicated AI Vulnerability Reward Program covering Search, Gemini Apps, Workspace — up to $30K per finding; moved AI issues from Abuse VRP
- **XBOW $75M raised** (March 2026): Series B led by Altimeter Capital with Sequoia; $117M total funding; but still operating at a loss (compute costs exceed bounty earnings)

---

## AI Vulnerability Trends

- **HackerOne valid issues**: 78,042 valid vulnerabilities found (+12% YoY); XSS still #1 at 18% of reports but **down 10% since 2023** — declining payouts
- **IDOR/IAC surge**: valid IDOR reports grew **116% over 5 years** and jumped **29% YoY**; **IAC awards up 134% YoY to $4M+**; improper access control up **18% YoY**; IDOR is #1 vuln class for government (18%), medtech (36%), and professional services (31%) programs
- **70% of researchers** now combine human creativity with AI tools ("bionic hackers") for report writing, PoC generation, exploit code refinement, and technology research (HackerOne 2025)
- **210% increase** in AI vulnerability reports; **540% jump** in prompt injection reports
- **Top hunter composition** (2025): ~40% AI bugs, remainder split across IDOR, CSRF, information disclosure, and business logic — AI is now the plurality finding class for elite researchers
- **339% increase** in bounties for AI vulnerabilities YoY
- **Prompt injection in 73%+ of production AI deployments** assessed; only 34.7% have dedicated defenses
- **AI agent attack success rates 66-84%** when testing against auto-execution systems
- **Over 21,500 CVEs** disclosed in H1 2025 alone — 16-18% increase over 2024; CVE volume continues accelerating
- **Average HackerOne pentest** uncovers 12 vulnerabilities, with 16% classified as high/critical; paired with bug bounty, 25% of reports are high/critical
- **Bugcrowd CrowdMatch**: ~32% market mindshare as of January 2026; AI-powered researcher matching technology
- **Supply chain dominance**: Group-IB 2026 reports 68% of severe incidents linked to supply chain compromise — nearly double from prior years
- **AI agent skill supply chain**: Snyk ToxicSkills audit found 100% of malicious skills use dual-vector attacks (code + prompt injection); mcp-scan detection achieves 90-100% recall with 0% false positives
- **CAI 3,600x**: Cybersecurity AI framework demonstrated 3,600x performance improvement over human pentesters in standardized CTF benchmarks (aliasrobotics)
- **Multi-turn prompt injection** achieves up to **92% success rates** across 8 open-weight models
- **RAG poisoning threshold**: just 5 carefully crafted documents manipulate AI responses 90% of the time; indirect prompt injection targets all data AI ingests (webpages, PDFs, MCP metadata, RAG docs, emails, memory, code)
- **Prompt injection attack evolution**: persistence capabilities in 12 of 21 documented multi-stage attacks (2025-2026); lateral movement grew from 0 incidents in 2023 to 8 of 21; attacks escalating from single-shot to multi-stage campaigns
- **35% of real-world AI security incidents** caused by simple prompts — advanced attacks not necessary
- **Only 10% of AI-generated code is secure** (Endor Labs study, March 2026)
- **Apiiro "4x Velocity, 10x Vulnerabilities"** — AI coding assistants produce 10x more security findings; 322% increase in privilege escalation bugs
- **97% of AI-related incidents** stemmed from basic access control flaws (HackerOne 2025)
- **Critical CVEs in AI coding tools**: GitHub Copilot RCE (CVE-2025-53773, CVSS 9.6), Cursor IDE (CVSS 9.8), Microsoft Copilot (CVSS 9.3)
- **Joseph Thacker prediction**: 2x bug submissions in 2026 driven by AI coding agents
- **Prompt injection 84% success rate** in agentic systems with auto-execution; production exploits now carry CVSS 9.0+ scores (OWASP Agentic Top 10 data)
- **CVE-2026-22807 (vLLM)**: AI inference engine loads Hugging Face `auto_map` modules without `trust_remote_code` gating → attacker-controlled Python executes at server startup. Pattern: model metadata as code execution vector
- **VDP as compliance baseline** (March 2026): NIST CSF 2.0, ISO 27001, and EU CRA all explicitly reference coordinated vulnerability disclosure; not having a VDP is "like not having a privacy policy in 2018"
- **EU Cyber Resilience Act enforcement**: **September 11, 2026** — mandatory vulnerability reporting and incident notification for all products with digital elements sold in EU; manufacturers must report actively exploited vulnerabilities to ENISA within 24 hours; significant expansion of VDP-adjacent obligations
- **Bugcrowd AI agent false positives**: researchers building multi-agent bounty systems report agents flagging "information disclosure" because files contain word "password" in explanations, not as actual sensitive data; "fully unattended" claims overstated

---

## Autonomous Agents & Competition

- **XBOW** reached **#1 on HackerOne US leaderboard** in 90 days, submitting ~1,060 vulns (54 critical, 242 high, 524 medium); cumulative 1,400+ findings; **$75M Series B** ($117M total, Altimeter + Sequoia)
- **XBOW Public API** launched Feb 2026 ($6K per pentest, 5-day turnaround)
- **78% local XBOW benchmark**: fully local agent via feedback-driven iteration — cloud infrastructure no longer required
- **AISLE discovered all 12 OpenSSL zero-days** in January 2026 (15 total across both releases, credited for 13 of 14 CVEs in 2025); including CVE-2025-15467 (stack buffer overflow, CVSS 9.8 CRITICAL RCE); three bugs dated back to 1998-2000 (25-27 years), one inherited from SSLeay; **100+ CVEs across 30+ projects** (Linux kernel, glibc, Chromium, Firefox, WebKit, Apache HTTPd, GnuTLS, OpenVPN, Samba, NASA CryptoLib)
- **Claude Opus 4.6 found 500+ vulnerabilities** across production OSS codebases; **22 Firefox vulns** in 2 weeks
- **OpenAI Codex Security** (March 6, 2026): 1.2M commits scanned, 14 CVEs found; $10M in API credits for OSS
- **Novee Security benchmark**: proprietary model outperformed Gemini 2.5 Pro and Claude 4 Sonnet by 55%+, achieving up to 90% accuracy
- **Shannon** (Keygraph): open-source autonomous pentester achieving **96.15%** on hint-free XBOW benchmark (100/104 exploits); source-code-aware testing
- **Terra Security** ($30M Series A): agentic AI continuous PTaaS with human-in-the-loop; hybrid AI + human supervision model validated by Felicis
- **March 2026 escalation**: Both Anthropic and OpenAI launched enterprise scanning the same week (March 6)
- **GPT-5.4 native computer use** (March 5, 2026): first general model operating desktops natively — 75% OSWorld (surpassing human 72.4%); 1M token context; with OpenClaw enables autonomous GUI-navigating security testers

---

## Enterprise AI Security Gap

- **83% plan agentic AI** deployment, only **29% ready** to secure it (Cisco 2026)
- **Enterprise AI readiness gap**: only 34% have AI-specific security controls; less than 40% test AI regularly
- **3+ million AI agents** in corporations; **88% reported security incidents**; **47% not monitored** (Gravitee 2026)
- **Enterprise AI failure costs**: 64% of $1B+ companies lost more than $1M to AI failures (Help Net Security 2026)
- **Shadow AI breaches** cost **$670,000 more** than standard incidents; 1 in 5 orgs reported breaches linked to unauthorized AI. Bugcrowd 2026 prediction: shadow AI is the new shadow IT — employees creating unauthorized AI agents with privileged access will drive major IP compromise incidents. **Hunter angle:** test whether target's AI features expose unsanctioned agent endpoints or unmonitored tool integrations
- **97% of organizations** with AI breaches lacked basic access controls; 63% pasted sensitive data into chatbots
- **Non-human identities** (AI agents, service accounts) represent **higher risk (52%)** than human users (37%) in cloud (Tenable 2026)
- **48% of cybersecurity professionals** identify agentic AI as the #1 attack vector heading into 2026 (Dark Reading)
- **Gartner** (2026): **40% of enterprise apps** will embed task-specific AI agents by end of 2026 (up from <5% in 2025)
- **Cycode 2026**: **100% of surveyed organizations** have AI-generated code in their codebases; **81% lack visibility** into AI usage across the SDLC; **62% of code from latest LLMs** has at least one exploitable vulnerability; 97% using or piloting AI coding assistants
- **Aikido 2026**: **1 in 5 organizations** experienced a serious security incident from AI-generated code
- **PwC 2025**: 79% of companies have deployed agentic AI, but security lags adoption
- **Tenable Cloud & AI Risk Report**: 70% of organizations integrated AI/MCP packages without centralized security oversight; 18% granted AI services admin permissions

---

## Standards & Regulation

- **EU AI Act deadline**: **August 2, 2026** — penalties up to 35M EUR or 7% of global turnover
- **OWASP Agentic Top 10** (December 2025): ASI01-ASI10, developed by 100+ experts
- **OWASP AIVSS** (AI Vulnerability Scoring System): extends CVSS for agentic capabilities; v1.0 targeted RSA 2026
- **OWASP AI Testing Guide v1** (November 2025): first comprehensive standard for AI Trustworthiness testing
- **OWASP MCP Secure Development Guide** published — actionable checklist for MCP auditors
- **OWASP GenAI Security Summit at RSAC 2026** — March 23-26, featuring Agentic Security Hackathon
- **OWASP AIVSS AIUC-1 integration** announced February 27, 2026
- **CoSAI MCP Security Whitepaper** (January 2026): 12 core threat categories, ~40 distinct threats
- **NIST AI RMF and ISO 42001** do not yet address agentic deployment controls
- **NIST CAISI AI Agent Standards Initiative** (February 2026): RFI due March 9, concept paper due April 2
- **OpenSSF AI/ML Security Working Group** (February 2026): bi-weekly with CoSAI, AGNTCY, NIST, SPDX, OWASP
- **Bright Security 2026 State of LLM Security**: all context untrusted by default; tool-enabled LLMs = highest risk
- **HackTheBox AI Red Teamer Certification** launching Q1 2026 — developed with Google/SAIF
- **OWASP FinBot CTF** reference application for practicing agentic security skills
- **Promptware Kill Chain** (arxiv:2601.09625): 7-stage model; 21 of 36 attacks traverse 4+ stages
- **Adversa AI MCP Security TOP 25** — definitive catalog with red team guides and defensive playbooks
- **CoSAI MCP Security White Paper** (January 2026, OASIS): 12 core threat categories, ~40 distinct threats — THE reference taxonomy alongside OWASP MCP Top 10; mapped to recon signals and test procedures in mcp-playbooks.md
- **AI-driven DAST revolution** (2026): auto-discover endpoints from source code reduces setup from weeks to hours; traditional DAST vendors integrating agentic AI for continuous testing — competitive pressure on manual assessment workflows

---

## Threat Landscape & CVE Trends

- **21,500+ CVEs** disclosed in H1 2026 alone — 16-18% increase over 2024
- **28.96% of vulnerabilities** now exploited on or before CVE disclosure day (VulnCheck State of Exploitation 2026; up from 23.6% in 2024)
- **IBM X-Force 2026**: vulnerability exploitation now **#1 initial attack vector at 40%** (up from 26% prior year); validates that exploitation speed is outpacing patch cycles — hunters should prioritize recently-patched CVEs in programs where patch lag is visible
- **884 KEVs** identified in 2025; exploitation was #1 cause of incidents at 40% (IBM X-Force)
- **FIRST CVE forecast** (Feb 2026): predicted median 59,427 new CVEs (first year to exceed 50,000)
- **AI-enabled attacks** surged 89% YoY; average eCrime breakout time now 29 minutes, fastest 27 seconds (CrowdStrike 2026)
- **90 zero-day vulnerabilities** tracked in 2025; 48% targeted enterprise technology (Google review)
- **API security detection gap**: only 21% detect attacks at API layer; 97% of API vulns exploitable with single request (42Crunch 2026)
- **Malicious AI browser extensions**: ~900,000 installs across 20,000+ enterprise environments (Microsoft Defender, March 2026)
- **GreyNoise LLM infrastructure targeting** (Oct 2025-Jan 2026): 91,403 attack sessions captured; two campaigns — (1) SSRF exploitation via Ollama model pull + Twilio webhooks with Christmas spike (1,688 sessions in 48 hours), 99% sharing single JA4H signature (Nuclei tooling), 62 IPs across 27 countries; (2) systematic enumeration of 73+ LLM model endpoints from just 2 IPs over 11 days (80,469 sessions). Assessment: if running exposed LLM endpoints, you're already on target lists
- **CrowdStrike 2026**: FANCY BEAR, PUNK SPIDER, FAMOUS CHOLLIMA named as AI-using threat actors; cloud intrusions up 37%
- **IBM X-Force 2026**: 44% increase in attacks via public-facing apps; 300,000+ ChatGPT credentials on dark web
- **Cisco broke DeepSeek R1** with 100% jailbreak success rate (50/50 prompts)
- **CyberStrikeAI attacks**: AI offensive tool deployed across 55 countries against FortiGate firewalls (Jan-Feb 2026)

---

## MCP & Agent Infrastructure

- **40+ MCP CVEs** filed in Q1 2026; 38% of servers lack auth; 43% vulnerable to command execution (Adversa AI); **1,412 MCP servers indexed** (232% increase in 6 months, Adversa AI March 2026 digest). **Field scan** (DEV Community/Kai Security, March 2026): 539 active production endpoints scanned — 201 (37.4%) require no authentication; 70% of all MCP protocol traffic is `initialize` + `tools/list` + `disconnect` (pure reconnaissance, not exploitation) — attackers are systematically mapping the ecosystem before attacking
- **8,000+ MCP servers** found publicly exposed (Feb 2026)
- **Endor Labs analysis** of 2,614 MCP implementations: 82% Path Traversal, 67% Code Injection, 34% Command Injection risks
- **MCP server audit** (March 2026): 118 findings across 68 of 194 packages audited
- **MCP tool name collision** (CVE-2026-30856): namespace hijacking in MCP clients
- **MCP OAuth account takeover**: CSRF-style attacks on Remote MCP servers (Obsidian Security); spec updated to mandate OAuth 2.1 + PKCE
- **OpenClaw supply chain crisis**: 1,184+ malicious skills, 13+ CVEs, 42,665 exposed instances; most extensively exploited AI agent platform
- **ContextCrush (Context7)**: supply chain via MCP server (~50K GitHub stars); patched Feb 23, 2026
- **Schema Drift + Context Pivoting + Full Schema Poisoning**: three new MCP attack vectors discovered March 2026
- **ICON defense** (March 2026): two-stage indirect injection defense — emerging defense to test against
- **AgentAudit MCP registry**: 270+ packages audited, 247 vulnerabilities found; Schema Drift discovery
- **n8n multi-CVE** (Feb-Mar 2026): 7+ CVEs including CVSS 10.0, 9.9, 9.4 — workflow platforms are goldmines
- **Pynt MCP exploit probability** (March 2026): 10 MCP plugins = **92% exploit probability**; 1 plugin = 9%, 2 = 36%, 3 = 50%+; 72% of MCPs expose sensitive capabilities (code execution, file system, privileged APIs); 13% accept untrusted inputs (web scraping, Slack, email, RSS); 9% intersection = direct attack paths; 281 configurations analyzed (VentureBeat/Pynt)
- **Enkrypt AI 1,000 MCP server scan** (March 2026): **33% had critical vulnerabilities**, averaged 5.2 vulns each; K8s MCP server: 26 vulns including 6 CVSS 9.8; malicious Postmark MCP server silently exfiltrated every email it processed
- **ChatGPT MCP integration** (March 2026): ChatGPT supports MCP apps/connectors via developer mode (beta); admin-approved remote MCP servers can trigger tools; malicious MCP servers targeting Enterprise/Business users = emerging threat vector
- **Knostic exposed MCP servers**: 1,862 MCP servers found with zero authentication; every server tested responded without credentials
- **Noma x OWASP Red Team Playbook** (March 2026): comprehensive agentic AI red team playbook covering prompt injection, tool misuse, excessive agency, multi-agent coordination failures; continuously updated attack library from Noma Labs

---

## Notable Incidents & Disclosures

- **EchoLeak (CVE-2025-32711, CVSS 9.3)**: first real-world zero-click prompt injection in production (MS 365 Copilot)
- **React2Shell (CVE-2025-55182, CVSS 10.0)**: pre-auth RCE in React Server Components; #1 on HackerOne
- **Microsoft Entra ID (CVE-2025-55241, CVSS 10.0)**: Global Administrator via Actor Tokens
- **Cisco FMC CVSS 10.0 pair** (March 4, 2026): CVE-2026-20079 + CVE-2026-20131; no workarounds
- **VMware Aria Operations** (CVE-2026-22719, CVSS 8.1): actively exploited; CISA KEV March 3
- **Qualcomm zero-day** (CVE-2026-21385): 230+ chipsets; nation-state exploitation
- **VoidLink AI-generated malware** (March 2026): 88,000+ lines of AI-written advanced malware
- **AI as C2 Proxy** (Check Point): AI browsing features weaponized as bidirectional C2 channels
- **IDEsaster campaign**: 30+ vulnerabilities, 24 CVEs across AI coding tools
- **Shai-Hulud supply chain worm**: 454,648 malicious npm packages in 2025
- **GTG-1002**: first state-sponsored espionage primarily orchestrated by AI agent
- **Claude Code Hooks RCE** (CVE-2025-59536/CVE-2026-21852): project file exploitation pattern
- **Firefox JIT exploit** (CVE-2026-2796, CVSS 9.8): first AI-authored browser engine exploit
- **AI app data breach epidemic**: 20+ incidents; Chat & Ask AI exposed 406M records
- **LLM-assisted deanonymization** (arXiv:2602.16800): 25-67% recall at $1-4 per identification
- **Google Antigravity IDE**: $10K bounty for RCE; 70+ architectural vulnerabilities; Forced Descent unpatched
- **Clawdbot/Moltbot** (Jan 2026): 2,000+ gateways on Shodan within 72 hours
- **New MCP CVEs**: gemini-mcp-tool (CVSS 9.8), mcp-atlassian RCE, mcp-nmap injection, eBay API MCP (unpatched)
- **Check Point Claude Code Hooks**: hooks-based auto-RCE + API key exfiltration via `ANTHROPIC_BASE_URL`
- **Memory injection / sleeper agents** (Lakera AI): persistent false beliefs via poisoned agent memory
- **Azure MCP Server RCE chain** (Token Security, RSAC 2026): first demonstrated RCE against a major cloud vendor's official MCP server
- **Kubernetes ingress-nginx retirement** (March 2026): CVE-2026-24512 (CVSS 8.8) + 3 more CVEs in final patch; project retiring — no future patches for 50% of K8s clusters
- **Cisco SD-WAN active exploitation** (March 5, 2026): CVE-2026-20128/20122; file overwrite + privilege escalation; web shell activity at scale
- **NGINX SSL upstream injection** (CVE-2026-1642): TLS race condition in NGINX 1.3.0-1.29.4 allowing MITM response injection before handshake completes
- **Noma Security MCP blindspots**: >90% of orgs maintain dangerous default configs with all MCP tools enabled
- **Business logic vulnerabilities** account for **45% of all bounty awards** industrywide (Intigriti 2026) — largest single category
- **Chrome emergency update** (March 3, 2026): 10 CVEs fixed including 3 Critical — CVE-2026-3537 (PowerVR object lifecycle, $32K bounty), CVE-2026-3538 (Skia integer overflow), CVE-2026-3545 (sandbox escape, CVSS 9.6)
- **CVE-2026-21385** (Qualcomm Android zero-day, CVSS 7.8): exploited in wild, 234 chipsets, CISA KEV March 3, 2026
- **Cisco SD-WAN CVE-2026-20127** (CVSS 10.0): auth bypass exploited since 2023; CISA mandated 24-hour patch
- **OpenClaw inbox destruction incident** (Feb 23, 2026): Meta AI safety director lost 200+ emails after agent ignored STOP commands; context window compaction caused safety instruction loss
- **Gartner** (Feb 2026): 62% of large enterprises piloting AI agents, only 14% have formal governance frameworks
- **Aikido Infinite** (Feb 26, 2026): first continuous AI pentesting on every code change; found 7 CVEs in Coolify (RCE as root, 52K+ exposed instances)
- **CVE-2026-27203** (eBay MCP Server env injection RCE): unpatched — `updateEnvFile` allows arbitrary env var injection via newline characters
- **CVE-2026-29791** (Agentgateway MCP-to-OpenAPI): missing parameter sanitization in path/query/header values; fixed in 0.12.0
- **eBay MCP pattern**: demonstrates that even commerce-vendor MCP servers ship with trivially exploitable injection — environment file manipulation is a recurring vulnerability class
- **February 2026 Patch Tuesday**: 6 actively exploited zero-days — unprecedented count; Secure Boot cert expiration (June 2026) adds urgency
- **CVE-2026-3680** (biome-mcp-server, March 7 2026): command injection via `child_process` in CLI-wrapping MCP server; CVSS 5.3; unauthenticated remote; pattern repeats across MCP servers that wrap CLI tools
- **XBOW #1 globally** (2025-2026): topped both US and global HackerOne leaderboards; pivoting to pre-production scanning for enterprise customers
- **Cloudflare public bug bounty** launched: high-value target covering CDN, DNS, Workers, R2, zero trust
- **NEAR Intents bridge** bug bounty (March 3, 2026): cross-chain MPC infrastructure; bridges historically yield top payouts
- **Microsoft "In Scope by Default"** expanded to include third-party and open source code impacting Microsoft online services
- **Bugcrowd FedRAMP Moderate Authorization** (March 2026): sponsored by CISA; first crowdsourced security platform cleared for U.S. federal agencies; opens government programs to researchers
- **CISA VDP Platform FY2025**: federal agencies received **12,800 vulnerability reports** from public researchers, including 1,200+ valid reports (90% remediated); CISA managed **7 bug bounty programs** across 4 agencies, identifying 28 critical vulnerabilities and awarding **$345K+**; 238 high-risk vulnerabilities added to KEV catalog in FY2025 (CISA 2025 Year in Review)
- **HackerOne AI Research Safe Harbor** (January 2026): platform-wide opt-in specifically for AI system testing; reduces legal risk for AI security researchers; signals programs welcoming AI-focused submissions
- **Intigriti Global Hacker Ambassador Program** (March 4, 2026): early access to private programs, rewards, community building; 125,000+ ethical hackers in the community
- **NVIDIA private AI bounty on Intigriti**: private program specifically covering NVIDIA AI products and infrastructure; public VDP for all NVIDIA assets
- **Zscaler 100% enterprise AI failure rate**: median 16 minutes to first critical failure; 90% compromised in under 90 minutes; validates universal enterprise AI vulnerability
- **80% of organizations** reported risky AI agent behaviors (unauthorized system access, improper data exposure); only 21% of executives have complete visibility into agent permissions; 97% of AI security incidents lacked proper access controls (Gravitee March 2026)
- **Cisco State of AI Security 2026**: AI vulnerabilities from research labs now materializing in production incidents; agentic AI risk and AI supply chain as major themes
- **Joseph Thacker (rez0) critical window prediction**: 2026 is "massively important" — 2x bug submissions predicted; 1-2 years before internal AI code review commoditizes external bug bounty; Anthropic's Firefox exploitation cost $4K in API credits vs $3K-$20K typical bounty payouts
- **curl ended bug bounty** (January 2026): first program shutdown attributed to AI slop — submission volume spiked 8x, only 5% genuine; zero AI-only submissions found real vulns in 6 years
- **Wallarm 2026 API ThreatStats**: AI-related API threats surged 400% YoY (439 to 2,185 disclosed vulns); 36% of AI vulns also qualify as API vulns; MCP predicted as central 2026 attack vector
- **OpenAI Codex Security Research Preview** (March 6, 2026): context-aware vulnerability scanner with project-specific threat models and sandboxed validation; 1.2M commits scanned, 792 critical + 10,561 high findings; 50%+ FP reduction, 90% severity over-reporting reduction; free first month for Enterprise/Business/Edu; $10M API credits for OSS — major competitive pressure on human hunters for pattern-matching vulns
- **Microsoft "AI as Tradecraft"** (March 6, 2026): first documentation of state-sponsored agentic AI experimentation; Coral Sleet fully AI-enabled malware workflow (lure dev → infrastructure → payload testing); LLM jailbreaking for malicious code generation; AI code fingerprints (emojis as markers, conversational comments)
- **Checkmarx 11 MCP Risks** (March 2026): comprehensive taxonomy covering prompt injection, tool poisoning, confused deputy, supply chain, malicious servers, cross-agent context abuse, privilege escalation, insecure deserialization, context leakage, insecure clients, dependency confusion
- **OWASP Agentic AI Top 10** (finalized 2026): ASI01-ASI10 covering agent goal hijacking, tool misuse, privilege escalation, supply chain, excessive agency, memory poisoning, inter-agent comms, cascading failures, trust exploitation, rogue agents; input from 100+ security researchers; BleepingComputer published real-world examples mapping to each risk
- **CVE-2026-23864** (React Server Components DoS, CVSS 7.5): specially crafted HTTP requests to Server Function endpoints cause server crashes, OOM, or excessive CPU; affects Next.js and React metaframeworks (January 2026)
- **RondoDox botnet** exploiting React2Shell (CVE-2025-55182) to hijack IoT devices and web servers — demonstrates botnet adoption of web framework RCE
- **Apple Security Research Device 2026**: iPhone 17 devices with Memory Integrity Enforcement; expanded bounty categories; "Target Flags" for objective exploitability demonstration
- **Malicious Postmark MCP server** (Enkrypt AI): discovered during 1,000-server scan — silently exfiltrated every email it processed; demonstrates supply chain risk in MCP ecosystem
- **PentAGI multi-agent pentesting** (2026): fully autonomous system with researcher/developer/executor roles; Graphiti knowledge graph (Neo4j); sandboxed Docker; supports OpenAI, Claude, Gemini, local models; open-source
- **FastMCP 1M+ daily downloads** — powers ~70% of all MCP servers as the dominant framework; MCP SDK reaches **97 million monthly downloads** across 10,000+ active servers adopted by OpenAI, Microsoft, Google
- **OWASP Red Teaming Vendor Evaluation v1.0** (March 6, 2026): first standard for evaluating AI red teaming providers; distinguishes chatbot-level from agent-level testing; warns against vendors treating MCP/multi-agent architectures as simple chatbots
- **100% AI IDE prompt injection rate** (March 2026): all tested AI IDEs (Copilot, Cursor, Windsurf, Claude Code) vulnerable to prompt injection enabling RCE; 24 CVEs, 1.8M developers at risk; novel vulnerability class
- **OX Security 94 Chromium CVEs**: Cursor and Windsurf ship legacy Chromium builds — 94+ known vulnerabilities exploitable against 1.8M developers; CVE-2025-7656 weaponized against latest versions
- **Agents of Chaos (arXiv:2602.20021)**: first autonomous agent red-team study — 11 failure modes over 2 weeks including synonym-based PII bypass, self-destructive actions, 9-day infinite loops, false completion reporting
- **Aikido Infinite new CVEs** (March 2026): autonomous agents found CVE-2026-25545 (Astro SSRF), CVE-2026-27148 (Storybook WebSocket hijack → supply chain), in addition to 7 Coolify CVEs
- **Keysight MCP confused deputy** (January 2026): documented MCP command injection taxonomy — STDIO transport servers run with user privileges, creating confused deputy scenario; detection signatures added to ATI-2025-25 StrikePack
- **Contrast Security MCP Server**: defensive MCP server bridging IAST vulnerability data with AI agents for context-aware remediation; provides complete taint-flow trace from source to sink
- **OpenAnt (Knostic, March 6, 2026)**: open-source LLM vulnerability scanner with two-stage detect + exploit pipeline; supports 6 languages; Apache 2.0
- **Quantro Security VM.Analyst** (March 10, 2026): autonomous AI agents for vulnerability management; founded by ex-CrowdStrike/Tenable/Qualys engineers; backed by Gradient (Google AI fund); normalizes data across VM platforms, CMDBs, and cloud security suites; assesses risk in context of compensating controls
- **Claude Code macOS keychain fix** (March 2026): keychain corruption when using multiple OAuth MCP servers; large OAuth metadata overflows security buffer leaving stale credentials; trust dialog bug silently enabled all `.mcp.json` servers on first run
- **March 2026 Patch Tuesday preview**: 2 confirmed zero-days (Windows SmartScreen bypass, Outlook flaw); follows February's unprecedented 6 actively exploited zero-days
- **Krebs on Security OpenClaw exposure** (March 8, 2026): OpenClaw web admin interfaces exposed to internet with full credential configurations (API keys, OAuth secrets, signing keys); Meta AI safety director lost 200+ emails after agent ignored STOP commands
- **Apple CVE-2026-20700** (iOS dyld zero-day, CVSS 7.8): "extremely sophisticated attack" against specific individuals; discovered by Google Threat Intelligence; chained with WebKit CVEs; patched iOS 26.3
- **MSHTML CVE-2026-21513** (CVSS 8.8, APT28): MotW bypass via `ieframe.dll` URL validation → code execution; part of February 2026's 6 zero-day Patch Tuesday; 3 separate MotW bypasses indicate hot vulnerability class
- **Unit 42 MCP Sampling attacks** (March 2026): 3 new attack vectors — resource theft, conversation hijacking, covert tool invocation via MCP sampling mechanism; servers become active prompt authors
- **WeKnora 5-CVE multi-tenant compromise** (March 7, 2026): CVE-2026-30855 (CVSS 8.8) broken access control + CVE-2026-30858 (DNS rebinding SSRF) + CVE-2026-30857 (cross-tenant KB cloning); open registration → unauthenticated full ATO; LLM API key leakage from tenant configs. Pattern: multi-tenant AI platforms with open registration are goldmines
- **RustDesk 7+ CVE cluster** (March 5, 2026): session replay, missing auth, MitM, cleartext, broken crypto, pre-auth SSRF; popular self-hosted remote desktop (Client ≤1.4.5, Server ≤1.7.5); many instances on Shodan
- **Trail of Bits Claude Code security skills** (March 2026): battle-tested audit methodology from Testing Handbook as Claude Code plugin; `claude-code-config` for opinionated sandboxing; professional audit firm entering plugin ecosystem
- **AuthZed MCP breach timeline** (March 2026): systematic breach analysis — WhatsApp, GitHub, Asana MCP servers; root cause across all: authorization failures (over-privileged tokens, missing access controls)
- **CyberStrikeAI weaponization** (March 2026): open-source AI tool used to breach 600+ FortiGate devices across 55 countries; 21 unique IPs observed; CrowdStrike: 89% increase in AI-enabled adversary attacks YoY
- **Healthcare AI agent $14M breach** (Moltwire, March 2026): patient records exfiltrated via prompt injection over 3 months; only 17% of enterprises have adopted agentic AI; ACM identified 4 critical knowledge gaps
- **Dell RecoverPoint CVE-2026-22769 (CVSS 10.0)**: hardcoded credentials exploited by China-nexus UNC6201 since mid-2024; pattern: enterprise backup appliance hardcoded creds
- **HackerOne Report #3000510**: $25K bounty for HackerOne's own `.json` endpoint leaking reporter emails, OTP codes, phone numbers, graphql_secret_token; fixed in 2 days
- **HackerOne $81M annual payouts** (March 2026): $81M paid in bounties over past 12 months (+13% YoY); top 10 programs accounted for $21.6M; individual researchers consistently surpassing six-figure annual earnings
- **AI vulnerability reports surged 540%**: prompt injection flaws driving 200%+ overall increase in AI security issues; authorization-related vulnerabilities (IDOR, improper access control) on the upswing as XSS/SQLi decline
- **XBOW operating economics**: despite #1 HackerOne ranking with 1,060+ reports, XBOW is operating at a loss — compute costs exceed bounty earnings; HackerOne split leaderboards to separate individuals from automated systems; demonstrates autonomous hunting is not yet profitable at scale
- **Cycode 62% exploitable LLM code** (March 2026): study found 62% of LLM-generated code contains exploitable vulnerabilities — validates ongoing demand for code security review as AI-generated codebases proliferate
- **RoguePilot AI-mediated supply chain** (February 2026): first documented AI-assisted repository takeover via passive prompt injection in GitHub issues; symlink + $schema exfiltration chain; patched by Microsoft; demonstrates IDE-integrated AI as attack surface
- **CVE-2026-29783 Copilot CLI RCE**: bash parameter expansion bypass in safety assessment — prompt injection via repo files can make "read-only" classified commands execute arbitrary code; patched in 0.0.423
- **CVE-2026-27127 Craft CMS DNS rebinding** (TOCTOU): SSRF validation bypass via DNS rebinding in GraphQL Asset mutation; separate DNS resolution from HTTP request creates race condition window; affects all versions 3.5.0-4.16.19 and 5.0.0-RC1-5.8.23
- **Cursor Automations** (March 2026): Cursor launched agentic automation triggered by codebase changes, Slack messages, or timers — enables automated security audits; AI IDEs shifting from reactive assistants to proactive autonomous agents
- **Teramind AI Governance** (March 2026): behavioral oversight platform for AI agents with full activity capture; first enterprise solution for AI agent audit trails and behavioral monitoring at scale
- **ArmorCode $81M AI AppSec** (March 2026): raised $16M (total $81M) to expand Agentic AI Platform for application security management; growing AI-native AppSec vendor category
- **FIRST CVE Forecast 2026**: median ~59,427 CVEs predicted (90% CI: 30,012–117,673); first year to exceed 50,000; realistic upper-bound scenarios reach 70,000–100,000; 2027 forecast: ~51,018; 2028: ~53,289
- **PleaseFix agentic browser hijack** (Zenity Labs, March 2026): zero-click agent hijack family — calendar invite triggers autonomous execution, exfiltrates local files and 1Password credentials; applicable to all agentic browsers (Perplexity Comet, ChatGPT Atlas, Chrome Gemini)
- **OpenAI Lockdown Mode** (February 2026): acknowledged prompt injection in AI browsers "may never be fully patched"; adaptive attacks bypass 12 recent defenses with 90%+ success using gradient descent, RL, random search
- **Chrome CVE-2026-3545** (CVSS 9.6): sandbox escape via insufficient data validation in Navigation component; high-value variant analysis target
- **Chrome CVE-2026-2441**: CSS zero-day (@property + paint() worklet use-after-free) actively exploited; patched February 13
- **Shannon Lite 96.15%**: hit 100/104 on hint-free, source-aware XBOW benchmark variant using Code Property Graphs; subscription model ($0–$49.99/mo)
- **Anthropic + Mozilla partnership** (March 6, 2026): Claude Opus 4.6 found 22 Firefox CVEs (14 high-severity) in 2 weeks scanning ~6,000 C++ files; 112 bug reports filed; fixes shipped in Firefox 148; PoC exploit generation cost ~$4,000 in API credits, succeeded in only 2 cases
- **OpenAI acquires Promptfoo** (March 9, 2026): open-source LLM red-teaming CLI (125K+ devs, 30+ Fortune 500, 50+ vuln types); pledged to remain open-source; signals AI red-teaming consolidation under frontier labs; 97.14% autonomous jailbreak success rate reported separately (Nature Communications)
- **VulnCheck 2026 VEIR**: 14,400+ exploits tracked for 10,000+ unique CVE-2025 vulns (16.5% YoY increase); AI-generated fake PoCs on the rise — validate any public exploit code before using
- **Bionic hacker dominance**: 67% of HackerOne researchers use AI tools; hackbots submitted 560 valid reports (primarily surface-level XSS); human + AI collaboration is the winning model
- **Barracuda agentic AI threat study**: identified 43 agent framework components with embedded supply chain vulnerabilities; AI offense compresses attack lifecycle — recon, exploitation, lateral movement run continuously in parallel
- **Blackbox AI VS Code RCE** (ERNW, March 2026): indirect prompt injection in Blackbox AI extension (30M+ users, 4.8M VS Code installs); malicious instructions in PNG files → OCR → code execution; vendor unresponsive 2+ months — image-based prompt injection weaponized against AI coding tools
- **CoSAI MCP Security White Paper** (January 2026): taxonomy of ~40 threats across 12 categories with concrete mitigations; key: agent identity traceability, least privilege, MCP server sandboxing, TEE-based credential protection; MCP spec updated with OAuth Resource Server roles and RFC 8707 Resource Indicators
- **Endor Labs AURI** (March 3, 2026): AI-native security intelligence for agentic development; addresses "vibe coding" security risks; free developer tier (MCP server, CLI, Skills plugin); found 7 OpenClaw vulns
- **Wallarm API ThreatStats 2026**: 11,053 API-related CVEs (17% of all 67,058 published); **97% exploitable in single request, 59% need no authentication**; 43% of CISA KEV additions were API-related; cross-site issues now top attack volume
- **Cloudflare 2026 Threat Report**: bots make 94% of login attempts; of human logins, **46% use already-compromised credentials**; 230 billion threats blocked daily; DDoS doubled to 47.1M attacks; attackers shifting from "breaking in" to "logging in"
- **Bugcrowd AI adoption 2026**: 82% of hackers now use AI (up from 64% in 2023); human + AI collaboration validated as winning model
- **March 2026 Patch Tuesday**: 67 CVEs, **4 actively exploited zero-days** (Windows kernel UAF for SYSTEM, NTFS, MMC); all 4 added to CISA KEV; Feb 2026 PT had 6 zero-days
- **Group-IB supply chain 2026**: 68% of global incidents involving severe consequences linked to compromised supply chains — nearly double from prior years
- **CyberStrikeAI dual-use incident** (Feb 2026): AI-native Go platform with 100+ security tools adopted by threat actors to compromise **500+ FortiGate devices** across 55 countries in 5 weeks; 21 unique IPs; no zero-days needed — just exposed management + weak auth + AI-generated attack plans
- **Adversa AI BIG Innovation Award** (March 2026): won 2026 BIG Innovation Award for agentic AI security platform; continuous AI red teaming for autonomous agents; MCP Security TOP 25 cited as leading knowledge base for MCP vulnerabilities and defenses
- **Ivanti EPMM mass exploitation** (March 2026): CVE-2026-1281/1340 (CVSS 9.8) pre-auth RCE via **bash arithmetic expansion** in `/mifs/c/appstore/fob/` and `/mifs/c/aftstore/fob/`; mass exploitation across US, Germany, Australia, Canada; sectors: gov, healthcare, manufacturing. Root cause: attacker input in `$((...))` arithmetic evaluation within `map-appstore-url` shell script
- **Zscaler 100% enterprise AI failure** (March 2026): red team testing found 100% of enterprise AI systems had critical flaws; median time to first critical failure: 16 minutes; 90% compromised under 90 minutes; one system bypassed in 1 second
- **Chrome Gemini panel hijack** (CVE-2026-0628, CVSS 8.8, March 2026): low-privilege extension escalates to camera/mic/file access via Gemini WebView; new browser privilege escalation surface from AI panel integration
- **Dell RecoverPoint CVE-2026-22769** (CVSS 10.0): hardcoded admin credentials exploited by China-nexus UNC6201 since mid-2024; Ghost NIC lateral movement technique; backup appliances as persistent APT footholds
- **Crimson Collective AWS threat** (March 2026): emerging actor targeting AWS environments via exposed long-term access keys and IAM misconfigs; claimed ~570GB data theft from Red Hat private GitLab repos
- **RSAC 2026 Innovation Sandbox** (March 23, 2026): ZeroPath (AI-native SAST, business logic + chained vulns), Crash Override (CI/CD provenance), Charm Security (agentic AI scam prevention), Token Security (machine identity) — all receive $5M; ZeroPath founded by 100K+ earned bug bounty hunter
- **All 12 prompt injection defenses bypassed** (March 2026): joint OpenAI/Anthropic/DeepMind research confirmed all published defenses bypassed at 90%+ success rate under adaptive conditions — defense-in-depth for AI is still insufficient
- **Bugcrowd CNA status** (March 2026): recognized as CVE Numbering Authority — direct CVE assignment for disclosed findings, streamlined disclosure pipeline
- **mcp-remote CVE-2025-6514** (March 2026): OS command injection via malicious `authorization_endpoint` in MCP remote proxy; **437K+ npm downloads**; demonstrates that MCP infrastructure itself is an attack surface, not just the servers it connects to
- **MCP reconnaissance pattern**: 70% of MCP traffic is `initialize` + `tools/list` + `disconnect` (reconnaissance, not exploitation); attackers are mapping the MCP ecosystem before attacking — treat MCP server enumeration as a recon signal
- **Claude DXT zero-click RCE** (LayerX, March 2026): CVSS 10.0 zero-click RCE in 50+ Claude Desktop Extensions; malicious Google Calendar invite silently compromises system; DXT runs unsandboxed with full system privileges; Claude chains low-risk connectors (calendar) to high-risk executors (local code); 10,000+ active users affected; **Anthropic declined to fix**, stating "falls outside current threat model" — pattern: MCP/DXT treats data from low-risk sources as trusted input for high-risk actions
- **Augustus LLM scanner** (Praetorian, February 2026): open-source Go-native LLM vulnerability scanner; 210+ adversarial probes, 28 LLM providers, single binary; covers 47 attack categories (jailbreaks, prompt injection, data extraction, agent attacks); "Buff" system for dynamic probe transformations (paraphrasing, translation, encoding); production-grade with concurrent scanning and rate limiting; Apache 2.0
- **CAI framework** (Alias Robotics, arXiv:2504.06017): open-source cybersecurity AI achieving 3,600x faster than humans on specific CTF tasks, 11x average; first among AI teams at "AI vs Human" CTF ($750 reward); enables non-professionals to discover CVSS 4.3-7.5 bugs at expert rates; 156x cost reduction for security testing; top-30 Spain, top-500 worldwide on HackTheBox within a week
- **SecureClaw** (Adversa AI, February-March 2026): first open-source OWASP-aligned security plugin for OpenClaw AI agents; 55 automated audit checks; addresses OpenClaw supply chain crisis (1,184+ malicious skills, 13+ CVEs, 42,665 exposed instances)
- **ReDAct + FrameShield** (March 2026): ReDAct framework disentangles goal vs framing representations in LLM activations; FrameShield anomaly detector significantly improves concealed attack detection — emerging defense to test against
- **GuardLLM** (March 2026): defense-in-depth Python library for LLM security — new defensive tool to evaluate and bypass
- **Apache Tika SSRF/XXE** (CVE-2025-66516, CVSS 10.0): unauthenticated remote SSRF/XXE in Apache Tika; primary ransomware target; pattern: document processing libraries as SSRF entry points
- **Flowise IDOR + business logic** (2026): critical IDOR in PUT `/api/v1/loginmethod` — any low-privilege user overwrites SSO config of any organization; no ownership validation; pattern: multi-tenant AI platforms with broken access control on admin endpoints
- **SmarterMail unauthenticated RCE** (CVE-2025-52691): insecure file upload → RCE; unauthenticated; webmail appliances as persistent access vectors
- **Gemini-in-Chrome CVE-2026-0628** (March 2026): Google Gemini AI integration in Chrome browser with disclosed vulnerability; AI browser integrations expanding Chrome's attack surface
- **Adversa AI March 2026 digest**: GenAI attacks becoming large-scale production operations; Anthropic exposure of massive distillation attacks; Microsoft discovery of widespread recommendation poisoning; industrial-scale incidents driving defense architecture evolution
- **Microsoft recommendation poisoning** (March 2026): widespread poisoning of AI recommendation systems documented; pattern: systemic manipulation of AI-mediated content delivery at scale
- **Anthropic Code Review** (March 9, 2026): agentic multi-step reasoning loop for security research; builds on Claude Code Security (500+ vulns, 22 Firefox CVEs); continuous improvement of AI-driven code review — competitive pressure on pattern-matching bounty submissions
- **XBOW Palo Alto GlobalProtect discovery** (March 2026): found vulnerability in Palo Alto GlobalProtect VPN; demonstrates AI agent capability expanding beyond web apps into enterprise network security appliances
- **XBOW #1 on HackerOne US leaderboard** (2025-2026): 1,060+ submissions in 90 days (54 critical, 242 high, 524 medium); **outcome breakdown: 130 resolved, 303 triaged, 208 duplicates (20%), 209 informative (20%), 36 N/A** — even the #1 bot gets ~40% non-payable results; 45% of findings still awaiting resolution; **85% solve rate on custom challenges**; **80x faster** than manual teams; launched **XBOW Pentest On-Demand** ($6K/engagement, 5 business days); pivoting to pre-production scanning; still operating at loss — compute exceeds bounty earnings. **XBOW methodology** ("Tales from the Trace" blog series): merges static source code analysis with dynamic exploitation — reads code to identify suspicious patterns, then crafts targeted exploits against the running application; uses automated "validators" (peer reviewers) to confirm each finding before submission
- **IDOR/access control reward surge** (HackerOne 2026): IDOR-related rewards increased **23% YoY**; valid IDOR reports grew **29% YoY**; fastest-growing payout category. Authorization flaws climbing CWE Top 25 (Missing Authorization up 5 positions to #4). Programs shifting rewards toward identity, access, and business logic; XSS/SQLi declining in relative payout value
- **GitHub Security Lab Taskflow Agent** (March 6, 2026): open-source structured security audit framework using LLMs; **80+ vulnerabilities** across 40 repos; business logic findings had 25% confirmed rate; **IDOR/access control produced the largest absolute count at 38 confirmed findings**; notable find: **CVE-2026-28514** (Rocket.Chat CVSS 9.3 — missing `await` on `bcrypt.compare()` → any password accepted); validates structured multi-step AI audit approach; async auth bypass is an emerging AI-discoverable bug class
- **PortSwigger Top 10 Web Hacking Techniques 2025** (published early 2026): parser differentials named #1 exploitation technique; Unicode normalization attacks create systematic WAF bypass gaps; HTTP/2 CONNECT exploitation resurfacing; side-channels named "the defining trend of 2025." **Hunter implication:** parser differentials are now a first-class cross-cutting attack family — see parser-differentials.md
- **TeamPCP cloud worm** (Feb-March 2026): compromised **60,000+ servers** across AWS (36%), Azure (61%), GCP; exploits exposed Docker APIs, misconfigured K8s, Ray dashboards, Redis servers; monetization: cryptomining, proxy sales, ransomware C2, data exfiltration. **Hunter implication:** cloud misconfigurations at scale remain highly exploitable; test Docker API exposure, Redis unauth, K8s dashboard access
- **AgentShield Benchmark** (March 2026): first open comparison of 6 AI agent security tools across 537 test cases; scores ranged 39-98; tools catching 95%+ prompt injections **miss most unauthorized tool calls**. **Hunter implication:** tool abuse and unauthorized tool execution are under-defended — focus MCP/agent testing on tool misuse, not just injection
- **94.4% of LLM agents vulnerable** to prompt injection, 83.3% to retrieval-based backdoors, **100% to inter-agent trust exploits** (arXiv:2510.23883). Multi-agent systems leak more data through internal channels than external outputs
- **50+ MCP CVEs by March 2026** (30 in 60 days; Adversa AI March digest); **42,665 exposed instances**, 5,194 actively vulnerable; **38%** of 500+ scanned servers completely lack authentication; **43% vulnerable to command execution**. MCP vulnerability rate accelerating, not plateauing
- **Autonomous jailbreak agents** (Nature Communications, March 2026): reasoning LLMs achieve **97.14% jailbreak success** across 9 target models in multi-turn conversations with zero human supervision (DOI: s41467-026-69010-1)
- **Coruna iOS exploit kit** (March 2026): nation-state grade kit targeting iOS 13-17.2.1 with 23 exploits across 5 chains; first mass exploitation of mobile phones by criminal groups using government-grade tools
- **n8n dual CVSS 10.0** (Jan-Feb 2026): CVE-2026-21858 (Ni8mare, unauth RCE via content-type confusion) + CVE-2026-25049 (sandbox escape via destructuring); ~100K servers impacted; workflow automation platforms as systematic attack surface
- **Cisco March patch wave**: 48 firewall vulnerabilities including dual CVSS 10.0 (CVE-2026-20079 + CVE-2026-20131 FMC) + SD-WAN auth bypass (CVE-2026-20127 CVSS 10.0) exploited since 2023 by UAT-8616
- **Phantom agent hijacking** (arXiv:2602.16958, Feb 2026): Template Autoencoder maps chat template structural patterns into latent space; **79.76% ASR** across 7 closed-source agents (GPT-4.1, Gemini-3); **70+ confirmed vendor vulnerabilities**; structural/token-level jailbreak that bypasses instruction-tuning — model-agnostic. First systematic approach to role confusion attacks
- **Sage ADR launch** (Gen Digital/Avast, March 2026): first Agent Detection & Response tool; **200+ detection rules** intercepting tool calls in Claude Code, Cursor, OpenClaw; 1,000+ installs within weeks. Signals enterprise defense tooling catching up to agent attack surface — rule-based detection will handle commodity attacks
- **SACR UADP framework** (March 2026): Software Analyst Cyber Research published first analyst framework for Unified Agentic Defense Platforms covering agent identity, tool authorization, memory protection, inter-agent communication security; runtime AI governance with intent-aware JIT enforcement; SentinelOne named leader. Signals enterprise buyer demand for agentic defense category
- **Microsoft March 2026 Patch Tuesday**: 84 CVEs, 6 zero-days — ICMP RCE CVE-2026-24985 (CVSS 9.8, no auth/interaction), Outlook EoP CVE-2026-24993 (CVSS 9.8, email auto-triggers), Windows kernel UAF → SYSTEM, NTFS heap read via USB, MMC exploitation; 4 actively exploited added to CISA KEV. VHD + MSC delivery vectors in phishing chains
- **CISA KEV March 2026 wave**: VMware Aria CVE-2026-22719 (command injection, active exploitation), SolarWinds WHD CVE-2025-26399 (CVSS 9.8, third deser bypass — Huntress observed ManageEngine RMM post-exploitation), Ivanti EPM CVE-2026-1603 (CVSS 8.6, "magic number 64" auth bypass), Omnissa Workspace ONE CVE-2021-22054 (SSRF). **10+ KEV additions in March** across 3 batches; 2 Emergency Directives active (ED 26-03 for Cisco SD-WAN). Enterprise management platforms are top exploitation targets
- **Google GTIG zero-day report** (March 5, 2026): 90 zero-days exploited in 2025 (up from 78 in 2024); **48% targeted enterprise** (network appliances #1 target); commercial spyware vendors surpassed state actors as largest zero-day users; PRC-nexus 10 zero-days; mobile zero-days rose to 15
- **Bug bounty market $2.06B** (2026), projected $7.74B by 2035 at 15.94% CAGR. 97% of organizations would consider AI pentesting (Aikido, 450 CISOs). 56.4% of ransomware CVEs first identified via zero-day exploitation (up from 33% in 2024; VulnCheck). Attackers exploit 28% of vulns within 1 day of CVE disclosure
- **mcp-atlassian CVE-2026-27825** (CVSS 9.1): critical unauth RCE + SSRF in most popular Atlassian MCP connector; 82% of 2,614 MCP implementations vulnerable to path traversal (Endor Labs)
- **Synack Sara launch** (March 2026): multi-agent PTaaS with hundreds of specialized AI agents + 1,500 human SRT researchers; PTaaS incumbents adopting agentic AI architecture
- **Microsoft expanded bug bounty** (March 2026): now covers third-party and open-source code impacting Microsoft services; Zero Day Quest spring 2026 targeting Azure, Copilot, Identity, M365
- **SAP March 2026 Patch Tuesday** (March 10): 15 security notes, 2 Critical (CVSS 9.8 Log4j 1.2.17 code injection + CVSS 9.1 insecure deser). Enterprise portal deser targets remain high-value
- **Android March 2026 bulletin**: 129 vulnerabilities patched; CVE-2026-0006 Critical System RCE in Media Codecs (no privileges, no interaction); CVE-2026-21385 Qualcomm 0-day (actively exploited, 234 chipsets, CISA KEV March 3)
- **Claude Code Review** (March 10, 2026): multi-agent PR review at $15-$25/review, ~20 min avg. Complements Claude Code Security for continuous code security scanning
- **Terra Security Portal** (March 10, 2026): $30M Series A from Felicis; agentic desktop app with Ambient AI (autonomous) + Copilot AI (human-governed) agents for live production pentesting; targets Fortune 100; signals shift toward human-governed agentic pentesting
- **Aikido Infinite** (February 26, 2026): first continuous self-remediating AI pentesting; every code change triggers agentic agents that discover, validate, remediate, and retest; found 7 CVEs in Coolify (including RCE as root) and CVE-2026-25545 (Astro SSRF); 97% of CISOs would consider AI pentesting (Aikido survey, 450 CISOs)
- **Cloudflare 2026 Threat Report** (March 3, 2026): **63% of all logins** involve previously compromised credentials; **94% of login attempts** originate from bots; signals shift from "breaking in" to "logging in" — credential-based attacks are the dominant threat vector
- **Cisco Emergency Directive ED 26-03** (March 2026): 24-hour patch mandate for CVE-2026-20127 (Cisco SD-WAN CVSS 10.0); largest SD-WAN attack spike March 4; UAT-8616 exploiting since 2023; chained with CVE-2022-20775 via software downgrade-then-restore; first Emergency Directive for SD-WAN infrastructure
- **AWS Security Agent multi-agent pentest** (February 2026): specialized agents for recon, vuln analysis, and exploit validation working collaboratively; supports shared VPC and GitHub Enterprise Cloud
- **PromptArmor plugin hook injection** (March 2026): Claude Code marketplace plugins inject prompts via exit-2 stderr channel → agent follows as system instructions → full codebase exfiltration demonstrated. Generalizable to any AI tool with plugin hook systems lacking source attribution
- **Salesforce bounty record year** (March 2026): **$6.2M paid in 2025** — 696 hackers, individual payouts up to $60K; $30.4M total since 2015. Mature enterprise programs paying more per finding as AI handles low-hanging fruit
- **HackerOne Agentic PTaaS** (January 2026): continuous pentesting combining AI agents with human experts; **88% accuracy** on fix-verified findings; trained on proprietary exploit intelligence
- **Gogs LFS supply chain risk** (March 2026): CVE-2026-25921 (CVSS 9.3) — cross-repository LFS object overwrite without auth. Git-based artifact registries as supply chain attack vectors
- **MCPTox benchmark** (arXiv:2508.14925): first systematic tool poisoning benchmark — 45 servers, 353 tools; **72.8% attack success** (o1-mini); more capable models MORE susceptible; <3% refusal rate
- **Cisco SD-WAN active exploitation** (March 2026): CVE-2026-20128/20122 web shell deployment confirmed (ACSC + CISA); combined with CVE-2026-20127 (CVSS 10.0) makes SD-WAN a top exploitation target
- **Nginx-UI CVE-2026-27944** (CVSS 9.8, March 5, 2026): unauthenticated backup download + AES-256 encryption key leaked in `X-Backup-Security` response header; credentials, session tokens, TLS private keys exposed. Pattern: admin panel backup endpoints without auth — generalizable to any management UI with backup/export features
- **WordPress Greenshift CVE-2026-2371** (March 6, 2026): broken access control in AJAX handler `gspb_el_reusable_load()` — missing `current_user_can()` check + nonce exposed on public pages → unauthenticated leak of private/draft reusable blocks. Pattern: WordPress plugin AJAX endpoints with missing authorization — affects 100K+ active installs
- **48% of cybersecurity pros** identify agentic AI as the #1 attack vector for 2026 — outranking deepfakes, ransomware, and supply chain compromise (Dark Reading poll, March 2026)
- **Bug bounty platforms market alternate projection**: $1.19B (2024) → **$3.98B by 2032** at 16.3% CAGR (Verified Market Research, March 2026); higher than earlier $2.06B (2026) base due to enterprise AI security demand
- **JetStream Security** ($34M seed, March 3, 2026): AI security startup with "AI Blueprints" platform mapping agents, models, data, tools, identities; led by Redpoint Ventures with CrowdStrike Falcon Fund; angels include George Kurtz (CrowdStrike), Assaf Rappaport (Wiz), Frederic Kerrest (Okta); signals VC conviction in AI security as distinct category
- **OWASP Red Teaming Vendor Eval v1.0** (March 6, 2026): first standard for evaluating AI red teaming providers; distinguishes simple GenAI (chatbots, RAG) from advanced (tool-calling agents, MCP, multi-agent); warns against vendors treating complex architectures as chatbots. Green/red flags, governance considerations
- **Microsoft March 2026 Patch Tuesday (second wave)**: 79-83 CVEs (BleepingComputer: 79 flaws; Tenable: 83 CVEs including Edge/Chromium); 2 zero-days publicly disclosed + 3 Critical (Excel info disclosure CVE-2026-26144 exploitable via Copilot Agent mode, Office RCE CVE-2026-26113, Office RCE CVE-2026-26110)
- **Knostic OpenAnt** (March 6, 2026): open-source LLM vulnerability scanner with two-stage detect+exploit pipeline; 6 languages; Apache 2.0; free alternative to commercial LLM scanning
- **Snyk Evo** (March 2026): first agentic security orchestration system unifying SAST, SCA, container, and IaC scanning; enterprises adopting Evo will have broader automated coverage — narrows the window for commodity vulnerability findings
- **IDOR/access control dominance** (2026): ~50% of all high and critical severity findings now involve broken access control (IDOR, authorization bypass, business logic); **18-29% YoY increase** in valid IDOR/IAC reports; access control flaws associated with $953.2M in losses; 210% increase in valid AI-related vulnerability reports; 339% jump in AI vulnerability bounties paid — signals dual opportunity in auth bypass and AI feature testing
- **ShinyHunters Salesforce Aura campaign** (March 8-10, 2026): UNC6240 breached ~400 organizations via misconfigured Experience Cloud guest user profiles; no CVE — customer misconfiguration not platform flaw; Salesforce paid $6.2M in bounties in 2025. **Hunter implication:** Salesforce Experience Cloud is a high-value target — guest user profile testing via AuraInspector yields Critical findings at enterprise programs; auth-vs-authz confusion (SiYuan CVE-2026-30926 pattern: auth check present, role check missing) is a systemic pattern across self-hosted apps
- **69 vulns in 5 AI coding tools** (March 2026): systematic audit of Claude Code, Codex, Cursor, Replit, Devin found SSRF in every single tool; all five introduced SSRF in URL preview features — SSRF is now a universal defect class in AI-generated code. **Hunter implication:** any vibe-coded app with URL preview/link unfurling is likely SSRF-vulnerable
- **Base44 critical auth bypass** (Wiz, March 2026): undocumented endpoints accepted non-secret `app_id` for full access to private enterprise apps; JWT shared between platform and hosted apps. Pattern generalizes to any SaaS builder that hosts user-created applications
- **XBOW global #1 confirmed** (Black Hat 2026): live demo finding vulnerabilities on stage; 1,060 total submissions; pivoting to pre-production scanning; HackerOne restructured leaderboards to separate AI from human researchers; arXiv paper challenges "fully autonomous" claims — all findings required human review pre-submission. Published IDOR methodology showing reasoning-based authorization testing (CVE-2026-22589/22588 in Spree v5.2.5) — drops cookies, tests unauth access after 403, pattern-matches human pentesters
- **Endor Labs MCP = classical AppSec** (March 2026): 82% of 2,614 MCP implementations share CWE-22 path traversal; MCP concentrates existing risks into single layer; Anthropic mcp-server-git 3 CVEs (Jan 2026). **Hunter implication:** MCP hunting is classical AppSec at scale — CWE-22/CWE-77 skills transfer directly, massive underexplored surface
- **Docker MCP supply chain documentation** (March 2026): first full system compromise documented via MCP infrastructure; mcp-remote 437K downloads affected; MCP Inspector drive-by exploit; WhatsApp MCP exfil. Docker's "MCP Horror Stories" series establishing MCP as a recognized supply chain attack vector
- **Obsidian Security MCP+OAuth convergence** (March 2026): one-click ATO in 7 major MCP clients via OAuth flow manipulation; remote MCP servers from well-known organizations vulnerable to authorization code theft. Pattern: MCP server as both OAuth authorization server and client creates CSRF-style attack surface
- **Google Big Sleep 20 zero-days** (2025-2026): LLM-guided variant analysis found 20 flaws in FFmpeg, ImageMagick, SQLite (CVE-2025-6965 CVSS 7.2); DeepMind + Project Zero collaboration; confirmed AI can find bugs missed by years of fuzzing. **Hunter implication:** open-source C/C++ memory safety bugs increasingly AI-found — focus human effort on logic bugs in same codebases
- **curl bug bounty ended** (January 2026): first program shutdown attributed to AI slop; 8x submission volume, 5% genuine rate; not a single AI-only submission found a real vulnerability. **Hunter implication:** programs are now deploying AI-based triage filters (HackerOne Hai Insight) — low-quality submissions risk reputation damage
- **IronCurtain** (February 27, 2026): open-source safeguard layer for autonomous AI assistants; sandboxing, policy enforcement, and runtime isolation for AI agents; signals growing defense ecosystem for agent runtime security
- **Microsoft Agent 365** (March 9, 2026): runtime threat protection for AI agents using Agent 365 tools gateway; detection, blocking, and investigation of malicious agent activities; GA May 1, 2026, $15/user/month. **Hunter implication:** enterprises deploying Agent 365 will have agent activity monitoring — focus on bypass vectors and pre-detection exploitation windows
- **XBOW CVE-2026-21536** (March 2026): first AI-discovered Windows CVE — critical RCE (CVSS 9.8) in Microsoft Devices Pricing Program component; demonstrates AI agents finding high-severity zero-days without source code access. **Hunter implication:** commodity pattern-matching vuln classes increasingly AI-found; invest time in logic bugs and multi-step chains
- **Lovable VDP broken access control** (March 9, 2026): non-premium users can change project visibility to premium-only levels by modifying request body; classic BFLA pattern — UI restriction without server-side enforcement. **Hunter implication:** vibe-coded SaaS builders (Lovable, Base44, Bolt, Replit) are systematically under-enforcing authorization on plan/feature toggle endpoints
- **Chartbrew CVE-2026-27005** (March 2026): critical SQL injection in `applyMysqlOrPostgresVariables` — unsanitized user input in SQL query construction; affects versions before 4.8.3. Pattern: analytics/dashboard tools with dynamic query builders are high-value SQLi targets
- **CVE-2026-23674 MapUrlToZone bypass** (March Patch Tuesday): URL-to-security-zone mapping manipulation enables trusted-zone code execution from untrusted content; crafted URLs bypass Internet Explorer security zone restrictions. Pattern: URL parsing differential between zone classifier and handler — test any system that makes security decisions based on URL classification
- **Prompt Security MCP risk scoring** (March 2026): runtime enforcement layer for agentic AI with risk scoring across 13,000+ MCP servers; focuses on MCP-based autonomy and agent-server interaction monitoring. **Hunter implication:** MCP server risk scores will influence enterprise adoption decisions — low-score servers become priority targets for security research
- **Cyata Agentic Security Posture Management** (March 2026): continuous discovery and governance of AI agents across all environments; maps which agents exist, who owns them, what they access, and how they behave. **Hunter implication:** enterprises gaining visibility into agent sprawl — test for blind spots in agent discovery (ephemeral agents, nested tool chains, cross-platform agent identity)
- **Authzed MCP Breach Timeline** (March 2026): first comprehensive timeline of MCP security incidents — WhatsApp history exfil via tool poisoning (Apr 2025), GitHub private repo leak (Apr 2025), MCP Inspector RCE (May 2025), mcp-remote CVE-2025-6514 437K installs (Jun 2025); **MCP has become AI's fastest-growing attack surface** with authorization failures as the dominant root cause, not sophisticated zero-days. **Hunter implication:** MCP server auditing is high-ROI low-competition work — 38% of servers still lack basic auth
- **CVE-2026-26125 Payment Orchestrator missing auth** (March PT, CVSS 8.6): EoP via missing authentication in Microsoft Payment Orchestrator Service. **Hunter implication:** payment/billing service endpoints are systematically undertested for unauth access — test orchestrator, billing, and subscription management endpoints in all SaaS targets
- **Cloudflare AI Playground XSS** (CVE-2026-1721, March 2026): reflected XSS in Cloudflare's AI Playground OAuth handler due to unescaped `error_description` parameter interpolation. Pattern: OAuth error handling in AI platforms systematically lacks output encoding — test `error`, `error_description`, `state` params in all AI tool OAuth flows
- **Microsoft SharePoint RCE pair** (CVE-2026-26106/26114, March Patch Tuesday): critical unauthenticated RCE in on-premises SharePoint Server — CVE-2026-26106 allows code execution without auth, CVE-2026-26114 is deserialization-based. Pattern: on-prem SharePoint remains a persistent attack surface — 84 CVEs in March 2026 alone, 8 Critical
- **83% agentic AI deployment gap** (AIMult survey, March 2026): 83% of organizations planned agentic AI deployment but only 29% ready to operate securely. **Hunter implication:** massive "deploy first, secure later" window — enterprises racing to ship agentic capabilities create widespread authorization and isolation gaps
- **Azure MCP Server SSRF** (CVE-2026-26118, CVSS 8.8, March 10, 2026): server-side request forgery in Azure MCP Server Tools — attacker submits malicious URL instead of Azure resource identifier, MCP Server sends request including its **managed identity token**; captured token grants permissions of the MCP Server's managed identity across authorized Azure resources. Requires authenticated access to MCP-backed agent. **Hunter implication:** any cloud MCP server accepting user-provided URLs as resource identifiers is a token-theft vector — test all cloud AI service MCP integrations (Azure, AWS Bedrock, GCP Vertex) for SSRF-to-token-exfil pattern
- **Active Directory CVE-2026-25177** (March 10, 2026): EoP in AD DS caused by improper restriction of names/resources; authenticated attacker gains elevated domain privileges. **Kerberos CVE-2026-24297** (March 10, 2026): security feature bypass (CVSS 6.5, PR:N) — race condition in Kerberos auth allows circumventing security controls; chainable as initial access vector. **Scope note:** relevant mainly to self-hosted enterprise products, hybrid agent deployments, VPN/admin portals, and on-prem bug bounty programs exposing AD infrastructure
- **SQL Server EoP CVE-2026-21262** (March 10, 2026, publicly disclosed zero-day): privilege escalation to sysadmin; chains with SharePoint RCE (CVE-2026-26106 unauth + CVE-2026-26114 deser) and AFD.sys EoP (CVE-2026-24293 → SYSTEM). Additional SQL Server 2025-specific EoP: CVE-2026-26116. **Scope note:** on-prem Microsoft infrastructure programs only — chain initial web access with local privesc for maximum severity
- **Cisco State of AI Security 2026** (March 2026): 83% of orgs planned agentic AI deployment, only 29% ready to secure it; MCP ecosystem explicitly flagged for tool poisoning, RCE, overprivileged access, supply chain tampering; 86% of CISOs fear agentic AI social engineering; report examines AI supply chain fragility across datasets, models, and tools. **Hunter implication:** enterprise "deploy first, secure later" creates 6-12 month window where MCP integrations ship with default credentials and broad tokens
- **30 MCP CVEs in 60 days** (Kai Security analysis, Jan-Feb 2026): class breakdown — 43% exec/shell injection (13 CVEs), 20% tooling/infrastructure layer (6 CVEs), 13% auth bypass (4 CVEs), 10% path traversal/argument injection (3 CVEs, incl. Anthropic's own reference impl), 7% new classes (eval injection, env var injection). **Hunter implication:** MCP vuln class diversity expanding — tooling-layer attacks (scanners, inspectors, host apps) are an under-tested surface class targeting developers, not end users
- **Endor Labs MCP AppSec analysis** (March 2026): 82% of MCP implementations use file operations prone to CWE-22 (path traversal), 67% CWE-94 (code injection), 34% CWE-78 (command injection), 5-7% CWE-79/89/601 (XSS/SQLi/open redirect). Based on first-year CVE analysis + Anthropic mcp-server-git triple CVE disclosure. **Hunter implication:** classical AppSec vulns concentrate in MCP layer at scale — apply web vuln testing methodology directly to MCP tool parameters
- **Semgrep AI business logic detection private beta** (March 2026): 1.9x better IDOR recall vs standalone AI assistants (Claude Code); 80% of alpha customers found critical/severe IDOR; hybrid SAST+LLM reasoning approach. **Hunter implication:** IDOR detection being automated — human edge moves to cross-tenant chains, multi-step business logic, and context-dependent access control flaws
- **MCPwnfluence mcp-atlassian** (CVE-2026-27825 CVSS 9.1 + CVE-2026-27826 CVSS 8.2, Feb 24 2026): critical unauth RCE+SSRF in most-downloaded Atlassian MCP server (4.4K stars, 4M+ downloads); path traversal in attachment download → arbitrary file write → `.bashrc`/`.ssh/authorized_keys` overwrite; plus SSRF via unvalidated `X-Atlassian-Jira-Url` headers. Fixed v0.17.0 with `validate_safe_path()`. **Hunter implication:** test ALL SaaS-bridging MCP servers for path traversal in file download tools and header-based SSRF — mcp-atlassian pattern generalizes to any connector handling remote files
- **XBOW GPT-5 integration** (March 2026): integrating GPT-5 alone **doubled performance** on both benchmarks and real-world targets; executed **48-step exploit chains**; broke cryptographic implementations in 17 minutes; matched a principal pentester's **40-hour assessment in 28 minutes**. **Hunter implication:** XBOW's chain depth (48 steps) means even multi-step exploitation is within AI reach — human edge narrows to business-context chains, multi-tenant isolation reasoning, and novel attack patterns XBOW hasn't been trained on
- **Jenkins stored XSS** (CVE-2026-27099, March 2026): stored XSS in node offline cause description affecting Jenkins 2.483-2.550 and LTS 2.492.1-2.541.1. **Hunter implication:** Jenkins admin panel injection surfaces remain undertested; check all user-controllable fields that render in admin dashboards (agent labels, node descriptions, build parameters)
- **Malicious Rust crate supply chain** (March 4, 2026): `time-sync` crate published to crates.io exfiltrating `.env` files; removed ~50 minutes after publication. **Hunter implication:** supply chain attacks targeting language package managers continue across ecosystems (npm, PyPI, now crates.io) — monitor package registries for typosquatting and rapid-publish/rapid-remove patterns targeting CI/CD pipelines
- **MCP server auth gap persists** (March 2026): 38% of 500+ scanned MCP servers completely lack authentication (Prompt Security, 13,000+ servers scored); reinforces Endor Labs finding that 82% of implementations share classical AppSec flaws. **Hunter implication:** unauthenticated MCP servers remain the lowest-hanging fruit in AI security — enumerate exposed servers via Shodan/Censys and test for path traversal + command injection
- **ContextCrush documentation channel poisoning** (Noma Labs, Feb 18-23 2026): Context7 MCP server served user-generated "Custom Rules" verbatim to all users querying a library — no sanitization, open registration, gameable credibility. PoC demonstrated `.env` exfiltration + file deletion via agent's tool access. ~8M npm downloads affected. Fixed Feb 23. **Hunter implication:** any MCP server aggregating third-party/user-generated content (documentation hubs, package registries, marketplace listings, community templates) is a prompt injection delivery channel — the payload lives on trusted infrastructure itself, not in user messages
- **CVE-2026-30903 Zoom Workplace Mail** (March 11, 2026, CVSS 9.6): External Control of File Name/Path in Zoom Workplace for Windows Mail feature — unauthenticated, network-exploitable, no user interaction; arbitrary file write → DLL hijack → privilege escalation. Affects < v6.6.0. **Hunter implication:** desktop collaboration tool mail/attachment handlers are a systematic attack surface for file path control → EoP chains; test mail processing in Zoom, Teams, Slack, Webex for similar path manipulation
- **HackerOne autonomous agent submissions** (March 2026): 560+ valid vulnerability reports submitted by autonomous AI agents in the past year; 82% of hackers now use AI workflows (up from 64% in 2023). HackerOne paid $81M in bounties (13% YoY increase). Prompt injection reports surged 500%+ as fastest-growing AI security category. **Hunter implication:** autonomous agents now contribute meaningful valid findings — commodity vuln classes increasingly AI-found; invest in logic bugs, multi-step chains, and novel attack patterns
- **XBOW 1,060 attacks analysis** (March 2026): XBOW's blog "We Ran 1,060 Autonomous Attacks" directly counters international AI safety report claiming autonomous attacks "have not been reported"; XBOW demonstrated 48-step exploit chains, 17-minute crypto breaks, matched a principal pentester's 40-hour assessment in 28 minutes. XBOW raised $75M Series B ($117M total, Altimeter + Sequoia). Companies verified: AT&T, Epic Games, Ford, Disney. Still operating at a loss — compute exceeds bounty earnings. **Hunter implication:** XBOW's capabilities are real but economics favor human hunters who can target high-value logic bugs with minimal compute cost
- **Noma Security MCP blindspots expanded** (March 2026): 5 critical blindspots identified across 13,000+ MCP servers: (1) typosquatting (Playwright MCP example), (2) excessive permissions (code exec, file access, API calls), (3) malicious code execution via prompt injection, (4) insecure communications (full context transmitted to servers including secrets), (5) inadequate observability (no audit trail for agent-server interactions). >90% of orgs maintain dangerous default configurations. **Hunter implication:** MCP servers accepting untrusted input AND exposing sensitive capabilities are the highest-risk intersection; test for context leakage where MCP servers log or can access full conversation context including secrets

