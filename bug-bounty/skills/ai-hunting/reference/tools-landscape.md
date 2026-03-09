# AI Security Tools Landscape

Comprehensive catalog of AI-powered security tools for bug bounty hunting. Reference file for the ai-hunting skill.

> **Related files:** [mcp-playbooks.md](mcp-playbooks.md) for MCP test procedures and scanning tools | [ai-case-studies.md](ai-case-studies.md) for real-world incident case studies

---

## Table of Contents

- [Code Audit Tools (Vulnhalla, Hound, CAI, HexStrike, Shannon, AISLE, Codex Security, Claude Code Security)](#vulnhalla-pattern-codeql--llm-guided-questioning)
- [Autonomous Pentesting (Strix, NeuroSploit, PentAGI, Reaper, Shannon, AISLE, Codex Security)](#strix-autonomous-ai-pentesters)
- [Red Teaming Frameworks (AgentFence, Augustus, Garak)](#agentfence-ai-agent-security-testing)
- [AI-Powered Subdomain Enumeration](#ai-powered-subdomain-enumeration)
- [AI-Assisted Fuzzing & Payload Generation](#ai-assisted-fuzzing--payload-generation)
- [The XBOW/Autonomous Agent Reality Check](#the-xbowautonomous-agent-reality-check)
- [The 2026 Tool Landscape (40+ tools)](#the-2026-tool-landscape)

---

**Vulnhalla Pattern** (CodeQL + LLM-guided questioning):
- Feed code + CodeQL results into Vulnhalla or similar (CAI, Hound)
- AI asks you clarifying questions: "Is this user input validated here? What's the database query?"
- Result: **96% false positive reduction** vs raw CodeQL alerts
- Only investigate high-confidence findings → faster audits, fewer rabbit holes

**Hound (Knowledge-Graph-Based Code Auditor):**
- Open-source agent that models the cognitive processes of expert auditors
- Builds **adaptive knowledge graphs** of the target system — components, relationships, data flows
- Observations, assumptions, and hypotheses evolve with confidence scores, enabling long-horizon reasoning
- Uses **dynamic model switching**: lightweight "scout" models for exploration, heavyweight "strategist" models for deep reasoning
- Language-agnostic: works on any codebase, not just specific languages
- Best for: deep logic bugs, complex multi-file vulnerability chains, cumulative audits
- GitHub: `muellerberndt/hound`

**CAI (Cybersecurity AI Framework):**
- Open-source framework supporting 300+ AI models for building bug-bounty-ready security agents
- CAI achieved **Top-10 ranking in the Dragos OT CTF 2025** (Rank 1 during hours 7-8), completing 32 of 34 challenges with a 37% velocity advantage over top human teams
- Achieved first place among AI teams in the "AI vs Human" CTF challenge; solves challenges up to 3,600x faster than humans in specific tasks, averaging **11x faster** overall
- Reached **top-30 in Spain** and **top-500 worldwide** on Hack The Box within a week of active operation
- Reduces security testing costs by an average of **156x** compared to traditional approaches
- **Non-professional enablement:** enables non-professionals to discover significant security bugs (CVSS 4.3-7.5) at rates comparable to experts during bug bounty exercises — democratizing offensive security
- HackerOne's top engineers leverage CAI to explore agentic AI architectures; CAI's Retester agent inspired HackerOne's AI-powered Deduplication Agent, now in production
- Published as academic paper: "CAI: An Open, Bug Bounty-Ready Cybersecurity AI" (arXiv:2504.06017)
- **Security note:** CAI itself was found to have a critical command injection vulnerability (CVE-2025-67511, CVSS 9.6-9.7) — AI security tools can themselves be vulnerable
- GitHub: `aliasrobotics/cai`

**HexStrike AI (MCP-Based Security Automation):**
- Open-source MCP server bridging LLMs (Claude, GPT, Copilot) with 150+ professional security tools
- v6.0 features multi-agent architecture with autonomous AI agents and intelligent decision-making
- Integrates tools like Nmap, Burp Suite, Ghidra, and Metasploit via Model Context Protocol
- Claims 98.7% detection rates and 24x speed improvements over manual workflows
- **Now included in Kali Linux** tools — signaling commoditization of AI-assisted offensive security
- **Dual-use warning:** threat actors weaponized HexStrike to scan and exploit vulnerable instances within 12 hours of disclosure — stay ethical
- Best for: automated recon pipelines, tool orchestration, MCP-native security workflows
- GitHub: `0x4m4/hexstrike-ai`

**PentestAgent:**
- AI penetration testing tool with prebuilt attack playbooks and HexStrike integration
- Combines structured testing methodologies with LLM-driven decision-making
- Useful for automated exploitation chains in structured environments

**AgentFence (AI Agent Security Testing):**
- Open-source platform for automatically testing AI agent security
- Detects vulnerabilities like prompt injection, secret leakage, and system instruction exposure
- Best for: testing AI-powered features before reporting findings

**Strix (Autonomous AI Pentesters):**
- Open-source autonomous AI agents that behave like real attackers — run code, explore apps, find vulns, validate with working PoCs
- HTTP proxy for request/response manipulation, browser automation for client-side testing (XSS, CSRF), terminal sessions for command tests
- Bug bounty automation: generates PoCs for faster reporting; integrates into CI/CD pipelines
- Used by Fortune 500 security engineers and top 1% HackerOne hunters
- ~2,000 GitHub stars and ~8,000 downloads since launch
- GitHub: `usestrix/strix`

**NeuroSploit v2/v3:**
- AI-powered pentest framework with multi-LLM support (Claude, GPT, Gemini, Ollama)
- 9 specialized agent personas: bug bounty hunter, red team operator, malware analyst, OWASP expert, etc.
- v3 transforms from scanner to autonomous agent: real-time strategy adaptation, exploit chaining, anti-hallucination safeguards
- Executes tools inside isolated Kali Linux containers; integrates Nmap, Metasploit, Subfinder, Nuclei, SQLMap via JSON config
- Open-source (MIT); GitHub: `CyberSecurityUP/NeuroSploit`

**PentAGI:**
- Fully autonomous multi-agent AI pentesting platform coordinating researcher, developer, and executor agent roles (Go-based, open-source)
- Sandboxed Docker environment with 20+ tools (Nmap, Metasploit, SQLMap); isolated execution for safety
- Graphiti-powered knowledge graph using Neo4j for long-term memory of research findings
- Integrated browser and external search APIs for dynamic reconnaissance
- Best for: autonomous pentesting workflows requiring cross-tool coordination and persistent memory
- GitHub: `vxcontrol/pentagi`

**Reaper (Ghost Security):**
- Open-source agentic web app security testing and tampering tool
- Designed for AI-driven web application testing workflows

**Shannon (Autonomous AI Pentester):**
- Fully autonomous white-box AI pentester for web apps and APIs by Keygraph; **powered by Claude Agent SDK**
- Four-phase multi-agent pipeline: reconnaissance, parallel vulnerability analysis, exploitation, and reporting
- Analyzes source code to guide attack strategy, then validates with live browser and CLI-based exploits
- Scored **96.15% (100/104 exploits)** on a hint-free variant of the XBOW benchmark using **Code Property Graphs (CPG)** for semantic understanding of code structure and data flows
- **Full-loop capability**: discover → validate → exploit — handles the entire vulnerability lifecycle autonomously
- Discovered 20+ critical vulns in OWASP Juice Shop in a single automated run, including full auth bypass and complete DB exfiltration
- Automates reconnaissance (Subfinder, Amass, WhatWeb), vulnerability scanning (Nuclei, ffuf), exploit generation, and report writing
- Handles 2FA logins and browser-based attacks without human input; runs via Docker containers
- **Pricing**: subscription-based — Free ($0, 80K daily tokens), Starter ($3.14/mo), Plus ($5.99/mo), Pro ($49.99/mo, 10M daily tokens), Enterprise (custom). Previously characterized as ~$50/run, but actual model is subscription with token limits
- Rapidly growing: GitHub's fastest-rising security project in early 2026
- GitHub: `KeygraphHQ/shannon`

**AISLE (AI-Driven Vulnerability Discovery):**
- Landmark achievement in January 2026: discovered **12 of 12 OpenSSL zero-days** in the January 2026 coordinated release, plus 13 of 14 CVEs assigned in 2025 — **15 total** across both releases
- CVE-2025-15467 (HIGH severity, stack buffer overflow in CMS message parsing, potentially remotely exploitable); vulnerability types include heap overflows, type confusions, NULL dereferences, and a cryptographic bug in OCB mode
- Three of the OpenSSL bugs dated back to 1998-2000, lurking undetected for 25-27 years
- Has been assigned **100+ externally validated CVEs** (including all 12 OpenSSL zero-days) across 30+ projects including Linux kernel, glibc, Chromium, Firefox, WebKit, Apache HTTPd, GnuTLS, OpenVPN, Samba, and NASA CryptoLib
- **Full-loop capability**: discover → validate → patch — in 5 cases, AISLE's AI directly proposed patches accepted into official OpenSSL releases
- **Deep C/C++ memory safety specialization**: finds heap overflows, type confusions, NULL dereferences, use-after-free, and cryptographic bugs that have lurked undetected for decades
- Started hunting in August 2025 — reached these results in under 6 months
- curl cancelled its bug bounty the same week AISLE's OpenSSL results were published — a stark contrast between AI-discovered quality and AI-generated slop

**OpenAI Codex Security (formerly Aardvark):**
- **Launched March 6, 2026** as research preview for ChatGPT Enterprise, Business, and Edu customers via Codex web — free usage for the first month
- Three-stage approach: builds editable threat model, validates issues in sandboxed environments, proposes fixes with full system context
- **92% recall** on benchmark "golden" repositories with synthetically-introduced vulnerabilities
- Performance claims: **84% noise reduction** vs traditional SAST, **90% reduction in over-reported severity**, **50%+ lower false positive rates** compared to traditional scanners
- Scanned **1.2 million commits**, finding **14 CVEs** across major open-source projects (OpenSSH, GnuTLS, GOGS, Chromium, PHP, libssh); also flagged **792 critical** and **10,561 high-severity** issues in the past month
- Open-source maintainers can apply via the **Codex for OSS** program; **$10M in API credits** committed for open-source security
- Best for: continuous monitoring of large codebases; major competitive threat to human hunters on pattern-matching vulns

**Anthropic Claude Code Security:**
- **Launched March 6, 2026** with Mozilla partnership as limited research preview
- Claude Opus 4.6 found **500+ vulnerabilities** in production open-source codebases — bugs that had gone undetected for decades despite expert review
- **Mozilla partnership**: found **22 vulnerabilities in Firefox** (14 high-severity) over two weeks — representing almost a fifth of all high-severity bugs patched in Firefox in 2025; fixes shipped in Firefox 148 (Feb 24); 112 total reports submitted to Mozilla
- Worked in a virtual machine with access to standard utilities and fuzzers; no specific instructions or specialized knowledge — "out-of-the-box" capability
- Key technique: reasoning about code by tracing data flows and reading commit histories to find variants of partially fixed bugs
- PoC exploit generation remains hard: ~$4,000 in API credits for exploit attempts, succeeded in only 2 cases — finding vulns is easier than exploiting them
- **CVEs in Claude Code itself**: CVE-2025-59536 (CVSS 8.7, code injection via tool initialization, fixed v1.0.111) and CVE-2026-21852 (CVSS 5.3, data exfiltration including API keys, fixed v2.0.65)

**Endor Labs AURI (Agentic Security Intelligence):**
- Launched **March 3, 2026** — unified security intelligence for agentic software development
- Embeds security directly into AI coding workflows; verifies security of code produced by different AI agents
- **Free developer tier**: MCP server, CLI, and Skills plugin available at no cost for individual developers
- Study found only **10% of AI-generated code is secure** — AURI addresses this gap
- Identified **7 security vulnerabilities in OpenClaw** (acknowledged by development team)
- Claims **80-95% reduction** in security findings for enterprise customers
- Best for: developers wanting integrated security checks during AI-assisted coding

**ZAST.AI (Zero False Positive Security Scanner):**
- Raised **$6M Pre-A** round led by Hillhouse Capital (February 2026), ~$10M total funding
- Discovered hundreds of zero-day vulnerabilities in 2025, resulting in **119 CVE assignments** across Microsoft Azure SDK, Apache Struts XWork, Alibaba Nacos, Langfuse, WordPress, and others
- Claims "zero false positive" approach; detects both syntax-level vulns (SQLi, XSS, SSRF) and semantic-level vulnerabilities (IDOR, privilege escalation, payment logic flaws)
- Provides runnable PoC vulnerability reports automatically
- Best for: automated source code scanning with minimal false positive noise

**Trend Micro AESIR (AI-Powered Threat Intelligence):**
- Launched mid-2025 by TrendAI; two core components: MIMIR (real-time threat intelligence) and FENRIR (zero-day vulnerability discovery)
- Discovered **21 CVEs** across NVIDIA, Tencent, MLflow, and MCP tooling since mid-2025
- Notable: ZDI-25-1041 (CVE-2025-33183) and ZDI-25-1044 (CVE-2025-33184) — patch bypasses in NVIDIA tools
- Best for: zero-day discovery in AI infrastructure and enterprise tools

**OpenAI o3 (LLM-Assisted Zero-Day Discovery):**
- Security researcher Sean Heelan used o3 to discover **CVE-2025-37899**, a zero-day in the Linux kernel's SMB implementation (use-after-free), analyzing 12,000+ lines of code
- Demonstrates that frontier reasoning models can assist in finding complex kernel-level vulnerabilities when guided by expert researchers

**Assail Ares (Autonomous Pentesting):**
- Launched from stealth in January 2026 with autonomous AI agents for continuous penetration testing across APIs, web, and mobile
- Uses co-evolutionary training: adversary simulators compete against breachers in a continuous loop
- Best for: continuous API/web/mobile security testing

**BlacksmithAI (Multi-Agent Pentesting):**
- Open-source AI pentesting framework (GPL-3.0, March 2, 2026) with multi-agent architecture: orchestrator + specialized sub-agents (recon, scanning, exploitation, post-exploitation)
- Supports OpenRouter, vLLM, custom LLM backends; uses mini-kali Docker container for professional pentesting tools
- Terminal and web interfaces; best for: multi-phase pentesting with agent specialization
- GitHub: `yohannesgk/blacksmith`

**Trail of Bits Claude Code Security Skills (March 2026):**
- Published security-focused Claude Code plugin marketplace with battle-tested audit methodology
- Key skills: audit context building (line-by-line analysis, invariant tracking, cross-function flow tracing), static analysis (CodeQL, Semgrep, SARIF parsing), variant analysis, fix commit verification, security-focused diff review
- Also released `claude-code-config` — opinionated defaults for sandboxing, permissions, hooks, and MCP servers reflecting professional audit workflows
- Based on the Trail of Bits Testing Handbook methodology
- Best for: structured security code audits; complementary to Cohunt's source-code-audit skill
- GitHub: `trailofbits/skills`

**Cisco Skill Scanner (March 2026):**
- Companion to MCP Scanner — static + behavioral analysis for AI agent skill files (OpenClaw, Claude Skills, OpenAI Codex skills)
- Includes VirusTotal integration for malware detection
- Built in response to the ClawHub supply chain crisis (824+ malicious skills)
- Best for: scanning third-party agent skills before installation; defensive auditing
- GitHub: `cisco-ai-defense/skill-scanner`

**AgentShield Benchmark (AI Agent Security Testing):**
- First open benchmark testing 6 commercial AI agent security tools with **537 test cases**
- Key finding: providers catching 95%+ of prompt injections miss most unauthorized tool calls — weak tool abuse detection across the board
- Composite scores range from ~39 to ~98 across tested tools
- Uses **Trustless Benchmark Protocol** with Ed25519 signatures for credibility
- Fully open-source and auditable — useful for evaluating your target's AI security tooling
- GitHub: `doronp/agentshield-benchmark`

**Shift (AI Plugin for Caido):**
- AI plugin for the Caido web security proxy integrating LLMs directly into the proxy UI
- English-command control of Caido, context-aware wordlist generation, AI-powered HTTP request modification
- **Shift Agents**: micro-agent framework that performs proxy actions on behalf of the user (targeted fuzzing, parameter extraction, response analysis)
- Best for: AI-augmented manual pentesting within Caido proxy workflows
- GitHub: `caido-community/shift`

**Aikido Infinite (Self-Securing Software):**
- Industry's first self-securing software: every code change triggers autonomous pentesting agents that discover risk, validate exploitability, generate patches, and retest
- Found **7 CVEs in Coolify** including privilege escalation and full host compromise via RCE as root across 52,000+ exposed instances
- Best for: continuous pre-deployment security validation with automated remediation

**Zen-AI-Pentest:**
- Python-based open-source framework wrapping 20+ established security tools (Nmap, SQLMap, Metasploit, Burp Suite, Gobuster, Nuclei, BloodHound) under an AI-driven orchestration layer (February 2026)
- Uses LLMs to make strategic decisions and adjust testing approaches based on discovered information
- CI/CD integration with GitHub Actions, GitLab CI, and Jenkins
- Best for: automated pentest pipelines combining traditional tools with AI-driven strategy
- GitHub: `SHAdd0WTAka/Zen-Ai-Pentest`

**Penligent (Autonomous Security Agent):**
- Fully autonomous Security Agent with its own runtime environment — unlike PentestGPT (AI copilot for human), Penligent executes commands, analyzes return traffic, and plans next moves autonomously
- Implements military OODA Loop (Observe-Orient-Decide-Act) with feedback loops
- Generates payloads, sends them to targets, analyzes errors, refines payloads, and retries until successful
- Built on Large Action Models (LAMs) that predict state transitions
- Best for: autonomous black-box vulnerability validation and end-to-end pentesting
- Documented workflow: Claude Code Security (white-box) → Penligent (black-box proof of exploitability)

**Aikido Attack (Autonomous Deployment Pentesting):**
- Deploys autonomous agents that pentest every deployment; validates exploitability, generates patches, and retests fixes — all before code hits production
- Integrates into CI/CD pipeline for continuous security validation
- Best for: pre-deployment security testing with automated remediation

**Burp Suite with Burp AI (PortSwigger, 2025-2026):**
- Agentic pentesting assistant integrated into Burp Suite Professional 2025.2+
- **Explore Issue**: autonomously investigates scanner findings, attempts exploits, identifies additional attack vectors — turns passive scan results into active exploitation
- **Explainer**: AI-generated explanations of unfamiliar technologies within Repeater — reduces learning curve for complex targets
- **Broken Access Control false positive reduction**: intelligent filtering before results appear — addresses the #1 pain point in automated scanning
- **AI-powered recorded logins**: automatically generates login sequences for authenticated scanning — eliminates manual session management
- Builds on PortSwigger's existing scanning engine with AI-driven exploration
- Best for: enhancing manual pentesting with AI-suggested attack vectors; automated exploitation of scanner findings

**OWASP ZAP with MCP Integration (ZAP 2.17.0, December 2025):**
- OWASP ZAP integrating with Model Context Protocol for AI-powered web security testing
- AI enhancement for bug bounty hunting workflows — LLMs can drive ZAP scanning and analysis
- ZAP 2.17.0 released December 2025 with foundational MCP support
- Best for: free/open-source AI-augmented web app security testing via MCP

**DeepKeep (AI Agent Attack Surface Scanner, March 2026):**
- Maps enterprise AI agent risk across frameworks: Microsoft, Agentforce, OpenAI Agents, CrewAI, Amazon Bedrock AgentCore, n8n, Make
- Discovers and catalogs AI agent deployments across the organization
- Best for: AI attack surface discovery and risk mapping in multi-framework environments

**NVIDIA Garak (LLM Vulnerability Scanner):**
- Open-source scanner probing LLMs for hallucination, data leakage, prompt injection, misinformation, toxicity, and jailbreaks
- Supports Hugging Face Hub, Replicate, OpenAI API, LiteLLM, and REST-accessible systems
- ~100 attack vectors; AVID integration for community vulnerability sharing
- Best for: systematic LLM vulnerability scanning before deployment or as part of bug bounty testing

**Augustus (Praetorian, LLM Adversarial Testing):**
- Launches over 210 distinct adversarial attacks against 28 LLM providers
- Built-in rate limiting, retry logic, and timeout handling for production-grade testing
- Best for: broad adversarial testing of LLM deployments at scale

**Google Big Sleep (DeepMind + Project Zero):**
- Google's AI-based bug hunter found 20+ security vulnerabilities in open-source software including 5 Safari vulnerabilities (buffer overflows, use-after-free)
- **First AI agent to foil active exploitation in the wild**: discovered CVE-2025-6965 (critical SQLite memory corruption, CVSS 7.2) that was known only to threat actors and at risk of being exploited — Google coordinated patches before widespread attacks
- Can spot complex chaining issues that single-tool SAST misses
- Always verify: AI may hallucinate; always confirm any finding with manual proof-of-concept

### AI-Powered Subdomain Enumeration

**Subwiz (Hadrian):**
- Uses a lightweight transformer model (17.3M parameters, based on nanoGPT) trained on 26 million tokens of subdomain data
- Instead of predicting words, predicts likely subdomain names based on patterns in historical DNS records
- ~1,000x smaller than ChatGPT, runs on a laptop
- Internal testing shows up to 10% increase in discovered subdomains per domain
- Best for: augmenting traditional subdomain enumeration tools (Subfinder, Assetfinder) with AI-predicted candidates

**EnumerAIte (DEF CON 33 Recon Village):**
- AI-assisted web attack surface enumeration: generates high-value paths and subdomains tailored to specific targets
- Prioritizes promising endpoints from massive datasets, reducing noise while highlighting sensitive or admin-like endpoints
- Best for: building context-aware recon assistants that understand application structure

### AI-Assisted Fuzzing & Payload Generation

| Use Case | AI Workflow | Expected Gain |
|----------|-----------|---------------|
| **Wordlist Expansion** | Feed known payloads to AI, ask "generate 50 variations that might bypass WAF" | 3-5x more test cases without manual crafting |
| **Protocol Fuzzing** | Describe the protocol; ask AI to generate malformed packets | Faster discovery of parser bugs |
| **Context-Aware Payloads** | Show AI the tech stack and recent CVEs; ask "what should I test for?" | Targeted payloads vs. spray-and-pray |
| **AI PoC Generation** | Use PoCo (smart contracts) or PoCGen (npm) frameworks for automated exploit generation | PoCGen achieves 77% success rate (432/560 vulns), outperforming prior SOTA by 45 percentage points |

**Warning — "PoC Pollution":** AI-generated PoCs are flooding security repositories with broken exploits. The consistency of professional-looking write-ups suggests machine generation. Always validate any AI-generated PoC before using it — broken exploits waste your time and detection engineers' time.

### The XBOW/Autonomous Agent Reality Check

**XBOW reached #1 on both the US and global HackerOne leaderboards in 2025**, surpassing every human researcher within 90 days of active operation:
- Submitted **1,060 reports in ~90 days** (54 critical, 242 high, 524 medium); cumulative total exceeds **1,400 vulnerability reports** across the full spectrum: RCE, SQLi, XXE, Path Traversal, SSRF, XSS, Cache Poisoning, Secret Exposure
- 132 fixes confirmed by companies, 303 triaged awaiting resolution
- Runs up to **80x faster** than manual teams
- Architecture: "coordinator" performs initial discovery and spawns multiple "solvers" — individual AI pentesters with isolated attack machines
- **Benchmark performance:** handled **75% of standard web security challenges** entirely on its own, plus **85% of custom-built, never-before-seen vulnerabilities** that would stump skilled human researchers
- **Validation approach:** sophisticated automated "validators" — peer reviewers that confirm each vulnerability (e.g., headless browsers verifying JavaScript payloads execute on target sites) — avoiding the false positive problem
- **Raised $75M Series B** (led by Altimeter Capital, with Sequoia Capital and NFDG); **$117M total funding**
- However, XBOW is currently **operating in the red** — compute costs exceed bounty earnings, though costs are expected to drop
- XBOW is fully autonomous but still requires human review pre-submission to comply with HackerOne's policy on automated tools
- **XBOW Pentest On-Demand** launched late 2025 — fully automated pentest service delivering results within 5 business days at **$6,000 per engagement**
- **XBOW Public API** launched February 1, 2026 in Public Preview — programmatic access to start, pause, resume, cancel pentests and retrieve findings; enables running dozens of pentests in parallel; pricing starts at $6K
- Appointed Databricks CRO to board; expanding APAC presence through customer deployments and partnerships in 2026
- A fully local pentesting agent achieved **~78% on XBOW benchmarks** via feedback-driven iteration — demonstrates locally-run agents can approach XBOW-level capability without cloud infrastructure
- HackerOne responded by **splitting leaderboards** to separate individuals from companies/agents like XBOW
- Joel Noguera & Diego Jurado (XBOW founders) presented at DEF CON 2025 showing agents exploiting real-world XSS, JWT, and CSRF bugs autonomously
- **XBOW #1 globally** confirmed at Black Hat 2026 with live demo — ran real-time on HackerOne programs finding vulnerabilities on stage; nominated #6 in Enterprise Tech 30 Early Stage
- **Notable finding**: discovered a previously unknown vulnerability in **Palo Alto GlobalProtect VPN** affecting 2,000+ hosts — demonstrates autonomous agents finding high-impact network infrastructure bugs, not just web vulns
- Architecture detail: built infrastructure on top of XBOW core to **identify high-value targets and prioritize** for maximum return — target selection is as important as exploitation

**Wiz Research AI Agent Benchmarks (2025-2026):**
- Tested Claude Sonnet 4.5, GPT-5, Gemini 2.5 Pro on 10 lab challenges
- Agents were proficient in **directed tasks** (clear target, well-documented surface), but less effective in realistic, unstructured environments
- Without clear success indicators, AI agents **produce false positives, exaggerate severity, and struggle to distinguish meaningful access from noise**
- Best result: Gemini 2.5 Pro discovered developer documentation → found app creation endpoint → used session token → accessed protected endpoint, all in 23 steps

**Where Autonomous Agents Excel:**
- Mass SSRF scanning, subdomain enumeration, simple SQLi/XSS
- Structured exploitation tasks with well-documented attack surfaces
- Code analysis and pattern matching at scale

**Where Humans Still Win:**
- Context-dependent bugs, multi-step business logic flows
- Authentication-gated scenarios requiring human context
- Chaining bugs across different application components
- Social engineering vectors and creative attack paths
- 72% of hackers find more critical vulnerabilities when working in teams (Bugcrowd 2026)

**Your Role:** Use autonomous tools for coverage; manually verify and chain findings to create reportable exploits. By mid-2026, "AI as an accelerated, supervised staff member" is the dominant model in offensive security. Security leaders are moving toward continuous, data-driven exposure management combining human intelligence with automation. Researchers predict that by 2028 most cybersecurity actions will be autonomous, with humans teleoperating.

**March 2026 Competition Escalation:** Both Anthropic (Claude Code Security) and OpenAI (Codex Security) launched enterprise-grade autonomous vulnerability scanning on the same day (March 6, 2026). Codex Security scanned 1.2M commits finding 14 CVEs with 84% noise reduction vs traditional SAST; Claude Code Security found 500+ vulns including 22 Firefox vulns in 2 weeks. These tools will flood programs with findings — human hunters must differentiate with chains, business logic, and agent-specific attack patterns that scanning tools cannot replicate.

**Semgrep's AI Vulnerability Detection Benchmark (2025):**
- Tested Claude Code (Sonnet 4) and OpenAI Codex (o4-mini) against 11 real-world open-source Python projects
- **Claude Code**: found 46 vulnerabilities with 14% true positive rate (86% false positive rate)
- **OpenAI Codex**: found 21 vulnerabilities with 18% true positive rate (82% false positive rate)
- Critical finding: **non-determinism** — running the same prompt on the same codebase multiple times yielded vastly different results (3, 6, then 11 findings across three identical runs)
- Codex showed 0% true positive rate for IDOR, SQL injection, and XSS, but 47% for Path Traversal
- Takeaway: never trust a single AI scan; run multiple passes and manually validate all findings

**Sonar Code Security Study:**
- LLMs produce vulnerable code by default
- GPT-4o: only 10% of outputs free from vulnerabilities with naive prompts, 20% with security prompts
- Claude 3.7-Sonnet: 60% secure code output — best performer, but still needs human review
- Implication: when using AI to generate PoCs or test scripts, review for introduced vulnerabilities

**The 2026 Tool Landscape:**
- **AISLE** for deep C/C++ memory safety vulnerability discovery at scale (100+ CVEs including all 12 OpenSSL zero-days; full-loop: discover → validate → patch)
- **Shannon** for autonomous white-box web app pentesting (96.15% on XBOW benchmark via CPG semantic understanding; full-loop: discover → validate → exploit; subscription $0-$49.99/mo)
- **ZAST.AI** for zero-false-positive source code scanning (119 CVEs, $10M funded)
- **AESIR** (Trend Micro) for zero-day discovery in AI infrastructure (21 CVEs including NVIDIA, MCP tooling)
- **Strix** for autonomous web app testing with browser + proxy + terminal
- **NeuroSploit v3** for autonomous pentesting with exploit chaining and anti-hallucination
- **CAI** for CTF-level challenges and structured offensive tasks (note: CVE-2025-67511 in CAI itself)
- **Hound** for deep source code audits with knowledge-graph reasoning
- **HexStrike AI** for MCP-native tool orchestration across 150+ security tools (now in Kali Linux)
- **PentAGI** for fully autonomous pentesting in sandboxed Docker environments with 20+ tools (Go-based, PostgreSQL + Neo4j)
- **XBOW Pentest On-Demand** for commercial automated pentesting
- **Assail Ares** for continuous API/web/mobile pentesting with co-evolutionary AI agents
- **BlacksmithAI** for multi-agent pentesting lifecycle management (open-source, March 2026)
- **Zen-AI-Pentest** for autonomous agents + standard security utilities (open-source, Feb 2026)
- **Codex Security** (OpenAI, formerly Aardvark) for continuous source code vulnerability monitoring — **launched March 6, 2026**; 1.2M commits scanned, 14 CVEs found, 84% noise reduction vs traditional SAST
- **Claude Code Security** (Anthropic) for deep codebase analysis and variant finding — **launched March 6, 2026** with Mozilla; 500+ vulns found, 22 Firefox vulns in 2 weeks
- **Terra Security** ($38M funded) for agentic-AI continuous pentesting for Fortune 100 companies
- **AWS Security Agent** for multi-agent automated pentesting (preview since re:Invent 2025)
- **Hadrian** for attack-surface-driven 24/7 autonomous pentesting
- **Escape** for API-native DAST with agentic crawling and MCP endpoint discovery (Y Combinator backed)
- **Penligent** for fully autonomous black-box pentesting with OODA loop feedback
- **Aikido Attack** for autonomous pre-deployment security testing with remediation
- **Burp Suite with Burp AI** for AI-augmented manual pentesting (PortSwigger 2025)
- **DeepKeep** for AI agent attack surface discovery across multi-framework environments (March 2026)
- **NVIDIA Garak** for systematic LLM vulnerability scanning (~100 attack vectors)
- **Augustus** (Praetorian) for broad adversarial LLM testing (210+ attacks, 28 providers)
- **Semgrep AI-Powered Detection** (2026) — multimodal AppSec engine combining deterministic SAST analysis with LLM reasoning; detects business logic flaws (IDORs, broken authorization) beyond traditional pattern matching; AI noise filtering removes 1 in 5 false positives at 95% user alignment; "AI-Powered Memories" re-analyzes backlogs when new patterns are added
- **RunSybil** for AI-driven pentesting with coordinated autonomous agents (map, probe, chain exploits); built by OpenAI/Bishop Fox/Rapid7/CrowdStrike alumni; free tier + Pro at $99/mo
- **Shift** (Caido AI plugin) for AI-augmented proxy workflows — English-command control, context-aware wordlists, Shift Agents micro-agent framework
- **Aikido Infinite** for self-securing software — autonomous agents pentest every code change (found 7 CVEs in Coolify including RCE as root)
- **CyberStrikeAI** (THREAT TOOL) — open-source AI-native Go platform integrating 100+ security tools; weaponized by threat actors to breach FortiGate firewalls across 55 countries (Jan-Feb 2026); demonstrates offensive AI tool proliferation and supply chain risks; do NOT use offensively — listed here for awareness of competitive threat landscape
- **SecureClaw** (Adversa AI, February 2026) — first OWASP-aligned open-source security plugin for OpenClaw agents; 55 automated audit/hardening checks mapped to OWASP Agentic Top 10, MITRE ATLAS, and CoSAI frameworks; useful for auditing OpenClaw deployments before testing
- **AgentGuard** — runtime guard for AI agents blocking malicious skills, data leaks, and secret exposure; sub-millisecond latency local PolicyEngine with tamper-evident audit logs; compliance evidence for EU AI Act and SOC 2
- **Ironclaw** (February 2026) — OpenClaw-inspired Rust agent runtime with defense-in-depth (sandboxing, secret boundaries); open-source alternative focused on security-first agent execution
- **Nono** (February 2026) — kernel-enforced sandbox CLI and SDK to isolate AI agents with capability controls and safe secret injection; granular permission model for agent containment
- **Clawsec** (February 2026) — security skill suite for OpenClaw and NanoClaw agents with automated audits, drift detection, and integrity checks; open-source agent hardening toolkit
- **Novee Security** ($51.5M funded, January 2026) — AI penetration testing platform with proprietary offensive AI model; founded by Unit 8200/Talpiot veterans; outperformed frontier LLMs (Gemini 2.5 Pro, Claude 4 Sonnet) by over **55%** on constrained web exploitation challenges, achieving **up to 90% accuracy** where general-purpose models plateau at ~65%; transitions red teaming from scheduled events to continuous operational pressure
- **Maze** ($31M funded) — AI agents for vulnerability management modeling analyst workflows; deploys thousands of agents to investigate cloud data, proving 80-90% of findings are false positives and identifying exploitable ones
- **Enkrypt AI MCP Scan + Secure MCP Gateway** — automated MCP scanner with CVSS severity scores and line-level references; **scanned 1,000+ MCP servers: 33% had critical vulnerabilities**, averaging 5.2 vulns each; worst case: K8s MCP server with 26 vulns (6 CVSS 9.8); detected malicious Postmark MCP silently exfiltrating emails; open-source Secure MCP Gateway provides real-time AI safety filters; free assessments at enkryptai.com/mcp-scan; GitHub: `enkryptai/secure-mcp-gateway`
- **Pynt MCP Security Research** (March 2026) — quantified MCP exploit probability across 281 real-world configurations; **10 plugins = 92% exploit probability**; research adopted by VentureBeat as industry benchmark; free report at pynt.io/resources-hub/mcp-security-research-2025
- **Noma AI Red Teaming** (March 2026) — intelligent red team agent that dynamically builds attacks based on target analysis; aligned with OWASP Agentic Top 10, MITRE ATLAS, CoSAI; Noma Labs continuously updates attack library covering RAG exploitation, memory manipulation, tool misuse, MCP vulnerabilities; comprehensive AI-SPM + Red Teaming platform
- **MCPScan.ai** — platform with advanced Tool Metadata Scanner using specialized LLM classifier to detect poisoning attempts across MCP tool definitions
- **MCPTox Benchmark** — first benchmark for evaluating LLM agent vulnerability to tool poisoning on real-world MCP servers; 45 live MCP servers, 353 tools, 1,312 malicious test cases across 10 risk categories; attack success rates exceed 60% on GPT-4o-mini, o1-mini, DeepSeek-R1, Phi-4; GitHub: `zhiqiangwang4/MCPTox-Benchmark`
- **Palo Alto Networks Cortex AgentiX** — embeds agentic AI across the security platform from code to cloud to SOC; redefines SOC operations with agentic response
- **Cecuro AI** (DeFi Security Agent) — purpose-built AI agent for smart contract vulnerability detection; detected vulnerabilities in **92% of 90 exploited DeFi contracts** ($96.8B exploit value) vs 34% for baseline GPT-5.1; performance gap from domain-specific security methodology, not model capability; best for: DeFi/Web3 bug bounty programs
- **Hacktron AI** (Bug Bounty Research Team) — AI-powered research team earning bounties across AI IDEs; $10,000 for Google Antigravity RCE; CVE-2026-1731 (CVSS 9.9) pre-auth RCE in BeyondTrust Remote Support; findings in Cursor, Windsurf, Perplexity Comet, OpenAI Atlas; demonstrates AI-augmented bug bounty team model — currently in private beta
- **Cisco MCP Scanner v4.0.1** (Open Source, March 2026) — three scanning engines: YARA static analysis, LLM-as-judge semantic evaluation, and Cisco AI Defense integration; contextual/semantic analysis of tool definitions and implementations; partnership with AWS for MCP registry/gateway integration; also covers A2A protocol and agentic skill scanning; GitHub: `cisco-ai-defense/mcp-scanner`
- **Snyk Agent Scan** (February 2026) — AI agent, MCP server, and Claude Skills security scanner; detects prompt injection, tool poisoning, tool shadowing, toxic flows, rug pull attacks, malware payloads, credential handling issues, and hardcoded secrets; CLI scan + background MDM mode; **Skill Inspector free self-service website launched Feb 13, 2026** — auto-discovers Claude Code/Desktop, Cursor, Gemini CLI, and Windsurf configurations; GitHub: `snyk/agent-scan`
- **AgentAudit** (ecap0, March 2026) — MCP server security registry using multi-agent consensus-based auditing; **270+ packages** audited with **247 vulnerabilities** found (up from 194 packages in early March); discovered Schema Drift and Context Pivoting attack vectors; covers npm packages, pip packages, and AgentSkills; registry-wide average Trust Score 98/100 but 14 critical/high findings represent real exploitable vulnerabilities; agentaudit.dev
- **PentAGI** (2026) — fully autonomous AI agent pentesting platform integrating 20+ security tools; multi-agent system (researcher, developer, executor roles); uses Graphiti temporal knowledge graph (Neo4j) for semantic understanding; runs in sandboxed Docker; supports OpenAI, Claude, Gemini, and local Ollama models; GitHub: `vxcontrol/pentagi`
- **Penetrify** (March 3, 2026) — world's first fully autonomous AI red team for continuous security testing; CI/CD integration; starting at $50/mo; Czech Republic startup
- **GHOSTCREW** (January 2026) — open-source MCP-based AI red team toolkit integrating Metasploit, Nmap, SQLMap; 450+ GitHub stars; built on Model Context Protocol for tool orchestration; GitHub: `ghostcrew/ghostcrew`
- **OpenAnt** (Knostic, March 6 2026) — open-source LLM vulnerability scanner with two-stage detect + exploit pipeline; supports 6 languages; Apache 2.0 license
- **Jailbreak Foundry (JBF)** (arXiv:2602.24009, updated March 5 2026) — multi-agent system that translates academic jailbreak papers into executable attack modules; bridges gap between research and practical exploitation; useful for rapidly testing AI targets against latest published attacks
- **QUICker** (ScienceDirect 2026) — first race condition exploitation tool for HTTP/3; extends attack surface beyond HTTP/1.1 and HTTP/2; fills testing gap for QUIC-based targets
- **Simbian AI Pentest Agent** (February 2026) — context-aware autonomous pentesting with LRQA partnership; business context integration and transparency traces for audit compliance
- **Contrast Security MCP Server** — bridges IAST vulnerability data with AI agents for context-aware remediation; provides complete taint-flow trace (source to sink), specific HTTP requests that triggered findings, and environment details; AI generates precise fixes; **security note:** only use with services that prohibit model training on inputs (data isolation); GitHub: `Contrast-Security-OSS/mcp-contrast`
- **OpenAnt** (Knostic, March 6 2026) — open-source LLM vulnerability scanner with two-stage detect + exploit pipeline; 6 language support; Apache 2.0 license; best for: systematic LLM security testing with automated exploit validation
