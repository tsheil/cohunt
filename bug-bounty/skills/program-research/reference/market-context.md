# Market Context (2025-2026)

*Last updated: March 2026 (v1.3.0)*

Reference data for program evaluation. Loaded on demand when program-research needs market statistics, reward benchmarks, notable programs, competition landscape, or disclosed vulnerability examples.

> **For AI/MCP-specific market data**, see `ai-hunting/reference/market-context.md`. For AI security tools, see `ai-hunting/reference/tools-landscape.md`. This file focuses on bug bounty program economics and hunter-relevant competitive data.

## Table of Contents

- [Key Market Metrics](#key-market-metrics)
- [AI & Automation Summary](#ai--automation-summary)
- [Platform & Vendor Statistics](#platform--vendor-statistics)
- [Notable Programs & Expansions](#notable-programs--expansions)
- [Notable Disclosed Vulnerabilities](#notable-disclosed-vulnerabilities)
- [Hacker Demographics](#hacker-demographics)
- [Competition Landscape](#competition-landscape)
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

## AI & Automation Summary

Key program-evaluation metrics for AI scope. For detailed AI/MCP attack surface data, tools, and techniques, see `ai-hunting/reference/market-context.md` and `ai-hunting/reference/tools-landscape.md`.

| Metric | Value | Source |
|--------|-------|--------|
| AI vulnerability reports | 210% increase YoY | HackerOne 2025 |
| Programs with AI in scope | 1,121 (270% YoY increase) | HackerOne 2025 |
| Hacker AI adoption | 82% (up from 64% in 2023) | Bugcrowd 2026 |
| AI security startup funding Q4 2025 | $2.17B across 28 deals (8x growth over 2 years) | Industry 2026 |
| Enterprise AI readiness gap | Only 34% have AI-specific security controls | Help Net Security 2026 |
| MCP CVEs filed | 50+ by March 2026 (30 in 60 days) | MCP Security Research 2026 |

---

## Platform & Vendor Statistics

| Metric | Value | Source |
|--------|-------|--------|
| Avg pentest findings | 12 vulns, 16% high/critical | HackerOne 2025 |
| Bug bounty high/critical rate | 25% of submissions | HackerOne 2025 |
| Top vulnerability type (bounty) | XSS (but declining) | HackerOne 2025 |
| Fastest growing vuln class | Authorization flaws (IDOR, BOLA) | HackerOne 2025 |
| IAC award growth | **134% YoY** to $4M+ (Improper Access Control) | HackerOne 2026 |
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
- **Ethereum Foundation** (March 11, 2026): maximum bounty quadrupled from $250K to **$1M** for critical bugs fundamental to blockchain integrity
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
- VDP as compliance driver (March 2026): EU CRA creates reporting obligations for in-scope manufacturers from September 2026; ISO/IEC 29147 and 30111 are the direct coordinated vulnerability disclosure standards; NIST CSF 2.0 maps to vulnerability disclosure handling but does not itself mandate a public VDP
- CVE-2026-28514 (Rocket.Chat, CVSS 9.3): critical authentication bypass — missing `await` on async `bcrypt.compare()` in microservices account-service; un-awaited Promise evaluates truthy → any password accepted for any user with bcrypt hash set; discovered by GitHub Security Lab Taskflow Agent. Pattern: async/await bugs in auth paths
- CVE-2026-3783 (curl, March 11 2026): OAuth2 bearer token leak via `.netrc` redirect — `machine` or `default` entries bypass `Curl_auth_allowed_to_host()` check; affects curl 7.33.0 through 8.18.0; curl bug bounty already ended but report came through HackerOne
- CVE-2026-1603 (Ivanti EPM, CISA KEV March 9 2026): authentication bypass before 2024 SU5 allowing unauthenticated attacker to leak stored credential data; CVSS 8.6; Horizon3.ai research details a crafted HTTP request with specific header manipulation reaching protected endpoints (Horizon3.ai)
- CVE-2026-24858 (Fortinet FortiOS SSO, CVSS 9.4): FortiCloud SSO authentication bypass — attacker with a FortiCloud account and registered device can log into other users' devices when FortiCloud SSO is enabled; affects FortiOS/FortiManager/FortiWeb/FortiProxy/FortiAnalyzer; actively exploited since January 2026; Arctic Wolf observed automated attack cluster creating generic accounts and exfiltrating firewall configs
- CVE-2025-26399 (SolarWinds WHD, CVSS 9.8): unauthenticated RCE via insecure deserialization in jabsorb library AjaxProxy; patch bypass of two prior CVEs (CVE-2024-28988, CVE-2024-28986) — triple-CVE bypass chain; must chain with CVE-2025-40536 for CSRF bypass; Huntress observed active exploitation with LSASS credential theft and ransomware
- CVE-2026-25177 (AD Domain Services, March Patch Tuesday): Unicode character manipulation creates duplicate SPNs/UPNs bypassing AD security checks — only requires standard SPN write permissions; novel privilege escalation technique
- CVE-2026-26110 + CVE-2026-26113 (Microsoft Office, CVSS 8.4 each, March 11 Patch Tuesday): RCE via Preview Pane — untrusted pointer dereference and type confusion; no user click required beyond previewing a document. Pattern: any target rendering user-uploaded documents server-side
- CVE-2026-26125 (Microsoft Payment Orchestrator, CVSS 8.6, March 11): missing authentication on payment service endpoint — elevation of privilege; template for business-logic testing on fintech payment endpoints
- CVE-2026-21262 (SQL Server, CVSS 8.8, March 11): publicly disclosed zero-day; authenticated user → SQL administrator privilege escalation
- MCP offensive infrastructure in the wild (Cyberwarzone, March 11 2026): exposed server hosting AI-assisted intrusion infrastructure using MCP to connect LLMs to attack environments; over 1,000 files including credential dumps; **43% command-execution vulnerability rate** across scanned MCP servers; confirms MCP servers deployed in production without basic security controls
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
- XBOW reached #1 on both US and global HackerOne leaderboards with 1,400+ vulnerability findings, running 80x faster than manual teams; raised **$117M total** ($75M Series B led by Altimeter/Sequoia); pivoting to pre-production scanning in 2026; **CVE-2026-21536** (CVSS 9.8) — AI-discovered critical RCE in Microsoft Devices Pricing Program, one of the first CVEs attributed to an AI agent against a Microsoft cloud service; demonstrated fully autonomous vulnerability discovery without source code access
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
- **Google completes $32B Wiz acquisition** (March 11, 2026): largest acquisition in Google's history; Wiz maintains multi-cloud stance (AWS, Azure, OCI, GCP); integration seams between Wiz and Google Cloud are potential attack surface during merge; Google bounty scope may expand to Wiz-integrated products
- **F5 BIG-IP v21.1 MCP traffic security** (March 11, 2026): first network appliance vendor with MCP-specific session persistence, traffic inspection, and security protections; NGINX expanded to inspect AI agent traffic; creates new attack surface for MCP WAF bypass and parsing differentials between F5 inspection and actual MCP servers

---

## Industry Trends & Predictions

> **For AI/MCP security tools, attack techniques, and incident catalog**, see `ai-hunting/reference/tools-landscape.md`, `ai-hunting/reference/market-context.md`, and `ai-hunting/reference/ai-case-studies.md`.

Key program-relevant trends:

- XSS/SQLi rewards declining — programs shifting toward identity, access control, and business logic flaws
- Continuous assurance testing replacing annual pentests as industry standard
- **EU AI Act compliance deadline August 2, 2026** — driving mandatory AI red teaming demand
- **Business logic = 45% of all bounty awards** (Intigriti 2026); **IDOR rewards surging** (+23% payout, +29% valid reports YoY)
- **Access control flaws** caused $953.2M in losses; logic flaws another $63M (FailSafe 2025). ~50% of all high/critical severity findings are now broken access control (IDOR, authorization bypass); **18-29% YoY increase** in valid IDOR/IAC reports — fastest-growing payout category
- **Supply chain compromise** nearly quadrupled since 2020 (IBM X-Force 2026); 454,648 malicious npm packages in 2025
- **curl ended bug bounty** (Jan 2026) — first shutdown from AI slop; confirmed-rate dropped below 5%
- **Joseph Thacker (rez0)**: 2x submissions predicted in 2026; internal AI code review will commoditize external bug bounty in 1-2 years
- **Vibe-coded apps**: 2,000+ vulns in 5,600 apps (Escape.tech) — systematically vulnerable new target class
- **Unit 42 2026 Global IR Report**: attacks accelerated 4x YoY; identity weaknesses in 89% of incidents
- **Google 2025 zero-day review**: 90 zero-days; 48% targeted enterprise technology (new high)
- **Bug bounty market alternate projection**: $1.19B (2024) → $3.98B by 2032 at 16.3% CAGR (Verified Market Research)
- **48% of cybersecurity pros** rank agentic AI as #1 attack vector for 2026 (Dark Reading poll)
- **Promptfoo acquired by OpenAI** (March 9): $86M valuation; frontier labs consolidating AI red-team tooling
- **Claude Code Review** (March 9-10): multi-agent PR review at $15-$25/review; 84% finding rate on large PRs; <1% rejection rate
- **OWASP Red Teaming Vendor Eval v1.0** (March 6): first standard for evaluating AI red teaming providers
- **ShinyHunters Salesforce Aura** (March 8-10): UNC6240 breached ~400 orgs via Experience Cloud guest user misconfig; $6.2M Salesforce bounties in 2025; no CVE — misconfiguration class
- **SiYuan auth-vs-authz pattern** (CVE-2026-30926): systemic — CheckAuth present but CheckRole missing across 4+ endpoints; applies broadly to self-hosted apps with role-based access
- **XBOW deterministic validators** (March 2026): key methodology behind #1 HackerOne ranking — uses headless browsers for XSS verification, automated two-request diff for IDOR, OAST callbacks for SSRF/RCE; 0% false positive rate on confirmed submissions; deterministic (non-LLM) validation is the differentiator vs other AI tools with ~25% N/A rates
- **HackerOne hackbot policy** (Feb 2026): all automated/AI findings require human expert review before submission; hackbots must operate within published VDP + HackerOne Code of Conduct; fully autonomous submission prohibited; HackerOne does NOT train AI on researcher submissions (CEO Kara Sprague confirmation Feb 2026)
- **HackerOne AI Research Safe Harbor** (Jan 2026): opt-in safe harbor for AI system testing; Gold Standard Safe Harbor enabled by default for new programs; AI-specific protections for good-faith security research
- **March 2026 Patch Tuesday** (March 10): 77-83 CVEs depending on counting methodology (CrowdStrike: 82; Tenable: 83 incl. Edge/Chromium); 8 critical; **55% privilege escalation bugs** (CrowdStrike: 46 EoP); SharePoint RCE (CVE-2026-26106), Windows Kernel EoP (CVE-2026-24289 exploitation more likely), .NET DoS (CVE-2026-26127), ASP.NET DoS (CVE-2026-26130), ACI Confidential Container EoP (CVE-2026-23651/26124); 2 publicly disclosed but none confirmed exploited
- **XBOW CVE-2026-21536 first AI-attributed Microsoft CVE**: CVSS 9.8 RCE in Microsoft Devices Pricing Program (exclusively hosted cloud service, already mitigated server-side); fully autonomous discovery without source code access; milestone for AI agent vulnerability discovery
- **Winlogon EoP CVE-2026-25187** (March 10 Patch Tuesday): discovered by Google Project Zero; CVSS 7.8; exploitation more likely; Winlogon is high-value persistence target
- **IDOR prevalence by industry** (HackerOne 2021 "Rise of IDOR"): government 18%, medical technology 36%, professional services 31%, retail/ecommerce 15% of total bounty spend — access control remains the highest-paying bug class across regulated industries; 2025-2026 data shows +18-29% YoY growth in valid IDOR/IAC reports
