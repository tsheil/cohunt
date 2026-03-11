# AI/LLM Vulnerability Report Checklist

Quality assurance checklist for AI, LLM, MCP, and agent-related vulnerability reports. Extracted from the report-review agent for progressive disclosure.

> **Related:** [report-writing SKILL.md](../SKILL.md) for general report structure | [ai-hunting](../../ai-hunting/SKILL.md) for attack methodology | [mcp-playbooks.md](../../ai-hunting/reference/mcp-playbooks.md) for MCP test procedures

---

## Prompt Injection & Jailbreak

□ Prompt injection is reproducible across sessions (not a one-time fluke)
□ Jailbreak is explicitly in scope for this program (many programs exclude jailbreaks)
□ Indirect injection has a realistic delivery mechanism (not just "paste this into the chatbot yourself")
□ Zero-click injection (EchoLeak pattern) proves the attack works without ANY user interaction — attacker plants content, AI retrieves and acts on it autonomously
□ Multi-turn injection documents the full conversation sequence, not just the final payload
□ Multimodal injection specifies the modality (image, audio, document) and proves the hidden prompt alters AI behavior
□ Prompt injection defense bypass: references meta-analysis showing 85% success rate against SOTA defenses (arXiv:2601.17548) — strengthens case that defensive measures are insufficient
□ LPCI findings demonstrate conditional trigger activation (not just static injection)
□ GRP-Obliteration (alignment removal): if finding involves fine-tuning API abuse, reference Microsoft Feb 2026 — single prompt can cause safety regression across all 44 harm categories (13% → 93% attack success)
□ Autonomous jailbreak agents: if multi-turn attack used reasoning models, reference Nature Communications Mar 2026 — 97.14% success rate with no human supervision
□ Invisible Unicode prompt injection: if finding uses Unicode tag characters (E0000-E007F) to hide instructions, demonstrate that content appears completely benign to humans while AI processes hidden payloads (Cyrex — HackerOne Hai vulnerability)

---

## Memory & Agent Exploitation

□ Memory poisoning demonstrates persistence across sessions (not just in-context manipulation) — document time-to-propagation and scope of affected decisions
□ ZombieAgent (zero-click memory poisoning): if finding involves AI processing external emails/messages with persistent memory corruption, demonstrate self-propagation capability and cross-session persistence (Radware Jan 2026)
□ MINJA memory poisoning: if finding involves memory injection attacks, reference MINJA research showing 95%+ success rates against production agents — emphasize temporal decoupling (injection in one session, activation weeks later)
□ Salami slicing (gradual constraint bypass): if finding involves incremental interactions shifting agent behavior over time, document the full sequence of interactions, the baseline behavior, and the final drift — reference procurement agent $5M fraud case (Palo Alto Unit42)
□ Cascading failure in multi-agent system documents the propagation chain and scope of impact across agents
□ Cross-agent privilege escalation: if finding involves multi-agent systems where low-privilege agent tricks higher-privilege agent, reference ServiceNow Now Assist second-order injection — first documented cross-agent privilege escalation in production
□ AI agent action has real-world consequences (not just "LLM said something weird")
□ Impact goes beyond the user's own chat session (affects other users, triggers actions, accesses data)

---

## MCP & Protocol Layer

