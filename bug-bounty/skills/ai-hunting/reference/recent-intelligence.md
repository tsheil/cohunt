# Recent Intelligence & Incident Analysis

Temporal security intelligence: MCP ecosystem risk quantification, platform-specific incident analysis, and monthly incident digests. This file captures rapidly-evolving context that informs hunt planning.

> **Related files:** [ai-case-studies.md](ai-case-studies.md) for core incident table, platform policies, tools, and CTF reference | [market-context.md](market-context.md) for market statistics and tool intelligence

---

## Table of Contents

- [MCP Ecosystem Risk Quantification](#mcp-ecosystem-risk-quantification)
- [Platform & Infrastructure Incident Analysis](#platform--infrastructure-incident-analysis)
- [March 2026 Incident Digest](#march-2026-incident-digest)

---

## MCP Ecosystem Risk Quantification

Two major studies quantified MCP risk at scale:

**Pynt Research (281 configurations):** 10 MCP plugins = 92% exploit probability. Live exploits observed: HTML→shell execution, crafted emails→code interpreter→RCE, Slack messages→terminal commands. 72% of MCPs expose sensitive capabilities, 13% accept untrusted inputs.

**Enkrypt AI (1,000+ servers):** 33% had critical vulnerabilities, averaging 5.2 per server. K8s MCP server: 26 vulns (6 CVSS 9.8). Malicious Postmark MCP server silently exfiltrated every processed email.

**ChatGPT MCP Integration:** ChatGPT can now connect to any remote MCP server, triggering tools autonomously. Malicious MCP servers are the most immediate threat — affects all ChatGPT Enterprise/Business users.

**Knostic Exposure Scan:** 1,862 MCP servers found on public internet with zero authentication; every server tested responded without credentials.

**Noma x OWASP Red Team Playbook:** Comprehensive agentic AI red team framework aligned with OWASP; dynamic attack generation covering prompt injection, tool misuse, excessive agency, multi-agent coordination failures; continuously updated by Noma Labs.

**Bug Bounty Relevance:** MCP plugin stacks are now quantifiably the most dangerous enterprise AI attack surface. Target companies with 5+ MCP plugins for near-certain exploitation paths. Cross-plugin chains (untrusted input → sensitive capability) are the highest-value findings.

**Krebs on Security: OpenClaw Admin Exposure (March 8, 2026):**
OpenClaw web admin interfaces found exposed to the internet, revealing full credential configurations including API keys, OAuth secrets, and signing keys. Combined with the ClawHavoc supply chain attack (1,184 malicious skills) and 42,665 exposed instances, this confirms OpenClaw deployments as a consistently high-value attack surface. Pattern: scan for exposed admin interfaces → credential harvest → lateral movement to connected services.

**Apple iOS Zero-Day Targeted Attack (March 2026):**
CVE-2026-20700 (CVSS 7.8) — memory corruption in `dyld` (Dynamic Link Editor) used in "extremely sophisticated attack against specific targeted individuals." Discovered by Google Threat Intelligence Group. Part of exploit chain with CVE-2025-14174 and CVE-2025-43529 (WebKit). Patched in iOS 26.3 and macOS Tahoe 26.3. Demonstrates continued nation-state interest in mobile zero-days — Apple's $2M bounty for zero-click remote exploits reflects the threat level.

**February 2026 Windows MotW Bypass Cluster:**
Three separate Mark-of-the-Web bypasses actively exploited in February 2026 Patch Tuesday — CVE-2026-21510 (Windows Shell), CVE-2026-21513 (MSHTML, attributed to APT28), CVE-2026-21514 (Microsoft Word). This concentration indicates MotW bypass is a hot vulnerability class being actively researched by both APT groups and independent researchers. MotW bypass defeats SmartScreen, Protected View, and macro-blocking — the primary trust mechanisms for files downloaded from the internet.

---

## Platform & Infrastructure Incident Analysis

**PleaseFix Agentic Browser Hijack (March 2026):**
Zenity Labs disclosed PleaseFix — a zero-click agent hijack family affecting Perplexity Comet (applicable to all agentic browsers). A calendar invite triggers autonomous agent execution that exfiltrates local files and steals 1Password credentials. No exploit, no user clicks, no explicit request for sensitive actions. Demonstrates that routine data sources (calendar, email) can weaponize AI browsing agents.

**Chrome CSS Zero-Day CVE-2026-2441 (February 2026):**
Actively exploited CSS zero-day using `@property` + `paint()` worklet to trigger use-after-free in Chrome. Patched February 13. Demonstrates browser rendering engine as a live attack surface.

**Chrome Sandbox Escape CVE-2026-3545 (March 2026):**
Critical (CVSS 9.6) sandbox escape via insufficient data validation in Chrome's Navigation component. High-value pattern for variant analysis.

**FIRST CVE Forecast 2026:**
Median **~59,427 CVEs** predicted for 2026 (90% CI: 30,012–117,673). First year to exceed 50,000. Realistic upper-bound scenarios reach 70,000–100,000. Bug bounty hunters face increasing competition from automated tools scanning these at scale.

**Android March 2026 Bulletin:**
129 vulnerabilities patched including critical RCE CVE-2026-0006 (CVSS 9.8, heap buffer overflow, no user interaction) and actively exploited Qualcomm zero-day CVE-2026-21385 (234 chipsets).

**OpenAI Lockdown Mode (February 2026):**
OpenAI launched Lockdown Mode for ChatGPT and acknowledged prompt injection in AI browsers "may never be fully patched." Adaptive attacks bypass 12 recent defenses with 90%+ success using gradient descent, RL, random search, and human-guided exploration.

**Fortinet FortiOS SSO Auth Bypass CVE-2026-24858 (January 2026):**
Critical (CVSS 9.4) auth bypass in FortiCloud SSO — attacker with one FortiCloud account accessed devices registered to other accounts. Actively exploited from Jan 20 (new local admin accounts created on victim FortiGate firewalls). CISA KEV Jan 28. Affects FortiOS, FortiManager, FortiAnalyzer, FortiProxy, FortiWeb. Pattern: SSO trust model abuse across multi-tenant cloud management platforms.

**n8n Ni8mare CVE-2026-21858 (January 2026):**
CVSS 10.0 unauthenticated RCE in n8n workflow platform (~100K servers globally). Content-type bypass → `req.body.files` override → credential extraction → admin session forging → arbitrary workflow execution. Second critical n8n CVE (CVE-2026-25049, CVSS 9.9) enables auth'd RCE via crafted expressions. Workflow automation platforms are goldmines.

**Moltbook AI Social Network Breach (February 2026):**
Wiz discovered misconfigured Supabase database exposing 1.5M API keys (OpenAI, Anthropic, AWS, GitHub, Google Cloud), 35K emails, and private agent messages. Root cause: missing Row Level Security + vibe-coding development practices. Secured within hours. Pattern: AI startups using vibe coding consistently ship critical misconfigurations.

**Vibe-Coded App Epidemic (2026):**
Escape.tech scanned 5,600 vibe-coded apps: 2,000+ vulnerabilities, 400+ exposed secrets, 175 PII instances. 10% of Lovable platform apps had open databases. Tea dating app exposed 72K images including 13K government IDs. AI agents commonly suggest `USING (true)` RLS policies. New systematically vulnerable target class for bug bounty.

**69 Vulnerabilities in 5 AI Coding Tools (March 2026):**
Systematic security audit of Claude Code, Codex, Cursor, Replit, and Devin across 15 test applications found 69 vulnerabilities. SSRF was present in every single tool — all five introduced SSRF in URL preview features, enabling internal URL access and credential leakage. Pattern: AI coding agents consistently fail to implement server-side URL validation, making SSRF a universal defect class in AI-generated code. Hunting heuristic: any vibe-coded app with URL preview, link unfurling, or OG tag fetching is likely SSRF-vulnerable.

**Base44 Critical Auth Bypass (Wiz, March 2026):**
Wiz found undocumented registration and email verification endpoints that accepted only the non-secret `app_id` value, granting full access to private enterprise applications. Additionally, the platform passed the same JWT used for the main Base44 account directly to user-created apps via URL — any hosted app could steal platform user credentials. Fixed within 24 hours. Pattern: SaaS builder platforms that host user-created apps share dangerous trust boundaries — test JWT/session sharing between platform and hosted apps.

**Wiz Cyber Model Arena (March 2026):**
257-challenge offensive security benchmark tested 25 agent-model combos. Claude Code + Opus 4.6 ranked #1; Gemini 3 Pro second. AI agents solved 9/10 directed challenges at under $50 total cost but failed when requiring external source searches or creative pivots. In one test, AI made ~500 tool calls in ~1 hour without finding the issue; human found it in ~5 minutes. Key conclusion: human direction + AI execution is the winning model.

**XBOW Mission Completed (March 2026):**
XBOW declared HackerOne primary mission concluded after reaching #1 globally with 1,060+ reports. Pivoting to pre-production scanning for enterprise customers. Raised $75M Series B ($117M total) but still operating at a loss. HackerOne split leaderboards in response.

**Microsoft "In Scope by Default" (December 2025, effective immediately):**
All Microsoft online services now in scope for bug bounty including third-party and open-source code. New services in scope from day one. Up to $100K for third-party/OSS vulns impacting Microsoft services. $17M paid out in 2025. Largest single expansion of a major bug bounty program.

**Google AI VRP Launch (2026):**
Dedicated AI Vulnerability Reward Program covering Search, Gemini Apps, Gmail, Drive, Sheets, Calendar — up to $30K per finding. Simplifies reporting process for AI-specific issues.

**Claude Code Security Launch (February 20, 2026):**
Anthropic's limited research preview using Claude Opus 4.6 found 500+ vulnerabilities in production open-source codebases — some undetected for decades. Focus: memory corruption, injection, auth bypass, complex logic errors. Open-source GitHub Action for automated PR security scanning. Codex Security launched March 6 — creates "AI code security race." Human-in-the-loop approach distinguishes from fully autonomous tools.

**CyberStrikeAI FortiGate Campaign (Jan-Feb 2026):**
AI-native Go platform with 100+ security tools adopted by threat actors to compromise 500+ FortiGate devices across 55 countries in 5 weeks. 21 unique IPs used the tool. No zero-days needed — just exposed management ports and weak authentication with AI-generated attack plans. Linked to Chinese developer alias "Ed1s0nZ." First major AI-orchestrated mass exploitation campaign. Dual-use concern: tools built for defense weaponized at scale.

**SolarWinds WHD Multi-Stage RCE (CVE-2025-40551, Feb 2026):**
Unauthenticated RCE (CVSS 9.8) exploited as initial access, then lateral movement via Zoho Meetings and Cloudflare tunnels for persistence, plus Velociraptor for C2. Microsoft documented the full attack lifecycle. Pattern: management platform → initial access → legitimate cloud services for persistence → dedicated C2.

**Supply Chain Package Compromise Wave (Feb 2026):**
StripeApi.Net malicious package (Feb 16) masquerading as legitimate Stripe.net NuGet library. Cline CLI NPM compromise (Feb 17) via stolen publish token — postinstall script installing OpenClaw on developer machines. Group-IB finding: 68% of severe incidents now linked to supply chains, nearly double from prior years.

**Semgrep Secure 2026 (February 25):**
First multimodal AppSec engine combining deterministic SAST with LLM reasoning for zero false positives. Signals industry shift from rule-only to hybrid analysis. Major implications for code audit skill — AI-augmented static analysis becoming standard.

**Equixly MCP Pentesting (EUR10M Series A):**
First autonomous penetration testing tool specifically targeting MCP servers via discovery mechanism enumeration. Uses reinforcement learning. Addresses the 30 MCP CVEs in 60 days surge and 38% of servers lacking authentication.

---

## March 2026 Incident Digest

- **ShinyHunters Salesforce Aura campaign** (March 8-10, 2026): UNC6240 breached ~400 organizations via misconfigured Salesforce Experience Cloud guest user profiles; custom AuraInspector tool mass-scans `/s/sfsites/aura` endpoints; `sortBy` bypass circumvents 2,000-record limit; stolen data used for follow-on vishing campaigns; no CVE (Salesforce classifies as customer misconfiguration). **Hunting heuristic:** Salesforce Experience Cloud sites with `aura:invalidSession` responses are testable; guest user "API Enabled" permission is the root cause; use AuraInspector (Mandiant) or Misconfig Mapper (Intigriti) for systematic testing; see auth-testing skill for full test procedures
- **LexisNexis cloud breach** (March 4, 2026): 2GB exfiltrated via vulnerable application; 21K enterprise customer accounts + 400K user profiles + complete VPC infrastructure map exposed; impacts law firms and government agencies. **Hunting heuristic:** legal/GRC SaaS platforms often have weaker AppSec than consumer apps — large data stores with sensitive data make high-value targets
- **Bugcrowd CNA status** (March 2026): recognized as CVE Numbering Authority — can now assign CVEs directly to disclosed vulnerabilities, streamlining researcher disclosure workflow
- **GitHub Taskflow Agent** (March 6, 2026): open-source YAML taskflow-based security audit framework; 80+ vulnerabilities in 40 repos (~20 publicly disclosed) including **CVE-2026-28514** (Rocket.Chat, CVSS 9.3 — missing `await` on async `bcrypt.compare()` → any password accepted); IDOR/access control = largest confirmed count (38); business logic only 25% confirmed. **Hunting heuristic:** structured multi-step audits trace async control flow across service boundaries — un-awaited async crypto/auth is an under-tested bypass class; human hunters need deeper reasoning chains to stay ahead
- **AgentShield benchmark** (March 2026): 6 commercial AI security tools tested across 537 cases — tools catching 95%+ prompt injections **missed most unauthorized tool calls**. **Hunting heuristic:** focus MCP/agent testing on tool abuse and unauthorized execution, not just prompt injection
- **RSAC 2026 Innovation Sandbox** (March 23, 2026): top 10 finalists each receive $5M; security-relevant: ZeroPath (AI-native SAST for business logic), Crash Override (CI/CD build provenance, SLSA Level-2), Charm Security (agentic AI scam prevention), Token Security (machine identity)
- **Anthropic prompt injection defense research** (March 2026): RL-trained Claude reduces successful attacks to 1% in browser ops; but joint OpenAI/Anthropic/DeepMind research showed **all 12 published defenses bypassed at 90%+ under adaptive attack**. **Hunting heuristic:** defense-in-depth claims are overstated — adaptive multi-step injection still works against all deployed defenses
- **AISLE 12/12 OpenSSL zero-days** (Jan 2026): AI system found ALL 12 zero-day vulnerabilities in OpenSSL's January patch release, plus 3 earlier CVEs (15 total). CVE-2026-22796 predates OpenSSL itself — inherited from SSLeay (1990s), a 27+ year old bug. In 5 cases, AISLE directly proposed the accepted patches. **Hunting heuristic:** AI-driven deep source review is now competitive with manual auditing for crypto/parser code; look for legacy inherited code paths
- **Claude Opus 4.6 + Mozilla Firefox** (March 6, 2026): 22 vulnerabilities (14 high-severity) found in Firefox in 14 days across ~6,000 C++ files; 112 total crash reports submitted. PoC exploit generation succeeded in only 2 cases (~$4K API credits). Fixes in Firefox 148. Represents nearly a fifth of all high-severity bugs patched in 2025. **Hunting heuristic:** AI excels at finding bugs in large codebases, but exploit generation remains human territory — focus variant hunting on disclosed AI-found bugs
- **FreeScout Zero-Width Space RCE** (CVE-2026-28289, CVSS 10.0, March 2026): zero-click RCE via email attachment with U+200B in filename; bypasses extension validation but stripped at storage → webshell. **Hunting heuristic:** generalized filename canonicalization + TOCTOU pattern applies to all file upload systems; test invisible Unicode chars in filenames
- **SSRF cluster wave** (March 2026): 5 SSRFs disclosed in 1 week across Soft Serve Git, Ghostfolio, Wallos, Plane, PinchTab. All shared same root cause: validators blocking loopback but not RFC 1918 ranges. **Hunting heuristic:** webhook, notification tester, and import-via-URL endpoints are systematically undertested for private IP SSRF
- **Adversa AI MCP digest** (March 2026): 30 MCP CVEs in 60 days; 38% of 500+ scanned servers lack authentication; working PoC published for external prompt injection, tool prompt injection, cross-tool hijacking. **Hunting heuristic:** MCP ecosystem vulnerability rate is accelerating, not plateauing
- **Claude DXT zero-click RCE** (LayerX, February 9, 2026): CVSS 10.0 zero-click RCE affecting 50+ Claude Desktop Extensions and 10,000+ users. Attack: malicious Google Calendar invite triggers silent system compromise. Root cause: DXT runs unsandboxed with full system privileges; MCP treats low-risk data sources (calendar) as trusted input for high-risk actions (code execution). Anthropic declined fix, stating "outside current threat model." **Hunting heuristic:** any AI integration that bridges low-privilege data sources to high-privilege executors is vulnerable to confused deputy attacks; calendar/email/notification → code execution chains are systematic across all MCP/DXT platforms
- **Apache Tika SSRF/XXE** (CVE-2025-66516, CVSS 10.0): unauthenticated remote SSRF/XXE in Apache Tika document processing. Primary ransomware target. **Hunting heuristic:** document processing libraries (Tika, LibreOffice, Ghostscript) remain high-value SSRF/XXE entry points; test any feature that processes uploaded documents server-side
- **Flowise multi-tenant IDOR** (2026): critical IDOR in PUT `/api/v1/loginmethod` allows any low-privilege user to overwrite SSO configuration of any organization. No ownership or admin validation on `organizationId`. **Hunting heuristic:** AI/ML platform admin endpoints consistently lack tenant isolation; test all organization-level settings endpoints with cross-tenant tokens
- **Microsoft recommendation poisoning** (March 2026): widespread poisoning of AI recommendation systems documented at scale. **Hunting heuristic:** RAG and recommendation systems can be manipulated through data poisoning; test by injecting biased content into retrieval sources and measuring output drift
- **Anthropic distillation attacks exposed** (March 2026): large-scale model distillation attacks documented. **Hunting heuristic:** model theft via systematic query-response collection is an underreported vulnerability class for hosted AI services; test rate limiting and output diversity controls
- **Check Point Claude Code RCE** (CVE-2025-59536/CVE-2026-21852, March 2026): hooks-based auto-RCE + API key exfiltration via `ANTHROPIC_BASE_URL` env var redirection. Attack: malicious repo with `.claude/` configs executes arbitrary code on clone + trust. Hooks run pre/post tool invocations; env vars redirect API calls to attacker-controlled endpoint capturing auth headers. **Hunting heuristic:** any AI tool processing project-level config files (hooks, env overrides, MCP configs) before explicit trust is a supply chain RCE vector; test pre-trust-window exploitation across all AI IDEs
- **mcp-remote CVE-2025-6514** (March 2026, CVSS 9.6): first real MCP client infrastructure RCE — OS command injection via malicious `authorization_endpoint` URL in MCP remote proxy client; 437K+ npm downloads. Malicious MCP server returns crafted OAuth metadata → client passes unsanitized URL to shell → arbitrary command execution. **Hunting heuristic:** MCP infrastructure components (proxies, gateways, bridges) that process server-provided URLs are systematically vulnerable to injection; test all MCP client-side URL handling
- **Anthropic Code Review launch** (March 9, 2026): agentic multi-step reasoning loop for security research — builds on Claude Code Security (500+ vulns, 22 Firefox CVEs). Combined with same-week Codex Security launch creates "AI code security race." **Hunting heuristic:** AI code review tools create false sense of completeness; focus human effort on multi-step business logic chains and auth-gated flows that automated review misses
- **SecureClaw launch** (Adversa AI, February 2026): first open-source OWASP-aligned security plugin for OpenClaw agents; 55 automated audit checks. **Hunting heuristic:** defensive tools create new audit baseline — findings that SecureClaw would catch are now lower-value; look for gaps in its 55-check coverage
- **XBOW Palo Alto GlobalProtect** (March 2026): discovered vulnerability in Palo Alto GlobalProtect VPN appliance. **Hunting heuristic:** AI agents expanding beyond web apps into network security appliances — enterprise perimeter devices becoming AI-discoverable attack surface
- **Cisco SD-WAN exploitation** (March 2026): CVE-2026-20127 (CVSS 10.0) exploited by UAT-8616 since 2023; largest spike March 4; chained with CVE-2022-20775 for root via downgrade-then-restore. CISA Emergency Directive ED 26-03. **Hunting heuristic:** SD-WAN peering endpoints are underexplored attack surface — management-plane auth bypass yields fabric-wide access
- **Cisco FMC dual CVSS 10.0** (March 2026): 48 firewall vulnerabilities patched; CVE-2026-20079 (boot-time auth bypass) + CVE-2026-20131 (Java deser to root). Both pre-auth, low complexity. **Hunting heuristic:** firewall management consoles rarely updated in enterprise — deser RCE on management interfaces is consistently CVSS 10.0
- **n8n Ni8mare 100K servers** (March 2026): CVE-2026-21858 (CVSS 10.0) unauth RCE via Content-Type confusion in Form Webhooks; 8 CVEs in Jan-Feb 2026 across n8n. **Hunting heuristic:** workflow automation platforms with public webhook endpoints are systematically vulnerable — test all webhook endpoints for Content-Type confusion
- **Coruna iOS exploit kit** (March 2026): 23 exploits across 5 chains targeting iOS 13-17.2.1; first mass exploitation of mobile phones by criminal groups using nation-state tools. Used by Chinese scam ops and Russian espionage. CISA added 3 related CVEs. **Hunting heuristic:** mobile exploit proliferation makes mobile-first programs higher-value targets
- **Ivanti EPM magic number** (March 9, 2026): CVE-2026-1603 auth bypass via sending specific numeric value; CISA KEV. **Hunting heuristic:** "magic number" hardcoded auth patterns remain common in enterprise management software — test with common default/backdoor values
- **Autonomous jailbreak agents** (Nature Communications, March 2026): 97.14% success rate across 9 target models using DeepSeek-R1/Gemini 2.5 Flash/Grok 3 Mini as autonomous adversaries in multi-turn conversations. **Hunting heuristic:** multi-turn safety testing far more effective than single-shot — AI safety evaluations using only single prompts dramatically underestimate risk
- **Phantom agent hijacking** (arXiv:2602.16958, Feb 2026): Template Autoencoder (TAE) exploits chat template tokens for role confusion across 7 closed-source agents; **79.76% ASR** on GPT-4.1, Gemini-3; **70+ confirmed vendor vulnerabilities**. Structural/token-level attack that bypasses instruction-tuning. **Hunting heuristic:** test chat template boundary exploitation — inject system/assistant role markers within user messages; structural attacks are higher-yield than prompt-level jailbreaks
- **Sage ADR launch** (Gen Digital/Avast, March 2026): first Agent Detection & Response tool with 200+ detection rules for Claude Code, Cursor, OpenClaw; intercepts tool calls pre-execution; 1,000+ installs. **Hunting heuristic:** commodity agent attacks are now detectable — differentiate human hunting with logic-level agent abuse and multi-step chains that rule-based ADR misses
- **SACR UADP framework** (March 2026): first analyst framework for Unified Agentic Defense Platforms; SentinelOne named leader. **Hunting heuristic:** enterprise agentic defense category maturing — buyer-driven demand means agent security testing will become standard scope on bug bounty programs
- **Microsoft March Patch Tuesday** (March 2026): 67 vulns, 4 actively exploited zero-days (kernel UAF→SYSTEM, NTFS heap read via USB, MMC exploitation); all added to CISA KEV. VHD + MSC delivery vectors common in phishing. **Hunting heuristic:** MotW bypass chains continue to be high-value — test VHD/MSC/ISO combinations that bypass Mark-of-the-Web for Windows targets
- **Google Big Sleep foils planned exploitation** (March 2026): first documented case of AI directly preventing an active exploitation attempt — Big Sleep (Project Zero + DeepMind) identified a critical SQLite vulnerability known only to threat actors. **Hunting heuristic:** AI defensive tools are catching zero-days before exploitation; offensive AI tools finding same-class bugs at 80x speed — pure pattern-matching hunting ROI declining
- **Aikido Infinite launch** (February 24, 2026): continuous AI pentesting on every code change; validates exploitability, generates patches, retests before production. Found 7 CVEs in Coolify including RCE as root. **Hunting heuristic:** targets using Aikido have pre-production testing — focus on logic and auth bugs that survive automated scanning
- **vLLM RCE via torch.load()** (CVE-2025-62164, Critical): AI inference engine deserializes user-supplied Base64-encoded prompt embeddings via `torch.load()` without `weights_only=True`. Attacker-controlled Python execution at server startup. **Hunting heuristic:** AI/ML platforms using PyTorch model loading are systematically vulnerable to deserialization RCE — grep for `torch.load(` without `weights_only=True`
- **Salesforce bounty record** (March 2026): $6.2M paid in 2025 (program's most productive year), 696 hackers, payouts up to $60K individual. **Hunting heuristic:** mature enterprise programs paying more per finding as AI handles low-hanging fruit — focus on Salesforce-specific business logic (Apex, Flow, Guest User access, SOQL injection)
- **SAP March 2026 Patch Tuesday** (March 10, 2026): 15 security notes — 2 Critical: CVSS 9.8 code injection via Log4j 1.2.17 in SAP Quotation Management Insurance, CVSS 9.1 insecure deserialization in SAP Enterprise Portal Administration; 1 High DoS in Supply Chain Management. **Hunting heuristic:** SAP systems using Log4j 1.x are systematically vulnerable; test SAP environments for Log4Shell variant exploitation on unpatched Log4j 1.2.x (different exploit path from Log4j 2.x CVE-2021-44228); SAP Enterprise Portal deserialization targets are high-value in enterprise bug bounty programs
- **Claude Code security fixes** (March 2026): three fixes — (1) nested skill discovery loaded skills from gitignored directories (e.g., `node_modules`) creating supply chain risk, (2) trust dialog silently enabled all `.mcp.json` servers on first run without individual consent, (3) macOS keychain corruption with multiple OAuth MCP servers. **Hunting heuristic:** all three patterns are generalizable — test other AI IDEs for (1) skill/plugin loading from untrusted directories, (2) bulk trust grants without per-item consent, (3) credential storage race conditions with multiple OAuth providers
- **PromptArmor Claude Code plugin hook injection** (March 2026): marketplace plugin uses `PostToolUse` hook with exit code 2 to inject arbitrary prompts via stderr — Claude treats as system instruction, follows commands including file reads and subagent spawning. Demonstrated full codebase exfiltration. **Hunting heuristic:** any AI tool with a plugin/extension hook system that feeds back into the model context is vulnerable — test stderr/error channels as prompt injection vectors, check if the tool attributes the source of feedback messages
- **OneUptime Synthetic Monitor RCE** (CVE-2026-30957, CVSS 9.9, March 10, 2026): Node.js `vm` sandbox escape via exposed Playwright `browser`/`page` objects in Synthetic Monitors; any project user can execute OS commands on oneuptime-probe. Fixed v10.0.21. **Hunting heuristic:** any platform exposing capability-bearing host objects (Playwright, Puppeteer, `child_process`) into `vm` sandboxes has the same escape — test custom code/script execution features in monitoring, CI/CD, and workflow platforms
- **Nginx-UI unauth backup** (CVE-2026-27944, CVSS 9.8, March 5, 2026): `/api/backup` returns full system backup without auth + AES-256 key in `X-Backup-Security` header → admin creds, TLS private keys, session tokens. **Hunting heuristic:** management UI backup/export endpoints are systematically under-protected — test `/api/backup`, `/export`, `/download`, `/settings/export` on any admin panel (cPanel, pfSense, Webmin, Proxmox, self-hosted tools)
- **WordPress Greenshift AJAX authz bypass** (CVE-2026-2371, March 6, 2026): `gspb_el_reusable_load()` handler missing `current_user_can()` → unauthenticated leak of private/draft reusable blocks (100K+ installs). **Hunting heuristic:** WordPress plugin AJAX handlers registered with `wp_ajax_nopriv_*` frequently miss capability checks — `grep -r 'wp_ajax_nopriv_' wp-content/plugins/` then test each handler without auth; nonce exposure on public pages via shortcodes is a common secondary weakness
- **LeakyLooker** (Tenable, March 10, 2026): 9 cross-tenant vulns in Google Looker Studio across distinct issue types — stored-credential inheritance in PostgreSQL-like connectors, Linking API abuse for BigQuery/Spanner cross-project queries, browser-proxied exfiltration via shared reports. Reported to Google starting June 2025; TRAs published Sept-Oct 2025; synthesis post March 2026. **Hunting heuristic:** cloud BI/analytics tools (Looker Studio, Power BI, Tableau Cloud, Metabase) with report sharing + embedded service credentials are systematic cross-tenant attack surface
- **Ethiack Moltbot autonomous RCE** (February 2026): autonomous security agent discovered exposed Moltbot deployments and chained misconfigurations into full RCE in under 2 hours with zero human guidance. **Hunting heuristic:** AI agents finding RCE chains in self-hosted tools validates focusing on exposed admin panels and default configs; automated tools chain basic findings faster than manual
- **Cisco SD-WAN dual incident** (March 2026): CVE-2026-20128/20122 actively exploited (web shell deployment, lateral movement — ACSC advisory) + CISA ED 26-03 mandating 24-hour patch for CVE-2026-20127 (CVSS 10.0, UAT-8616 exploiting since 2023 via software downgrade). **Hunting heuristic:** SD-WAN manager credential endpoints are high-value; test for chained file disclosure + API overwrite
- **Agentic pentesting convergence** (Feb-March 2026): AWS Security Agent (multi-agent pentest), Aikido Infinite (7 Coolify CVEs, Astro SSRF), Terra Security ($30M Series A) — commodity vulns increasingly pre-found. **Hunting heuristic:** focus on business logic, multi-step auth flows, and chain-building that resist automated discovery
- **Docker MCP Horror Stories** (March 2026): documented first full system compromise via MCP infrastructure — CVE-2025-6514 (mcp-remote CVSS 9.6) enabled OS command execution from server-provided OAuth URLs; WhatsApp MCP data exfil; MCP Inspector drive-by localhost breach (CVE-2025-49596). Pattern: MCP client-side tools processing server-provided URLs are systematically vulnerable
- **MCP ecosystem audit wave** (March 2026): Endor Labs found 82% of 2,614 MCP implementations vulnerable to CWE-22 (Anthropic mcp-server-git: 3 CVEs); Obsidian Security disclosed one-click ATO in 7 MCP clients via insecure `open()` + OAuth CSRF (4 CVEs). MCP attack surface growing faster than remediation
- **CVE-2026-30903 Zoom Workplace Mail** (March 11, 2026, CVSS 9.6): path control → arbitrary file write → DLL hijack → EoP in Zoom for Windows Mail. Network, requires user interaction (UI:R). < v6.6.0. **Pattern:** desktop mail/attachment handlers with path control → EoP chain
- **Claude Code Hook Injection** (PromptArmor, March 2026): marketplace plugins hijack Claude Code agent via hook system — `UserPromptSubmit`/`PreToolUse` hooks rewrite permissions and auto-approve commands; demonstrated full codebase exfiltration. **Hunting heuristic:** test all AI tool plugin/hook feedback channels for prompt injection via permission-rewriting paths
- **PerplexedBrowser** (Zenity Labs, March 2026): agentic browser (Perplexity Comet) — calendar invite prompt injection → local file access → 1Password credential theft. Zero-click. **Hunting heuristic:** agentic browsers combining web + local tools are highest-risk for cross-boundary injection
- **CVE-2026-25177 AD Unicode SPN collision** (March 2026, CVSS 8.8): crafted Unicode creates duplicate SPNs/UPNs bypassing AD name validation → EoP. CWE-641. **Hunting heuristic:** identity management systems with Unicode normalization gaps enable privilege escalation via name collision