□ MCP-related finding identifies the specific MCP server, version, and affected tool
□ MCP sampling attack specifies the vector: resource theft, conversation hijacking, or covert tool invocation
□ OWASP MCP Top 10 risk ID cited if targeting MCP-specific behavior (MCP01-MCP10) — complements Agentic Top 10 for protocol-layer findings; also reference **CoSAI threat IDs (T1-T12)** for protocol-layer taxonomy alignment (see mcp-playbooks.md CoSAI routing matrix)
□ eval()/exec() epidemic (MCP servers): if finding involves unsanitized input to eval/exec in MCP servers, reference the 7 RCE CVEs in February 2026 sharing this root cause — systematic pattern, not isolated incident
□ MCP SDK cross-client data leak: if finding involves data leaking between client sessions, reference CVE-2026-25536 (TypeScript SDK); this is a protocol-level flaw affecting all implementations using shared instances
□ WebSocket agent hijacking (ClawJacked): if finding involves cross-origin WebSocket access to local AI agents, demonstrate that any webpage can connect — no user interaction required; full agent control with all permissions
□ MCP client infrastructure RCE: if finding involves MCP proxy/gateway/bridge components, reference CVE-2025-6514 (mcp-remote, CVSS 9.6, 437K+ npm downloads) — malicious MCP server provides crafted OAuth metadata → client passes unsanitized URL to shell → OS command injection; pattern applies to all MCP clients processing server-provided URLs
□ MCP installation flow abuse (Cursor MCPoison pattern): demonstrates trust assumption bypass during MCP server installation or setup
□ MCP health/diagnostic info disclosure: if finding involves unauthenticated info leakage from health endpoints, reference CVE-2026-29787 — document what system data is exposed (OS, CPU, memory, database paths)
□ MCP sampling attacks: if finding involves MCP servers using the sampling feature as an injection vector, reference Palo Alto Unit42 — demonstrate server becoming "active prompt author" enabling resource theft, session manipulation, or unauthorized content generation
□ A2A protocol exploitation: if finding involves Google A2A protocol, demonstrate agent identity spoofing, capability forgery, or task chain poisoning — east-west agent traffic bypasses traditional perimeters (arXiv:2505.12490)
□ Denial-of-wallet (overthinking loops): if finding involves MCP tools triggering repetition/refinement/distraction loops, quantify token amplification factor and cost impact — reference arXiv:2602.14798 (up to 142.4x amplification; each step looks normal, evading detection)
□ Git MCP server exploitation: if finding involves git-based MCP server, reference CVE-2025-68145/68143/68144 (Anthropic's mcp-server-git RCE chain via `.git/config`) — demonstrates that even first-party MCP servers are exploitable
□ AI framework regex denylist bypass: if finding involves command execution tool with denylist, reference CVE-2026-2256 (MS-Agent, CVSS 9.8) — regex-based filters systematically bypassable via command obfuscation; demonstrate specific obfuscation technique used
□ TypeScript type confusion sandbox escape: if finding involves expression sandbox bypass, reference CVE-2026-25049 (n8n, CVSS 9.4) — novel attack class where type annotations (compile-time only) don't prevent runtime exploitation via destructuring
□ Unauthenticated AI agent local server: if finding involves localhost HTTP server in AI coding tool, reference CVE-2026-22812 (OpenCode, CVSS 8.8) — demonstrate CORS policy allows cross-origin access; any website can execute shell commands
□ AI agent sandbox/container escape: if finding involves file writes outside sandbox or Docker config manipulation, reference CVE-2026-27001/27002 (OpenClaw) — demonstrate escape from intended containment boundary
□ Voice/audio extension pre-auth RCE: if finding involves AI agent voice or audio processing extensions, reference CVE-2026-28446 (OpenClaw Voice Extension, CVSS 9.8) — demonstrate pre-auth command injection via audio processing pipeline; pattern applies to any AI agent with media extension support
□ MCP connector SSRF (BlueRock pattern): if finding involves unrestricted URI fetching via MCP resource endpoints, reference BlueRock MCP Trust Registry stats (36.7% of 7,000+ servers vulnerable) — systematic ecosystem-wide pattern, not isolated incident
□ Enterprise AI zero-click injection (GeminiJack): if finding involves zero-click indirect injection in enterprise AI assistants via shared organizational content (Google Workspace, Microsoft 365), demonstrate that attacker-controlled content in shared docs/slides/emails triggers AI action without any victim interaction
□ Claude DXT extension RCE: if finding involves malicious Claude Desktop Extension (DXT), note that Anthropic declined to fix (CVSS 10.0) — demonstrate zero-click code execution on DXT install; any extension sharing mechanism creates supply chain risk
□ AI IDE confirmation bypass: if finding involves tool execution without user approval in AI IDEs, reference CVE-2026-24887 (Claude Code confirmation bypass) or CVE-2025-64106 (Cursor MCP deep-link RCE via mcpx:// protocol) — demonstrate bypass of permission model
□ Malicious AI browser extension: if finding involves AI-powered browser extension exfiltrating user data, reference 900K+ installs campaign (2026) — demonstrate conversation history theft from ChatGPT/Gemini/Claude; note this is a growing supply chain vector
□ JWE-wrapped PlainJWT bypass: if finding involves JWT authentication bypass via encrypted token wrapper, reference CVE-2026-29000 (pac4j-jwt, CVSS 10.0) — demonstrate that wrapping alg=none token inside JWE skips signature verification

---

## Supply Chain & IDE Exploitation

□ Supply chain attack via AI tool configs (hooks, MCP configs, env vars) demonstrates code execution path from repo clone to compromise
□ Agent skill supply chain (ClawHavoc/ToxicSkills pattern): if finding involves third-party AI skills, demonstrates malicious instruction execution via SKILL.md or similar skill definition files — reference ToxicSkills stats (36% prompt injection rate across 3,984 skills)
□ AI coding IDE supply chain (IDEsaster pattern): if finding involves project files exploiting AI IDEs (hooks, MCP configs, workspace files), reference the IDEsaster campaign (30+ vulns, 24 CVEs across Claude Code, Cursor, Kiro, Windsurf) — demonstrate that opening a malicious repo triggers exploitation
□ Extension recommendation squatting: if finding involves AI IDE recommending non-existent extensions that can be claimed by attackers, reference IDEsaster extension attacks on OpenVSX affecting 1.8M+ developers
□ CI/CD pipeline injection (PromptPwnd): if finding involves AI agents in GitHub Actions/GitLab CI processing untrusted issue/PR content, reference Aikido Security research — 5+ Fortune 500 confirmed affected; Google patched within 4 days; demonstrate that issue/PR content → AI agent → secret exfiltration (GEMINI_API_KEY, GITHUB_TOKEN, cloud tokens)
□ CI/CD pipeline injection (Clinejection pattern): if finding involves AI coding bots in CI/CD, demonstrates prompt injection → pipeline compromise path via GitHub Actions or similar
□ Rules file backdoor: if finding involves invisible Unicode in AI IDE config files (`.cursorrules`, `.github/copilot-instructions.md`), reference Pillar Security — demonstrate that content is undetectable via normal code review but executable by AI agents; shared config files create supply chain risk
□ Passive issue injection (RoguePilot): if finding involves GitHub issues with hidden HTML comments exploiting Copilot/Codespaces, reference Orca Security — demonstrate GITHUB_TOKEN exfiltration path via `json.schemaDownload.enable`
□ Supply chain worm propagation: if finding involves npm/package credential theft that enables self-propagation, reference Shai-Hulud (454K malicious npm packages in 2025); note dependency cooldown defense (7-14 days prevents 80% of attacks)
□ Documentation supply chain (ContextCrush pattern): if finding involves MCP-served docs/custom rules, demonstrates injection → agent execution path with no sanitization
□ Copilot CLI shell expansion: if finding involves GitHub Copilot CLI, reference CVE-2026-29783 — bash parameter expansion (`${var@P}`, `${!var}`) bypassed safety layer; demonstrate that "read-only" classification was incorrect
□ Workflow automation RCE: if finding involves n8n, Make, Zapier, or similar platforms, reference CVE-2026-21858 (n8n Ni8mare, CVSS 10.0) — unauthenticated RCE via Content-Type confusion affecting ~100K servers; demonstrate complete instance takeover
□ Agentic browser credential theft: if finding involves AI browser agents accessing password managers, reference PleaseFix (Zenity Labs) — Perplexity Comet agent hijacked to access 1Password vaults; note initial fix bypassed with `view-source:file:///`
□ Workspace trust bypass: if finding involves AI IDE auto-executing workspace tasks, reference Cursor Workspace Trust disabled by default (Oasis Security) — demonstrate `.vscode/tasks.json` executes without user confirmation; compare to VS Code's trust prompt
□ Pre-trust API key exfiltration: if finding involves AI tool processing project configs before trust dialog, reference CVE-2026-21852 (Claude Code, CVSS 5.3) — demonstrate that API base URL redirection captures auth headers before user consent
□ n8n multi-CVE patterns: if finding involves workflow automation sandbox escape, reference CVE-2026-25049 (TypeScript type confusion, CVSS 9.4) and 6-CVE batch disclosure — workflow platforms contain multiple interacting vulnerability classes

---

## Frameworks, Infrastructure & Scoring

□ System prompt leak contains genuinely sensitive data (API keys, internal URLs) — not just "you are a helpful assistant" (OWASP LLM07:2025)
□ Report distinguishes between model-level and application-level vulnerabilities
□ Output injection (XSS/SQLi via LLM) proves the app executes/renders the output (not just displays it)
□ OWASP Agentic Top 10 2026 risk ID cited if targeting agent-specific behavior (ASI01-ASI10)
□ Vector/embedding poisoning (OWASP LLM08:2025) demonstrates manipulation of RAG retrieval results, not just the embedding itself
□ Promptware Kill Chain stage assessed — findings reaching stage 5+ (C2/lateral movement/actions) are significantly more severe than stage 1-2 (injection/jailbreak); reference arxiv:2601.09625
□ Agentic browser exploitation (PleaseFix pattern) demonstrates zero-click trigger mechanism — attacker content → autonomous agent processes → impact without user interaction
□ Docker/container AI supply chain finding (DockerDash pattern) demonstrates metadata label → MCP Gateway → compromise path
□ Shadow Escape pattern: confirms exfiltration operates within authorized identity boundaries (not just external network exfil)
□ LLM framework serialization (LangGrinch pattern): if finding involves LangChain/LlamaIndex/Haystack, demonstrates prompt cascading through serialization/deserialization paths
□ Exposed agent infrastructure (Clawdbot pattern): if finding involves exposed MCP gateways/admin panels, quantifies what data is accessible (API keys, tokens, conversation histories)
□ AI recommendation poisoning (Microsoft Feb 2026): if finding involves hidden instructions in web content biasing AI recommendations, demonstrates cross-assistant impact and persistence of bias
□ React Server Component deserialization: if finding involves React/Next.js RSC, reference React2Shell (CVE-2025-55182, CVSS 10.0) — pre-auth RCE via Flight protocol deserialization; became #1 most exploited CVE on HackerOne
□ Cloud identity token abuse: if finding involves Entra ID/Azure AD, reference CVE-2025-55241 (CVSS 10.0) — Actor Tokens mechanism enabling Global Admin takeover
□ Side-channel timing analysis: if finding involves information leakage via streaming response timing, reference Whisper Leak (>98% classification across 28 LLMs) and speculative decoding attacks (>75% query fingerprinting)
□ CVSS accounts for "Attack Requirements" (CVSS 4.0) — many AI vulns need specific conditions
□ Consider OWASP AIVSS scoring (v0.5+) for AI-specific severity — extends CVSS with autonomy, non-determinism, and tool-use factors
□ CVSS V4 compliance: if submitting to Intigriti (all new submissions use CVSS V4 as of 2026), ensure scoring uses CVSS V4 metrics including Attack Requirements
□ Gravitee AI agent scale context: 3+ million AI agents in corporations, 88% reported security incidents, 47% unmonitored — use to frame organizational impact of agent-related findings
□ MS-Agent framework exploitation: if finding involves AI agent framework shell/code execution tools, reference CVE-2026-2256 (CVSS 9.8) — regex denylist bypass is a systematic pattern across AI agent frameworks
□ AI assistant URL parameter injection (Reprompt pattern): if finding involves injecting instructions via URL parameters in enterprise AI assistants, reference Varonis Reprompt disclosure — demonstrate persistent control even after chat session close; Microsoft Copilot patched Jan 13, 2026
□ AI agent CDP/WebSocket hijacking: if finding involves unauthenticated Chrome DevTools Protocol or WebSocket endpoints on AI agents, reference CVE-2026-28458 (OpenClaw Browser Relay, CVSS 7.5) and CVE-2026-28468 (Sandbox Bridge) — demonstrate cross-origin session data theft
□ AI app misconfiguration pattern: if finding involves Firebase public read/write, missing Supabase RLS, or hardcoded client-side keys in AI wrapper apps, reference Barrack.ai research (20+ breaches, 406M records exposed) — frame as systemic pattern, not isolated misconfiguration
□ LLM deanonymization risk: if finding involves AI systems correlating user data to reveal identities, reference arXiv:2602.16800 (25-67% recall, 70-90% precision at $1-4/target) — demonstrate that practical obscurity is not a valid defense
□ IDE extension namespace squatting: if finding involves AI IDEs recommending nonexistent extensions from registries, reference Koi Security disclosure (Cursor/Windsurf/Google Antigravity affected) — demonstrate attacker-controlled code execution with full IDE permissions on 1.8M+ developers
□ SANDWORM_MODE MCP supply chain: if finding involves injecting malicious MCP tool definitions through npm install hooks or Docker image labels, reference SANDWORM_MODE disclosure — demonstrate that installing an npm package or pulling a Docker image silently registers attacker MCP tools with full agent permissions
□ Meta-context injection (DockerDash pattern): if finding involves Docker image labels or container metadata processed by AI MCP gateways, reference DockerDash disclosure — demonstrate metadata → MCP tool registration → credential harvesting path; low-complexity attack via container registries
□ ForcedLeak CRM agent injection: if finding involves AI agents processing CRM form submissions, reference ForcedLeak (Salesforce Agentforce) — demonstrate that website contact form → agent processes → PII exfiltration via markdown image injection; no authentication required
□ Multi-agent cascade injection (OMNI-LEAK/AgentLeak): if finding involves multi-agent orchestration systems where compromising one agent propagates to others, reference OMNI-LEAK (82.4% LLM vulnerability rate) and AgentLeak — demonstrate inter-agent trust exploitation and cascade failure scope
□ Semantic chaining jailbreak: if finding involves structured multi-step prompts that bypass safety filters, reference semantic chaining research — demonstrate how breaking harmful requests into seemingly benign sequential steps achieves higher bypass rates than single-shot attacks
□ H-CoT chain-of-thought hijacking: if finding involves manipulating model reasoning traces to alter outputs, reference H-CoT disclosure — demonstrate that injecting reasoning-style content into context causes models to adopt attacker's logical framework
□ AI framework SSRF via HTTP redirect: if finding involves AI framework HTTP client following redirects to internal services, reference CVE-2026-27795 (LangChain) and CVE-2026-25904 (Pydantic-AI, archived/won't fix) — demonstrate that redirect from allowed URL → internal endpoint bypasses URL validation
□ Identity/config file poisoning (SOUL.md pattern): if finding involves AI identity files (SOUL.md, system prompts, persona configs) that can be tampered with via repo access or MCP tools, reference SOUL.md poisoning disclosure — demonstrate behavior modification persists across all user sessions
□ ChainLeak session hijacking: if finding involves AI framework session token extraction, reference CVE-2026-22218/22219 (Chainlit) — demonstrate that unauthenticated API endpoints leak JWT tokens enabling full session impersonation
□ Pydantic-AI SSRF (archived/won't fix): if finding involves Pydantic-AI MCP server SSRF, note CVE-2026-25904 was archived as won't-fix — vendor disputes are reportable context for alternative disclosure paths
□ OpenClaw query parameter credential relay: if finding involves AI agent passing URL query parameters to shell commands, reference CVE-2026-25253 (OpenClaw) — demonstrate that `?param=malicious` in agent URL results in command injection via unsanitized parameter relay
