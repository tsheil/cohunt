# AI Attack Surface & Novel Vulnerability Patterns

Detailed attack patterns for MCP, agent supply chains, AI coding IDEs, agentic browsers, and novel vulnerability classes. Reference file for the ai-hunting skill.

---

### MCP (Model Context Protocol) as Attack Surface

MCP is rapidly being adopted to connect AI agents to enterprise tools and data. This creates a **new attack surface** for bug bounty hunters:

**Key Vulnerability Classes:**

| MCP Vuln Type | Description | Severity |
|---|---|---|
| **Tool Poisoning** | Malicious instructions embedded in MCP tool descriptions; invisible to users but interpreted by the AI model. Compromised descriptions manipulate the model into executing unintended tool calls | High-Critical |
| **Indirect Prompt Injection via MCP** | Attacker plants instructions in content the MCP server retrieves (GitHub issues, documents, emails). The LLM processes this as trusted context and follows injected instructions | Critical |
| **Over-Privileged Tokens** | MCP servers often connect with broad PATs or API keys. A single prompt injection can chain actions across multiple systems using the server's credentials | Critical |
| **Cross-System Data Exfiltration** | Once an agent can call tools via MCP, a single prompt can trigger actions across multiple systems — reading private repos, accessing databases, sending emails | Critical |
| **Tool Shadowing** | A malicious MCP server registers tools with names similar to legitimate ones, intercepting calls intended for trusted tools | High |

**Additional MCP Vulnerability Classes:**

| MCP Vuln Type | Description | Severity |
|---|---|---|
| **eval() Epidemic Pattern** | User-controlled input passed directly to `eval()`, `exec()`, or `execAsync()` in MCP server implementations. 7 RCE CVEs in February 2026 alone shared this root cause — all in integration tools bridging AI assistants to other systems. Named pattern: all are unauthenticated or low-privilege, most unpatched at disclosure | Critical |
| **SDK-Level Cross-Client Data Leak** | CVE-2026-25536: MCP TypeScript SDK vulnerability where reusing a single McpServer instance leaks responses across client boundaries — one client receives data intended for another. Protocol-level flaw, not implementation-specific | High-Critical |
| **SDK Case Sensitivity Bypass** | CVE-2026-27896: MCP Go SDK JSON parser handles field names case-insensitively, allowing crafted MCP responses with altered field names (e.g., "Method" vs "method") to bypass validation logic | High |
| **MCP Health Endpoint Info Disclosure** | CVE-2026-29787: `/api/health/detailed` endpoint in mcp-memory-service leaks OS version, CPU count, memory, disk, database path without authentication when anonymous access is enabled | Medium |
| **ClawJacked (WebSocket Hijacking)** | Malicious websites hijack local AI agent instances (e.g., OpenClaw) via WebSocket connections. Attacker page connects to localhost-bound WebSocket, gains remote control of local agent with all its permissions and tool access | Critical |
| **"Rug Pull" Attacks** | MCP servers modify tool definitions between sessions, presenting different capabilities than initially approved. Exploits dynamic nature of MCP tool definitions for post-deployment modification | High |
| **Command Injection in MCP Packages** | CVE-2025-6514: critical OS command injection in `mcp-remote` (437K+ downloads) — malicious MCP servers send booby-trapped authorization_endpoint for RCE. Effectively a supply-chain backdoor | Critical |
| **Sandbox Escape** | Anthropic's own Filesystem-MCP server had sandbox escape + symlink bypass enabling arbitrary file access and code execution | Critical |
| **Token Passthrough Anti-Pattern** | MCP server accepts tokens from client without validating they were properly issued to the server — fundamental auth architecture flaw | High |
| **Claude Code Project File Exploitation** | CVE-2025-59536 / CVE-2026-21852: RCE and API token exfiltration through Claude Code project files (Check Point Research) | Critical |
| **Git MCP Server RCE** | CVE-2025-68145, CVE-2025-68143, CVE-2025-68144: Three CVEs in Anthropic's Git MCP server enabling RCE via prompt injection | Critical |
| **MCP Server SSRF → Cloud Compromise** | Unpatched SSRF in Microsoft MarkItDown MCP server — compromises AWS EC2 instances via metadata service exploitation | Critical |
| **Gemini MCP 0-day** | CVE-2026-0755 (CVSS 9.8): command injection via execAsync in gemini-mcp-tool; vendor unresponsive, published as 0-day | Critical |
| **Godot MCP Command Injection** | CVE-2026-25546: command injection via exec() in godot-mcp server | High |
| **GitHub Kanban MCP Injection** | CVE-2026-0756: command injection in create_issue functionality | High |
| **eBay API MCP RCE** | CVE-2026-27203: RCE via updateEnvFile in eBay API MCP Server | Critical |
| **mcp-remote OAuth RCE** | CVE-2025-6514 (CVSS 10.0): OS command injection via malicious authorization_endpoint in mcp-remote npm package (437K+ downloads) | Critical |
| **DockerDash MCP Gateway** | Malicious Docker image labels compromise environments via Ask Gordon AI MCP Gateway — data exfil of container details, network topology | High-Critical |
| **Shadow Escape (Zero-Click MCP)** | Hidden instructions in documents cause MCP-enabled AI to exfiltrate PII from connected databases and CRM systems within authorized identity boundaries | Critical |
| **ContextCrush (Documentation Supply Chain)** | Malicious instructions injected into trusted documentation served via MCP servers (e.g., Context7's "Custom Rules"). AI coding assistants consume poisoned library docs as trusted context, executing attacker instructions (env file theft, file deletion) | Critical |
| **LLM Framework Serialization** | Untrusted LLM-influenced metadata deserialized as objects (LangGrinch/CVE-2025-68664 pattern). Single prompt cascades through serialization in streaming operations to exfiltrate secrets | Critical |
| **MCP TypeScript SDK ReDoS** | CVE-2026-0621 (CVSS 8.7): `partToRegExp()` generates regex with nested quantifiers for exploded template variables (e.g., `{/id*}`), causing catastrophic backtracking and 100% CPU utilization via crafted URI in `resources/read` request | High |
| **Pydantic-AI MCP SSRF** | CVE-2026-25904: overly permissive Deno sandbox settings allow network access to localhost, enabling SSRF against internal services from "sandboxed" MCP Python execution. Project **archived** — will NOT receive a patch | Medium-High |
| **Copilot CLI Shell Expansion RCE** | CVE-2026-29783: `env` command on Copilot CLI's allowlist used to bypass read-only assessment — `env curl | env sh` downloads and executes malware with zero approval. Exploitable via poisoned README prompt injection (PromptArmor, March 2026) | Critical |
| **OpenClaw Query String RCE** | CVE-2026-25253 (CVSS 8.8): `gatewayUrl` query parameter auto-establishes WebSocket connection transmitting user auth credentials to attacker-controlled server — 1-click RCE even on localhost-only instances | Critical |
| **OpenClaw TAR Path Traversal** | CVE-2026-28453: TAR archive extraction fails to validate entry paths, allowing `../../` traversal to write files outside intended directories | High |
| **OpenClaw SHA-1 Cache Poisoning** | CVE-2026-28479: deprecated SHA-1 used for sandbox identifier cache keys, vulnerable to collision attacks enabling unsafe sandbox state reuse | Medium |
| **OpenClaw Git Pre-Commit Injection** | CVE-2026-28484: maliciously-named files beginning with dashes inject git flags, leaking `.env` and sensitive files to git history | High |
| **GlicJack Chrome Gemini Hijack** | CVE-2026-0628 (CVSS 8.8, Palo Alto Unit 42): insufficient policy enforcement in Chrome WebView tag allows malicious extensions to hijack Gemini Live panel — camera/microphone access, screenshots of any website, local file access. Patched in Chrome 143.0.7499.192 | High-Critical |
| **Copilot JetBrains RCE** | CVE-2026-21516: remote code execution via command injection in GitHub Copilot for JetBrains IDEs | Critical |
| **ChainLeak (Chainlit)** | CVE-2026-22218 + CVE-2026-22219: arbitrary file read + SSRF via SQLAlchemy data layer in Chainlit AI framework — chained for full cloud environment compromise with no user interaction (Zafran Labs). Fixed in Chainlit 2.9.4 | Critical |
| **LangChain SSRF via Redirect** | CVE-2026-27795: `RecursiveUrlLoader` fails to handle HTTP redirects manually — attacker supplies benign URL that passes validation, then redirects to internal resources (e.g., AWS metadata at 169.254.169.254). Fixed in @langchain/community >= 1.1.18 | High |
| **Agentgateway Input Sanitization** | CVE-2026-29791: missing parameter sanitization in MCP-to-OpenAPI conversion — path, query, and header values from MCP tools/call requests sent unsanitized as OpenAPI requests. Fixed in 0.12.0 | Medium-High |
| **SANDWORM_MODE MCP Injection** | npm supply chain worm (Socket, Feb 2026): 19 typosquatting packages with "McpInject" module deploys malicious MCP server into AI coding assistant configs (Claude Code, Cursor, Windsurf). Rogue MCP tools embed prompt injections to steal SSH keys, AWS credentials, .env files. 48-hour delayed activation with per-machine jitter | Critical |
| **Pydantic AI SSRF** | CVE-2026-25580: SSRF in URL download functionality (versions 0.0.26 to <1.56.0) — when apps accept message history from untrusted sources, attackers include malicious URLs targeting internal resources. Fixed in 1.56.0 | High |

**Real-World Examples:**

1. **GitHub MCP Server Breach:**
```
1. Attacker creates a public GitHub issue with hidden prompt injection
2. Victim's AI assistant (connected via GitHub MCP server) processes the issue
3. Injected prompt instructs the agent to read private repos
4. Over-privileged PAT allows access to private repos, internal projects, personal financial data
5. Data exfiltrated through the agent's normal output channel
```

2. **Supabase Cursor Agent Breach (mid-2025):** Cursor agent running with privileged service-role access processed support tickets containing user-supplied input as commands. Attackers embedded SQL instructions to read and exfiltrate sensitive integration tokens.

3. **mcp-remote Supply Chain (CVE-2025-6514):** Critical command injection in the widely-used mcp-remote package allowed malicious MCP servers to achieve RCE via crafted authorization_endpoint URLs. With 437K+ downloads and adoption in major companies, unpatched installs became supply-chain backdoors.

4. **Clawdbot/Moltbot Incident (January 2026):** Open-source AI agent framework went viral over the weekend of Jan 24-25. Within 72 hours: exposed admin panels, critical RCE vulnerabilities, and active infostealer campaigns (RedLine, Lumma, Vidar) targeting its configuration directories. 2,000+ exposed gateways visible on Shodan. Exposed MCP endpoints revealed API keys, OAuth tokens, bot credentials, and conversation histories. During the rushed rebranding to Moltbot, scammers hijacked original X and GitHub handles to promote fake $CLAWD token. Demonstrates cascading risks of viral AI agent adoption without security review.

5. **ContextCrush (February 2026):** Noma Labs discovered supply chain vulnerability in Context7 (operated by Upstash) — a popular MCP server providing library documentation to AI coding assistants. Attackers could inject malicious instructions through Context7's "Custom Rules" feature; rules served verbatim through the MCP server with no sanitization. PoC showed a poisoned library entry prompting the AI to search for .env files, exfiltrate them, and delete files under the pretext of a "Cleanup" task. Patched February 23, 2026. Pattern: trusted documentation → MCP delivery → agent execution.

6. **ClawJacked WebSocket Hijack (February 2026):** Malicious websites exploited WebSocket connections to hijack local OpenClaw AI agent instances. Any webpage could connect to the localhost-bound WebSocket and gain full remote control of the local agent, including all tools and permissions. Demonstrates that locally-running AI agents with WebSocket interfaces are vulnerable to cross-origin hijacking.

7. **eval() Epidemic — 7 MCP RCE CVEs in One Month (February 2026):** Seven remote code execution vulnerabilities published in February 2026 alone, all sharing the same root cause: user-controlled input reaching dangerous execution functions (`eval()`, `exec()`, `execAsync()`) without sanitization. All were in MCP integration tools. Examples include CVE-2026-25546 (godot-mcp), CVE-2026-0755 (gemini-mcp-tool, CVSS 9.8), CVE-2026-0756 (GitHub Kanban MCP), CVE-2026-27203 (eBay API MCP RCE). Pattern: arbitrary execution is the feature; the question is whether that execution is controllable by adversarial input. In all seven cases, it was.

8. **LangGrinch (CVE-2025-68664, CVSS 9.3):** Critical serialization injection in LangChain Core where untrusted LLM-influenced metadata could be rehydrated as objects, enabling secret exfiltration and unsafe instantiation. A single text prompt cascading through serialization/deserialization in streaming operations. Impacts ~847M total downloads. LangChain awarded $4,000 — maximum ever for the project. Patched in versions 1.2.5 and 0.3.81. Demonstrates that AI frameworks themselves are high-value targets.

9. **SANDWORM_MODE Supply Chain Worm (February 2026):** Socket discovered a self-propagating npm worm using 19 typosquatting packages. Key MCP-specific capability: an "McpInject" module deploys a malicious MCP server and injects it into AI coding assistant configs (Claude Code, Claude Desktop, Cursor, VS Code Continue, Windsurf). The rogue MCP server registers seemingly-harmless tools embedding prompt injections to steal `~/.ssh/id_rsa`, `~/.aws/credentials`, `~/.npmrc`, and `.env` files. Also harvests API keys for 9 LLM providers (Anthropic, Cohere, OpenAI, etc.). Uses 48-hour delayed activation with per-machine jitter. Pattern: npm install → MCP injection → credential theft.

10. **DockerDash Meta-Context Injection (February 2026):** Noma Security disclosed that a single malicious metadata label in a Docker image could compromise the entire Docker environment through Docker's Ask Gordon AI assistant. Three-stage chain: Gordon AI reads malicious instruction → forwards to MCP Gateway → MCP tools execute with zero validation. Noma coined "Meta-Context Injection" for this class where MCP gateways cannot distinguish descriptive metadata from pre-authorized instructions. Fixed in Docker Desktop 4.50.0.

11. **Copilot CLI Malware Download (March 2026):** Two days after GitHub Copilot CLI hit general availability, PromptArmor demonstrated `env curl | env sh` bypasses all protections — downloading and executing malware with zero human approval. The `env` command was on Copilot's read-only allowlist. Works against any cloned repo with a poisoned README containing prompt injection. CVE-2026-29783 assigned for the underlying shell expansion bypass.

12. **PleaseFix/PerplexedBrowser Zero-Click Agent Hijack (March 2026):** Zenity Labs disclosed critical vulnerabilities in agentic browsers including Perplexity Comet. Two exploit paths: (1) zero-click file system exfiltration via attacker-controlled calendar invites triggering autonomous agent execution; (2) credential theft via 1Password takeover without exploiting the password manager itself. Perplexity classified as critical and patched before disclosure. Demonstrates that agentic browsers create fundamentally new attack surfaces.

13. **ForcedLeak Salesforce Agentforce (March 2026):** Varonis disclosed ForcedLeak (CVSS 9.4) — a vulnerability chain in Salesforce's Agentforce platform enabling CRM data exfiltration through indirect prompt injection via Web-to-Lead forms. Attackers embed malicious instructions in the "Description" field, leveraging prompt injection + agent overreach + misconfigured CSP.

14. **Operation Bizarre Bazaar — LLMjacking at Scale (January 2026):** Pillar Security documented the first publicly attributed campaign systematically targeting exposed LLM and MCP endpoints with commercial monetization. 35,000 attack sessions (972/day) targeting 175,000+ exposed Ollama instances across 130 countries. By late January, 60% of observed attack traffic shifted from compute theft to MCP reconnaissance. Operator ran "silver.inc" as a commercial marketplace reselling unauthorized access to 30+ LLM providers.

15. **Notion AI Data Exfiltration (January 2026):** PromptArmor demonstrated hidden white text in Notion documents with prompt injection causes Notion AI to exfiltrate salary data, candidate feedback, and internal role details via invisible image requests. Notion initially closed as N/A on HackerOne before remediating after public disclosure.

16. **ChainLeak — Chainlit Cloud Compromise (March 2026):** Zafran Labs chained CVE-2026-22218 (arbitrary file read) + CVE-2026-22219 (SSRF via SQLAlchemy data layer) in Chainlit AI framework for full cloud environment compromise with no user interaction. Demonstrates that AI framework vulnerabilities can cascade into infrastructure-level access.

**Implementation Vulnerability Stats:**
- **30+ MCP CVEs filed in just 60 days** — MCP is now AI's fastest-growing attack surface (MCP Security Research, early 2026)
- **43% of tested MCP server implementations** contained command injection flaws (March 2025)
- **38% of 500+ scanned MCP servers** completely lack authentication (2026 scan)
- **30% permitted unrestricted URL fetching** (SSRF-prone)
- **8,000+ MCP servers** found publicly exposed in February 2026, with 492 identified as vulnerable (lacking authentication or encryption)
- **Attack surface spans three layers:** MCP servers themselves, protocol implementation libraries (SDKs), and the machines running MCP clients — a vulnerability in any layer compromises the entire chain
- **CVE-2025-6514** rated CVSS 10.0 — critical command injection in mcp-remote npm package
- **CVE-2026-27896**: MCP Go SDK vulnerability — JSON parser handles field names case-insensitively, allowing attackers to craft malicious MCP responses with altered field names (e.g., "Method" instead of "method")
- **CVE-2025-53109**: critical symlink bypass vulnerability enabling modification of privileged files; system takeover if MCP server runs with elevated privileges
- **Palo Alto Unit42** published new research on prompt injection attack vectors through MCP sampling — agents with long conversation histories are significantly more vulnerable to manipulation
- **Adversa AI published MCP Security TOP 25** — definitive catalog of 25 MCP vulnerability categories
- **11 MCP vulnerability classes** systematically cataloged, including supply chain typosquatting and cross-server context abuse
- **MCP Security Bench (MSB)** accepted at ICLR 2026 as academic benchmark for MCP attacks
- VulnerableMCP.info tracks MCP-specific vulnerability database
- **MCP Security Tools:** mcp-scan v0.4.2 (Invariant Labs, acquired by Snyk Feb 2026 — standard MCP scanner for tool poisoning, rug pulls, cross-origin escalation; static + proxy modes), MCPTox (tool poisoning benchmark), MCPGuard (automated vuln detection), MCP Golf Testing (offensive toolkit), Semgrep MCP Server (code scanning via MCP), Escape ASM (discovers unauthenticated MCP endpoints)
- **OWASP MCP Top 10** (2026): dedicated security framework for MCP-specific risks — token mismanagement, privilege escalation, command injection, supply chain, tool poisoning, shadow servers
- **CoSAI MCP Security Whitepaper** (January 27, 2026): Coalition for Secure AI released comprehensive taxonomy identifying **12 core threat categories** and **~40 distinct threats** — tool poisoning, shadow servers, confused deputy problem, identity spoofing via weak authentication, malicious tool metadata manipulation; recommended controls include strong identity chains, zero-trust for AI agents, and sandboxing
- **MCP Auth Security**: 88% of MCP servers require credentials, but 53% rely on insecure long-lived static secrets; modern OAuth adoption only 8.5% (Astrix State of MCP Security 2025)
- **Enkrypt AI scan of top 1,000 MCP servers**: 33% had critical vulnerabilities, averaging **5.2 vulnerabilities per server** — systemic quality crisis
- **MCPTox benchmark results**: tool poisoning attack success rates exceed **60%** on models like GPT-4o-mini, o1-mini, DeepSeek-R1, and Phi-4 across 1,312 malicious test cases
- **CyberArk advanced tool poisoning variants**: malicious instructions embedded not just in descriptions but in function names, parameter types, required fields arrays, and default values — bypass description-only scanning
- **MCP protocol now mandates OAuth 2.1 + PKCE** for authentication; Protected Resource Metadata (PRM) mandatory; Resource Indicators required in authorization and token requests (2025-11-25 spec update)
- **BlueRock MCP Trust Registry** (March 2026): analyzed **7,000+ MCP servers** — **36.7% exposed to SSRF** vulnerabilities; launched MCP Trust Registry (mcp-trust.com) providing security analysis across security rules and tool analysis; disclosed "MCP fURI" SSRF in Microsoft MarkItDown MCP server enabling cloud credential theft via instance metadata
- **Docker MCP Defender + Gateway**: Docker published "MCP Horror Stories" series documenting real attacks (WhatsApp exfiltration, GitHub prompt injection, localhost breach, supply chain attack); MCP Defender provides runtime detection of tool poisoning and data exfiltration; Docker MCP Gateway provides infrastructure-level protection with sandboxed execution
- **Log-To-Leak Framework** (ICLR 2026 submission): new class of prompt-level privacy attacks targeting tool invocation — covertly forces agents to invoke a malicious logging tool to exfiltrate user queries, tool responses, and agent replies. Evaluated across 5 real-world MCP servers and 4 LLM agents (GPT-4o, GPT-5, Claude-Sonnet-4, GPT-OSS). Preserves task quality while exfiltrating data — extremely hard to detect
- **PoisonedRAG** (USENIX Security 2025): first knowledge corruption attack on RAG systems — attackers inject semantically meaningful poisoned texts into RAG databases, inducing LLMs to generate targeted attack outputs. Test RAG systems for poisoned document injection
- **"Lethal Trifecta" Pattern**: repeated pattern across real-world MCP incidents — privileged access + untrusted input + external communication channel. Check every MCP deployment for this combination
- **5.5% of MCP servers** in the wild exhibit active tool poisoning attacks; **33% allow unrestricted network access** (arXiv research on 2,614 MCP implementations)
- **Endor Labs MCP Analysis** (2,614 implementations): 82% use file system operations prone to Path Traversal (CWE-22), 67% use sensitive APIs related to Code Injection (CWE-94), 34% use APIs related to Command Injection (CWE-78) — MCP servers inherit classical web vulnerability patterns
- **ETDI** (Enhanced Tool Definition Interface, arXiv:2506.01333): proposed mitigation for tool squatting and rug pull attacks using OAuth-enhanced tool definitions, cryptographic signing, immutable versioning, and policy-based access control
- **MCP Sampling Attacks** (Palo Alto Unit42, 2026): MCP's sampling feature (where servers can initiate LLM text generation) creates new prompt injection angles — a malicious server becomes an "active prompt author" rather than a passive tool. Demonstrated attacks: resource theft, persistent session manipulation, unauthorized content generation
- **MCPJam Inspector RCE** (CVE-2026-23744): unauthenticated HTTP endpoint can install arbitrary MCP servers; listens on 0.0.0.0 by default, enabling remote code execution from the network
- **Tenable Cloud & AI Security Risk Report** (2026): 70% of organizations integrated AI/MCP third-party packages without centralized security oversight; 18% granted AI services administrative permissions rarely audited; non-human identities (AI agents, service accounts) represent higher risk (52%) than human users (37%)
- **Gravitee State of AI Agent Security 2026**: **3+ million AI agents** operating in corporations; **88% of organizations** reported confirmed or suspected AI agent security incidents; **47% of agents** not actively monitored or secured (~1.5 million at risk); only 14.4% of organizations have full security approval for their agent fleet
- **Endor Labs MCP shell=True pattern**: 34% of 2,614 studied MCP implementations use APIs prone to command injection (CWE-78) — systemic pattern, not isolated incidents; 38% completely lack authentication
- **n8n 6-CVE batch disclosure** (February 2026): six CVEs disclosed in a single day covering RCE, command injection, arbitrary file access, and XSS — including CVE-2026-25049 (CVSS 9.4, TypeScript type confusion sandbox escape) and CVE-2026-21877 (Git Node RCE affecting n8n Cloud)
- **MINJA** (Memory Injection Attack): research demonstrating **95%+ injection success rates** against production AI agents with persistent memory — temporally decoupled attacks where injection in one session activates weeks later

### Agent Skill Supply Chain Attacks (2026)

A major new attack surface emerging in early 2026 targeting AI agent skill/plugin ecosystems:

**ClawHavoc Campaign (February 2026):**
- Largest confirmed supply chain attack on AI agent infrastructure
- 1,184+ malicious skills identified on ClawHub marketplace (roughly 1 in 5 packages)
- 335 of 341 initial malicious skills traced to a single coordinated operation
- Deployed Atomic macOS Stealer (AMOS) stealing browser credentials, keychains, SSH keys, crypto wallets
- Adversarial instructions embedded directly in SKILL.md files — agents process as trusted instruction source
- Targeted macOS and Windows users running OpenClaw on always-on machines (Mac minis hosting AI agents)

**ToxicSkills Study (Snyk, February 2026):**
- First comprehensive security audit of AI agent skills ecosystem (3,984 skills from ClawHub and skills.sh)
- **36% contain prompt injection**; **1,467 contain malicious payloads**
- 13.4% (534 skills) contain critical-level issues
- 100% of confirmed malicious skills contain malicious code patterns; 91% simultaneously employ prompt injection
- 2.9% of skills dynamically fetch and execute content from external endpoints at runtime — attackers can modify behavior at any time
- SKILL.md to shell access achievable in **three lines of markdown**

**Clinejection (Snyk, 2026):**
- Prompt injection used to turn AI coding bots into supply chain attack vectors through GitHub Actions pipelines
- Malicious PR content → AI coding agent processes → injected code merged or pipeline compromised

**Agent Skill Scanner Tools:**

| Scanner | Type | What It Catches | Setup |
|---------|------|-----------------|-------|
| **Cisco MCP Scanner v4.0.1** | CLI (Python, 900+ stars) | YAML/YARA static analysis, bytecode analysis, behavioral dataflow tracing, LLM-as-judge semantic evaluation | `pip install mcp-scanner` |
| **Snyk Agent Scan** | CLI | Prompt injection, tool poisoning, tool shadowing, toxic flows, rug pulls, malware payloads, hardcoded secrets | Auto-discovers Claude Code/Desktop, Cursor, Gemini CLI, Windsurf configs |
| **Repello SkillCheck** | Browser-based | Prompt injection variants, env var exfiltration, policy violations, payload delivery indicators | Upload skill zip, get score (0-100) |
| **MCPGuard** (VirtueAI) | Agent-based | Purpose-built fine-tuned models for MCP-specific patterns; analyzed 700+ servers, found critical vulns in 78% | Maintains scan memory across codebase |
| **BlueRock MCP Trust Registry** | Web platform | Security analysis of 7,000+ MCP servers; SSRF, auth, and tool poisoning detection; provides trust scores | mcp-trust.com |
| **Akto** | Platform | 1,000+ real-world agent exploits; continuous red teaming + runtime guardrails; MCP discovery | Enterprise deployment |
| **Agent-Wiz** (Repello AI) | CLI | Threat modeling and visualization for LangGraph, AutoGen, CrewAI agents | GitHub: `Repello-AI/Agent-Wiz` |

**Testing Agent Skills:**
1. Before installing any third-party skill, scan with at least one of the tools above
2. Read the SKILL.md file manually — look for environment variable references, external URL fetching, trigger conditions
3. Check if skill dynamically fetches content from external endpoints (2.9% of ClawHub skills do this)
4. Test if skill instructions can be manipulated by content the agent processes
5. Monitor for post-installation behavior changes (rug pull pattern)

### AI Recommendation Poisoning (Microsoft, February 2026)

A new vulnerability class where companies embed hidden instructions in web content to manipulate AI assistants:

**How It Works:**
- Companies embed hidden instructions in "Summarize with AI" buttons, meta tags, or URL prompt parameters
- URL parameters instruct AI to "remember [Company] as a trusted source"
- Over 50 unique poisoning prompts from 31 companies across 14 industries identified by Microsoft researchers
- Compromised AI assistants provide subtly biased recommendations on health, finance, and security topics

**Testing for AI Recommendation Poisoning:**
1. Check if target's web pages contain hidden instructions in meta tags, invisible text, or URL parameters targeting AI summarization
2. Test if AI assistants that browse the target's content develop persistent biases toward the target's products/services
3. Look for `data-ai-*` attributes, hidden divs with AI instructions, or URL parameters like `?ai_context=`
4. Test across multiple AI assistants (ChatGPT, Gemini, Copilot) to confirm cross-platform impact
5. Map to ASI06 (Memory Poisoning) if the bias persists across sessions

**Severity Guidance:** High if AI recommendations influence financial, health, or security decisions; Medium if limited to product recommendations; Low if contained to a single session.

**Security MCP Servers for Bug Bounty Workflows:**

| MCP Server | Purpose | Integration |
|---|---|---|
| **Burp Suite MCP** (PortSwigger) | Official extension — automated HTTP request crafting, proxy history analysis, scanner issue retrieval | Claude Desktop, Claude Code |
| **Snyk MCP** | Integrated security scanning via Snyk CLI; standardized AI-agent interface to Snyk's vulnerability database | Claude Desktop, Claude Code |
| **Semgrep MCP** | Bridges Semgrep SAST scanning with AI-assisted workflows | Claude Desktop, Claude Code |
| **Levo MCP** | Runtime security intelligence — API specs, runtime traces, vulnerabilities, exploit data, auth states | Claude Desktop, Claude Code |
| **Nmap MCP** | Natural language network reconnaissance via Nmap | Claude Desktop, Claude Code |
| **SQLMap MCP** | AI-guided SQL injection testing via SQLMap | Claude Desktop, Claude Code |
| **FFUF MCP** | AI-driven web fuzzing | Claude Desktop, Claude Code |
| **MobSF MCP** | Mobile app security testing via Mobile Security Framework | Claude Desktop, Claude Code |
| **Kali Linux MCP** (`mcp-kali-server`) | SSH bridge to Kali box — Claude turns natural-language prompts into security tool commands | Claude Desktop, Claude Code |

**Popular Combined Workflow:** Claude Code + Kali Linux MCP — connect via SSH to a Kali box, use natural-language prompts to orchestrate nmap, gobuster, curl, etc. for automated security assessments

**Testing MCP Deployments:**
1. Check what permissions/scopes the MCP server's credentials have (PATs, API keys, OAuth tokens)
2. Test if tool descriptions can be poisoned by upstream content
3. Inject prompts into content the MCP server retrieves (documents, issues, messages)
4. Check if the agent validates tool call parameters before execution
5. Test cross-system chaining: can a prompt in System A trigger actions in System B?
6. Check for tool shadowing: register a tool with a similar name to a legitimate one
7. Test "rug pull" scenarios: can tool definitions change between sessions without re-approval?
8. Check for command injection in MCP server configuration parameters (especially OAuth/auth endpoints)
9. Test sandbox/containment escape via symlinks, path traversal, or filesystem operations
10. Map findings to **OWASP MCP Top 10** risk IDs (MCP01-MCP10) for stronger reports
11. Test documentation supply chain (ContextCrush pattern): can "custom rules" or contributed docs inject instructions?
12. Scan for shadow MCP servers (MCP07): unauthorized servers running on enterprise networks without security awareness
13. Test for eval()/exec() epidemic pattern: does the MCP server pass user input to dangerous execution functions? (7 CVEs in Feb 2026 shared this root cause)
14. Test for SDK-level cross-client data leaks: if MCP server reuses a single instance across clients, can responses leak between sessions? (CVE-2026-25536)
15. Test for WebSocket hijacking (ClawJacked pattern): if AI agent runs locally with WebSocket interface, can a malicious webpage connect and control it?
16. Test for salami slicing: can repeated small interactions gradually shift agent behavior past its constraints?
17. Test for MCP sampling attacks: can an MCP server use the sampling feature to become an "active prompt author" and inject instructions? (Unit42 research — resource theft, session manipulation, unauthorized content generation)
18. Test for rules file backdoor: do AI IDE configuration files (`.cursorrules`, `.github/copilot-instructions.md`) contain invisible Unicode characters with hidden instructions? (Pillar Security)
19. Test for CI/CD pipeline injection (PromptPwnd pattern): can a malicious GitHub issue or PR inject prompts into AI agents running in CI/CD pipelines, leaking secrets? (Aikido Security — 5+ Fortune 500 confirmed affected)
20. Test for denial-of-wallet via MCP overthinking loops: can crafted tool responses trigger repetition, forced refinement, or distraction loops that amplify token consumption up to 142.4x? (arXiv:2602.14798 — each step looks normal, making detection difficult)
21. Test for invisible prompt injection via Unicode tag characters (range E0000-E007F): can hidden instructions be embedded within normal-looking text input to manipulate AI behavior undetectably? (HackerOne Hai vulnerability — Cyrex)
22. Test for hybrid prompt injection 2.0: can prompt injection payloads combine with traditional exploits (XSS, CSRF, SQLi) to create compound attack chains? (arXiv:2507.13169)
23. Test for mcp-server-git RCE: if target uses git-based MCP servers, can malicious `.git/config` files achieve code execution? (CVE-2025-68145/68143/68144 — even Anthropic's first-party MCP server was vulnerable)
24. Test for AI framework regex denylist bypass: if target uses AI agent frameworks with command execution (Shell tools, code interpreters), can command obfuscation bypass regex-based denylists? (CVE-2026-2256, ModelScope MS-Agent, CVSS 9.8 — prompt injection → full system compromise)
25. Test for unauthenticated AI agent local servers: does the AI coding tool auto-start an HTTP server on localhost? Is CORS permissive? Can any website trigger command execution? (CVE-2026-22812, OpenCode — any local process or website could execute arbitrary shell commands)
26. Test for workspace trust bypass: does the AI IDE disable workspace trust by default? Can malicious `.vscode/tasks.json` auto-execute without user confirmation? (Oasis Security — Cursor ships with Workspace Trust disabled)
27. Test for sandbox/container escape via AI agent config: can AI agent installation or skill download flows write files outside intended directories or inject dangerous Docker options? (CVE-2026-27001/27002, OpenClaw — sandbox escape + Docker container escape)
28. Test for TypeScript type confusion sandbox escape: if target uses sandboxed expression evaluation, can TypeScript type annotations (not enforced at runtime) bypass sanitization via destructuring syntax? (CVE-2026-25049, n8n, CVSS 9.4 — novel sandbox escape pattern)
29. Test for MCP server SSRF via unrestricted URI handling: does the MCP server accept arbitrary URIs without validation? In cloud deployments, can instance metadata (169.254.169.254) be queried to steal credentials? (BlueRock "MCP fURI" — 36.7% of 7,000+ MCP servers exposed; Microsoft MarkItDown MCP SSRF → AWS credential theft)
30. Test for AI agent voice/audio extension RCE: if target has voice-call or audio processing extensions, test inbound allowlist bypass via empty caller IDs, suffix matching, or malformed SIP/VoIP headers (CVE-2026-28446, OpenClaw voice-call extension, CVSS 9.8 — pre-auth RCE affecting 42,000+ instances)
31. Test for JWT alg=none authentication bypass in AI platforms: if target uses JWT-based authentication (especially with encrypted JWTs/JWE), test for PlainJWT wrapping that skips signature verification (CVE-2026-29000, pac4j-jwt, CVSS 10.0 — RSA public key + JWE-wrapped PlainJWT = full auth bypass as any user including admin)
32. Test for AI CLI tool allowlist bypass: if target AI CLI has a "safe commands" allowlist, can dangerous commands be wrapped via allowed utilities? (CVE-2026-29783, Copilot CLI — `env curl | env sh` bypasses read-only assessment via allowlisted `env` command)
33. Test for query parameter credential relay: does the AI agent accept connection URLs via query string parameters? Can an attacker-controlled URL steal auth tokens by auto-establishing WebSocket connections? (CVE-2026-25253, OpenClaw — `gatewayUrl` param = 1-click RCE)
34. Test for MCP SDK ReDoS: does the MCP SDK's URI template processing use regex with nested quantifiers? Can crafted URIs cause catastrophic backtracking and 100% CPU utilization? (CVE-2026-0621, MCP TypeScript SDK — exploded template variables like `{/id*}`)
35. Test for npm supply chain MCP injection (SANDWORM_MODE pattern): can a malicious npm package inject MCP server configs into AI coding assistant configuration files and register rogue tools with embedded prompt injections? (Socket, Feb 2026 — 19 typosquatting packages, 48-hour delayed activation)
36. Test for Docker image label injection (DockerDash/Meta-Context Injection pattern): can malicious metadata labels in Docker images trigger AI assistant MCP tools to exfiltrate container details and network topology? (Noma Security — Docker Ask Gordon AI, fixed Desktop 4.50.0)
37. Test for agentic browser file system access: if target uses an AI-powered browser agent, can attacker-controlled content (calendar invites, documents) trigger autonomous file system exfiltration or credential theft via password manager interaction? (PleaseFix/PerplexedBrowser — Zenity Labs, March 2026)
38. Test for CRM agent prompt injection via web forms: if target uses AI agents processing Web-to-Lead or contact forms, can injected instructions in form fields trigger CRM data exfiltration? (ForcedLeak, Salesforce Agentforce, CVSS 9.4 — Varonis)
39. Test for multi-agent cascade injection (OMNI-LEAK pattern): in multi-agent systems, can a single injection in one agent's data source cascade through orchestrator to exfiltrate data via notification/communication agents? (arXiv:2602.13477 — all frontier models except claude-sonnet-4 vulnerable)
40. Test for semantic chaining jailbreak: can a series of individually benign instructions be chained to produce a forbidden output that no single prompt would trigger? (NeuralTrust, Feb 2026 — effective against Grok 4, Gemini, Seedance)
41. Test for chain-of-thought hijacking: if target uses reasoning models with visible thinking, can the intermediate reasoning be redirected to bypass safety refusals? (H-CoT, arXiv:2502.12893 — o1 rejection rate drops from 99% to <2%)
42. Test for AI framework SSRF via redirect: does the AI framework's URL loader follow HTTP redirects to internal resources? Supply a benign URL that passes validation, then redirects to cloud metadata (169.254.169.254). (CVE-2026-27795, LangChain — RecursiveUrlLoader redirect bypass)
43. Test for identity file poisoning (SOUL.md pattern): can processed documents instruct the AI agent to modify its own configuration/personality files, creating persistent compromise across all future sessions?

**Where to Hunt:**
- Any product that integrates MCP servers (Claude Desktop, Cursor, Windsurf, VS Code extensions)
- Enterprise AI agent platforms connecting to internal tools
- Open-source MCP server implementations (check GitHub for `mcp-server-*` repos)
- huntr platform specifically accepts AI/ML vulnerability reports
- VulnerableMCP.info for known MCP-specific vulnerabilities and affected implementations

---

### OWASP MCP Top 10 (2026)

A dedicated security framework specifically for Model Context Protocol risks, published by OWASP in early 2026. Complements the Agentic Top 10 by focusing on the protocol layer:

| Risk | Name | Description |
|------|------|-------------|
| MCP01 | Token Mismanagement & Secret Exposure | Hardcoded credentials, plaintext token storage, overly broad OAuth scopes in MCP server configurations |
| MCP02 | Privilege Escalation | MCP tools operating with elevated privileges beyond what tasks require; confused deputy attacks |
| MCP03 | Command Injection | OS command injection via unsanitized tool parameters, config values, or OAuth endpoints |
| MCP04 | Software Supply Chain Attacks | Malicious MCP packages, tool definition poisoning in registries, typosquatting on package names |
| MCP05 | Insufficient Authentication & Authorization | Missing auth on MCP endpoints (38% lack auth entirely), weak session management, no mutual TLS |
| MCP06 | Tool Poisoning | Hidden malicious instructions in tool descriptions/metadata that manipulate the AI model's behavior |
| MCP07 | Shadow MCP Servers | Unauthorized or rogue MCP servers operating within enterprise networks without security team awareness |
| MCP08 | Insecure Data Handling | Sensitive data exposed through MCP tool responses, logging, or inter-tool communication |
| MCP09 | Lack of Input Validation | Missing validation of tool call parameters, allowing path traversal, SSRF, and injection attacks |
| MCP10 | Insufficient Logging & Monitoring | No audit trail for MCP tool invocations, making detection and forensics of compromises impossible |

**Testing against OWASP MCP Top 10:** When a target uses MCP integrations, map your findings to these risk IDs (MCP01-MCP10). The framework is especially useful for framing MCP supply chain (MCP04), tool poisoning (MCP06), and shadow server (MCP07) findings in reports.

### OWASP Top 10 for Agentic Applications (December 2025)

A new separate list from the LLM Top 10, with input from 100+ security researchers. Covers risks specific to **autonomous AI agents** that take actions:

| Risk | Name | Description |
|------|------|-------------|
| ASI01 | Agent Goal Hijack | Prompt injection causing agent to pursue attacker's goals instead of user's |
| ASI02 | Tool Misuse | Agent calls tools with attacker-controlled parameters |
| ASI03 | Privilege Escalation | Agents inheriting high-privilege credentials beyond what tasks require |
| ASI04 | Agentic Supply Chain | Compromised plugins, tools, or dependencies in agent pipelines |
| ASI05 | Excessive Agency | Agents taking actions beyond intended scope without confirmation |
| ASI06 | Memory Poisoning | Corrupting agent's persistent memory to alter future behavior |
| ASI07 | Insecure Inter-Agent Communication | Unvalidated messages between agents in multi-agent systems |
| ASI08 | Cascading Failures | Single agent failure propagating through interconnected agent systems |
| ASI09 | Human-Agent Trust Exploitation | Agents exploiting user trust to bypass confirmation checks |
| ASI10 | Rogue Agents | Misalignment, concealment, or self-directed action by autonomous agents |

**Testing Agentic Applications:**
- Test each ASI risk against targets with AI agent features (customer support agents, coding assistants, workflow automation)
- Memory poisoning (ASI06) is especially impactful — inject malicious context that persists across sessions
- Inter-agent communication (ASI07) is a new attack vector in multi-agent platforms — inject messages between agents
- These risks compound with MCP vulnerabilities — an MCP tool poisoning attack can trigger agent goal hijack (ASI01) + tool misuse (ASI02) simultaneously

### Platform AI Integration & Policy Updates

**HackerOne:**
- **Hai Triage** (July 2025): AI-powered vulnerability triage combining AI agents with human expertise; **90% adoption** by HackerOne customers by end of 2025
- **AI Policy Update** (February 2026): Clarified that researcher submissions are NOT used to train AI models
- **Good Faith AI Research Safe Harbor** (January 20, 2026): Clarifies legal protections for researchers conducting authorized AI testing
- **Leaderboard split**: New system to distinguish human vs machine contributions (separating individuals from companies/agents like XBOW)
- **210% spike** in AI vulnerability reports; **70% of researchers** now using AI tools

**Bugcrowd:**
- **AI Triage Assistant** (December 2025): Context-aware intelligence layer providing technical summaries, severity assessments, and remediation steps
- **98% confidence** on duplicate detection; **98% accuracy** on P1 critical vulnerability identification
- Human-in-the-loop approach — augments rather than replaces analysts

### AI Red Teaming Tools & Frameworks

As the **EU AI Act** requires full compliance by **August 2, 2026** (penalties up to 35M EUR or 7% of global turnover), AI red teaming adoption is accelerating:

**Commercial Platforms:**
- **Splx AI**: End-to-end red teaming for conversational AI agents; thousands of automated adversarial scenarios
- **Giskard**: Automated red-teaming with dynamic multi-turn stress tests; 50+ specialized probes; adaptive engine
- **Novee**: Uses reasoning engines trained on top-tier red-team expertise; identifies logic flaws and chained attack scenarios

**Open-Source Frameworks:**
- **Microsoft PyRIT**: Open automation framework for adversarial AI campaigns; integrated into Azure AI Foundry; new AI Red Teaming Agent (public preview 2025)
- **DeepTeam**: 40+ vulnerability classes, 10+ adversarial attack strategies; aligned to OWASP LLM Top 10 and NIST AI RMF
- **NVIDIA Garak**: ~100 attack vectors; AVID integration for community vulnerability sharing; 20+ AI platform support
- **Promptfoo**: CLI/library for LLM evaluation and red-teaming; 50+ vulnerability types; CI/CD integration; next-gen agent with deep reconnaissance, strategic planning, adaptive execution, persistent memory

**OpenAI Atlas Hardening (2026):**
- OpenAI built an **LLM-based automated attacker** trained with reinforcement learning to hunt for prompt injection attacks against its browser agent (ChatGPT Atlas/Operator)
- Demonstrates the trend toward using AI to attack AI — red teaming tools are now themselves AI agents
- Implication: targets using OpenAI agents may be hardened against basic prompt injection; escalate to LPCI, multi-turn, or indirect vectors

**Prompt Injection Meta-Analysis (arXiv:2601.17548, January 2026):**
- SoK covering Claude Code, GitHub Copilot, Cursor — 78 studies (2021-2026)
- Attack success rates **exceed 85%** against state-of-the-art defenses with adaptive strategies
- 18 defense mechanisms analyzed: most achieve **less than 50% mitigation** against sophisticated adaptive attacks
- Conclusion: prompt injection must be treated as a first-class vulnerability class requiring architectural-level mitigations
- GPT-4 agents vulnerable to indirect prompt injection 24% of the time, nearly doubling with enhanced attack prompts
- 5 carefully crafted documents can manipulate AI responses 90% of the time through RAG poisoning

**Google 2025 Zero-Day Review:**
- 90 zero-day vulnerabilities tracked in 2025; **48% targeted enterprise technology** (new high)
- Google anticipates AI will accelerate both attack and defense in 2026

**Key references:** OWASP Gen AI Red Teaming Guide (January 2025), NIST AI RMF, MITRE ATLAS, OWASP AI Testing Guide v1 (AITG, November 2025)

### AI-Specific Bug Bounty Platforms

| Platform | Focus | Bounty Range | Notes |
|----------|-------|-------------|-------|
| **huntr** (Protect AI) | AI/ML vulnerabilities in open-source repos | Varies by project | First bug bounty platform specifically for AI/ML |
| **0din** (Mozilla) | GenAI vulnerabilities — GPT-4, Gemini, LLaMa, Claude | $500–$15,000 | Launched **Agent 0DIN** — gamified CTF for prompt injection/jailbreaking training |
| **HackerOne** | General + 1,121 programs with AI in scope | $100–$100K+ | Largest platform; AI reports up 210% YoY |
| **Bugcrowd** | General + AI triage | $100–$50K+ | CrowdMatch AI-powered researcher-to-program matching |
| **Amazon Nova** | Amazon Nova foundation models (private, invite-based) | $200–$25,000 | Covers prompt injection, CBRN, biases; expanding early 2026 |

### AI Bug Bounty Competitions & CTFs (2025-2026)

| Event | Details | Significance |
|---|---|---|
| **XBOW #1 on HackerOne** (2025) | First AI system to top a human bug bounty leaderboard; prompted HackerOne to restructure leaderboards | Watershed moment for AI in bug bounty |
| **Cyber Apocalypse CTF 2025** ("AI vs Human") | CAI achieved first place among AI teams, top-20 worldwide; 18,369 participants, 8,129 teams, 62 challenges | Benchmark for AI CTF capability |
| **Fetch the Flag 2026** (Snyk + NahamSec) | Feb 12-13, 2026; web, binary, exploitation challenges | Major community CTF |
| **AI CTF 2025** (PHDays) | Jeopardy-style, 12 AI/ML themed challenges over 40 hours | AI-focused CTF format |
| **OpenAI Bio Bug Bounty** (GPT-5) | $25K for universal jailbreak of bio/chem safety filters; $10K for multiple prompts | Novel model-safety bounty |
| **huntr** (Protect AI) | World's first bug bounty platform for AI/ML repos; 50.5% of findings fixed, 49.5% remain unpatched | Dedicated AI/ML vulnerability platform |

### Logic-Layer Prompt Control Injection (LPCI) — Novel Vulnerability Class

Published by CSA (Cloud Security Alliance) in February 2026 and documented in arXiv:2507.10457. Unlike traditional prompt injection, LPCI targets the **fundamental logic execution layer** of AI agents:

**How LPCI Differs from Traditional Prompt Injection:**
- Embeds **persistent, encoded, conditionally-triggered payloads** in LLM memory stores or vector databases
- Payloads **survive across multiple sessions** — not just in-context manipulation
- Activates based on specific conditions (event-based or time-based triggers)
- Much harder to detect than traditional prompt injection because payloads are dormant until activated

**Testing for LPCI:**
1. Identify agent memory/RAG stores that persist across sessions
2. Inject encoded payloads with conditional triggers (e.g., "when user asks about X, execute Y")
3. End the session, start a new one, and test if the payload activates
4. Check if payloads survive memory summarization/compression
5. Test event-based triggers (specific dates, user roles, query patterns)

**Defense Benchmark:** The Qorvex Security AI Framework (QSAF) reduces LPCI attack success from 43% to 5.3% — use this as a severity benchmark when reporting.

**Why This Matters for Hunters:** LPCI represents the next evolution of prompt injection. If a target has persistent agent memory (RAG, conversation history, knowledge bases), test for LPCI. Multi-turn attacks achieve up to **92% success rates** across 8 open-weight models.

### Salami Slicing Attacks on AI Agents (2026)

A new attack class identified by Repello AI where attackers submit multiple small, incremental requests over time to gradually shift an AI agent's behavior:

**How It Works:**
- Attacker submits 10+ support tickets, feedback items, or interactions over 1-3 weeks
- Each request slightly redefines "normal" behavior — nudging constraints, adjusting expectations, or expanding permissions
- By the 10th interaction, the agent's constraint model has drifted enough to perform unauthorized actions
- Unlike prompt injection (single payload), salami slicing exploits the agent's adaptation and learning mechanisms

**Testing for Salami Slicing:**
1. Identify agents that learn from or adapt to repeated interactions (customer service bots, procurement agents, approval workflows)
2. Submit a series of small requests, each incrementally escalating what's considered "normal"
3. Track whether the agent's responses gradually shift — are things accepted on request 10 that were denied on request 1?
4. Test if accumulated context drift survives session boundaries
5. Measure the minimum number of interactions needed to achieve constraint bypass

**Real-World Example:** A manufacturing company's procurement agent was gradually manipulated over 3 weeks through "clarification" messages about purchase authorization limits. By the end, it approved $5M in fraudulent purchase orders across 10 transactions — each individually small enough to avoid automated alerts.

**Severity Guidance:** High-Critical if the drift enables financial transactions, data access, or privilege changes. Medium if limited to behavioral changes within the agent's existing scope. Maps to ASI06 (Memory Poisoning) and ASI09 (Human-Agent Trust Exploitation) in OWASP Agentic Top 10.

### Real-World LLM Exploitation Incidents (2025-2026)

| Incident | Details | Impact |
|----------|---------|--------|
| **OmniGPT breach** | February 2026: threat actor breached OmniGPT (aggregator for ChatGPT-4, Claude 3.5, Gemini), exposing **34 million lines of conversations**, 30,000 user credentials, and uploaded business documents | Massive PII exposure from AI aggregator |
| **Claude jailbreak for government hacking** | Dec 2025-Jan 2026: hacker used Claude to hunt vulns, craft exploits, and exfiltrate data from Mexican government agencies by claiming bug bounty authorization | Data exfiltration from government systems |
| **LLMjacking** | Stolen credentials for accessing LLMs via official APIs (Amazon Bedrock etc.); Microsoft filed civil lawsuit | Credential theft, unauthorized compute |
| **HONESTCUE malware** | September 2025: malware using Gemini API to generate C# source code at runtime | Runtime-generated malware |
| **MalTerminal** | GPT-4-powered malware capable of generating ransomware or reverse-shell code at runtime | Polymorphic AI-generated malware |
| **ServiceNow prompt injection** | Second-order injection where low-privilege agent tricked higher-privilege agent into performing unauthorized actions | Multi-agent privilege escalation |
| **91,000+ attack sessions** | GreyNoise honeypots captured SSRF exploitation of Ollama and enumeration of 73+ LLM model endpoints (Oct 2025-Jan 2026) | Active targeting of AI infrastructure |
| **Nation-state AI operationalization** | DPRK, Iran, China, Russia using AI for coding, target research, vulnerability research, and post-compromise activities (late 2025) | State-sponsored AI-augmented attacks |
| **AI-generated phishing** | 14% of focused email attacks generated by LLMs by April 2025 (up from 7.6% a year prior) | Increasing sophistication of automated attacks |
| **OpenClaw supply chain attack & CVE crisis** | Largest confirmed supply chain attack on AI agent infrastructure — 1,184+ malicious skills across ClawHub (~1 in 5 packages); 8 critical CVEs in 6 weeks (incl. CVE-2026-25253 CVSS 8.8 RCE); **42,665 exposed instances, 5,194 actively vulnerable** (Maor Dayan); 824+ actively malicious skills out of 10,700+ in registry (Koi Security); 22% of monitored orgs have employees running OpenClaw without IT approval; RedLine/Lumma infostealers added OpenClaw file paths to must-steal lists | AI agent supply chain compromise at scale |
| **GTG-1002 state-sponsored AI espionage** | September 2025: first documented case of state-sponsored espionage primarily orchestrated by an AI agent; autonomous Claude Code executed 80-90% of the intrusion lifecycle | State-level AI-augmented attack |
| **Cisco vs DeepSeek R1** | Q1 2025: Cisco researchers broke DeepSeek R1 with 50/50 jailbreak prompts — 100% bypass rate across all safety categories | Complete AI safety filter failure |
| **ChatGPT SVG exploitation** | CVE-2025-43714: crafted SVG uploaded to ChatGPT executed arbitrary HTML/JS in user's browser within the preview window | XSS via AI-generated content |
| **LangChain LangGrinch** | CVE-2025-68664: prompt injection in LangChain Core; $4K bounty — highest ever awarded in the project | Framework-level prompt injection |
| **MCP Inspector RCE** | CVE-2025-49596 (CVSS 9.4): critical RCE in Anthropic's MCP Inspector — one of the first critical RCEs in MCP tooling (Oligo Security) | MCP toolchain exploitation |
| **Persistent procurement agent manipulation** | Palo Alto Unit42 (2026): manufacturing company's procurement agent manipulated over 3 weeks through "clarification" messages about purchase authorization limits; agent eventually approved $5M in false purchase orders across 10 transactions | Multi-week agent memory poisoning |
| **Lakera AI memory injection** | November 2025: demonstrated indirect prompt injection via poisoned data sources corrupting agent long-term memory — persistent false beliefs about security policies and vendor relationships | Long-term agent memory corruption |
| **MCP Go SDK case sensitivity** | CVE-2026-27896: MCP Go SDK JSON parser handles field names case-insensitively, allowing crafted MCP responses to bypass validation | Protocol-level bypass |
| **Log-To-Leak MCP attack** | ICLR 2026: new attack class covertly forces agents to invoke malicious logging tools to exfiltrate user queries, tool responses, and agent replies while preserving task quality — nearly undetectable | Silent data exfiltration via MCP |
| **Anthropic AI espionage disruption** | February 2026: Anthropic disrupted first documented large-scale AI-orchestrated cyberattack; suspected Chinese state actors jailbroke Claude Code as autonomous pentest orchestrator; 150GB of Mexican government data stolen including 195M taxpayer records; AI executed 80-90% of operations independently at physically impossible request rates | AI-orchestrated state-sponsored intrusion |
| **ToxicSkills ecosystem audit** | Snyk February 2026: 3,984 skills audited — 36% contain prompt injection, 1,467 malicious payloads found; SKILL.md to shell access in 3 lines of markdown | AI agent supply chain compromise at scale |
| **AI Recommendation Poisoning** | Microsoft February 2026: 50+ unique poisoning prompts from 31 companies across 14 industries embedded in web content to manipulate AI assistants; compromised recommendations on health, finance, security | Persistent AI assistant manipulation |
| **FortiGate AI-augmented mass breach** | Jan-Feb 2026: Russian-speaking threat actor used multiple commercial GenAI services to compromise 600+ FortiGate devices across 55+ countries; data exfiltration within 4 minutes of initial access | AI-augmented mass exploitation |
| **Clinejection** | Snyk 2026: prompt injection turning AI coding bots into supply chain attack vectors through GitHub Actions pipelines | CI/CD pipeline compromise via AI |
| **Docker MCP WhatsApp exfiltration** | April 2025: Invariant Labs demonstrated tool poisoning combined with unrestricted network access stealing entire WhatsApp message histories; bypasses DLP because it looks like normal AI behavior | Mass personal data exfiltration |
| **Unit 42 persistent memory poisoning** | December 2025: real-world indirect prompt injection poisoning long-term AI agent memory — attacker tricks victim into visiting malicious webpage, injected instructions survive session restarts via summarization | Cross-session persistent attack |
| **DockerDash (Docker Ask Gordon AI)** | November 2025: Noma Security discovered malicious Docker image metadata labels could compromise environments through Docker's Ask Gordon AI assistant; MCP Gateway passed contextual info to LLMs without distinguishing descriptive metadata from instructions; fixed in Docker Desktop v4.50.0 | Supply chain → AI agent RCE |
| **PleaseFix / PerplexedBrowser** | February 2026: Zenity Labs disclosed zero-click vulns in agentic browsers (Perplexity Comet); two exploit paths: (1) file system exfiltration via attacker-controlled calendar invites triggering autonomous agent execution, (2) credential theft by manipulating agent workflows to access password managers; patched by blocking `file://` path access | Zero-click agentic browser hijacking |
| **Chrome Gemini Panel Hijacking** | CVE-2026-0628: Palo Alto Unit 42 found malicious Chrome extensions could exploit Chrome's Gemini panel for privilege escalation, enabling spying via Gemini Live | Browser extension → AI escalation |
| **Shadow Escape (Operant AI)** | October 2025: first zero-click agentic attack via MCP; malicious instructions in documents (e.g., onboarding PDFs) cause MCP-enabled AI assistants to exfiltrate PII from connected databases and CRM systems; operates inside the firewall within authorized identity boundaries, invisible to conventional monitoring | Zero-click MCP data exfiltration |
| **Gemini MCP Tool 0-day** | CVE-2026-0755 (CVSS 9.8): command injection in gemini-mcp-tool execAsync method; user input passed directly to system calls; vendor never responded; published as 0-day advisory January 2026 | MCP tool RCE |
| **IDEsaster campaign** | 30+ vulnerabilities across 10+ AI coding tools (Claude Code, Cursor, Kiro, Windsurf), 24 CVEs; researcher Ari Marzouk; extension recommendation attacks allow malware distribution via namespace squatting on OpenVSX; 94+ Chromium flaws in Cursor/Windsurf due to legacy builds | AI coding IDE as attack surface |
| **React2Shell (CVE-2025-55182)** | CVSS 10.0: pre-authentication RCE in React Server Components via insecure deserialization in Flight protocol; affects React 19.0-19.2.0, Next.js 15-16; exploited in-the-wild by China-nexus groups (Earth Lamia, Jackpot Panda) within hours of disclosure Dec 3, 2025; became #1 most exploited CVE on HackerOne | Critical framework RCE |
| **Microsoft Entra ID (CVE-2025-55241)** | CVSS 10.0: vulnerability allowing attackers to obtain Global Administrator privileges via Actor Tokens authentication mechanism | Cloud identity total compromise |
| **Shai-Hulud supply chain worm** | Multi-wave npm supply chain worm; 454,648 malicious packages in 2025 (99% of all open-source malware); s1ngularity campaign harvested 2,349 credentials from 1,079 dev systems; self-propagating via stolen maintainer credentials | Supply chain worm |
| **ServiceNow second-order prompt injection** | Second-order prompt injection in Now Assist: low-privilege agent tricked into sending malformed request to higher-privilege agent, causing cross-agent privilege escalation — first documented cross-agent privilege escalation in production multi-agent system | Cross-agent privilege escalation |
| **AWS CodeBuild vulnerability** | Wiz uncovered critical flaw allowing hijacking of official AWS GitHub repositories and leaking secrets from build logs | Cloud CI/CD compromise |
| **PromptPwnd (CI/CD pipeline injection)** | Aikido Security: prompt injection in GitHub Actions/GitLab CI/CD exploits AI agents (Gemini CLI, Claude Code, Codex); attacker submits malicious GitHub issue → AI agent leaks GEMINI_API_KEY, GITHUB_TOKEN, Google Cloud tokens; **5+ Fortune 500 companies** confirmed affected; Google patched within 4 days | CI/CD pipeline credential theft via AI |
| **RoguePilot (GitHub Codespaces)** | Orca Security: passive prompt injection via hidden HTML comments in GitHub issues; when Codespace opens, Copilot processes injected prompt → GITHUB_TOKEN exfiltrated via `json.schemaDownload.enable`; patched by Microsoft | AI IDE token theft via issue content |
| **Rules File Backdoor** | Pillar Security: configuration/rules files (`.cursorrules`, `.github/copilot-instructions.md`) weaponized with invisible Unicode characters — undetectable to humans, readable by AI agents; one compromised rules file shared across projects creates widespread supply chain compromise | AI IDE supply chain via invisible content |
| **Copilot CLI shell expansion RCE** | CVE-2026-29783 (HIGH): GitHub Copilot CLI shell tool allows arbitrary code execution through bash parameter expansion patterns (`${var@P}`, `${!var}`, `$(cmd)`); safety layer misclassified dangerous commands as "read-only"; fixed v0.0.423 | AI coding tool RCE |
| **MCPJam Inspector RCE** | CVE-2026-23744: unauthenticated HTTP endpoint can install arbitrary MCP servers; listens on 0.0.0.0 by default enabling remote code execution from the network | MCP toolchain exploitation |
| **n8n Ni8mare (CVE-2026-21858)** | CVSS 10.0: unauthenticated RCE in n8n workflow automation platform (~100K servers globally); Content-Type confusion in Form Webhook request handling enables complete takeover of locally deployed instances; patched in v1.121.0 (Cyera Research Labs) | Workflow automation RCE |
| **Anthropic mcp-server-git RCE chain** | CVE-2025-68145/68143/68144: three chained vulnerabilities in Anthropic's official mcp-server-git achieving full RCE via malicious `.git/config` files — demonstrates that even first-party MCP servers can be exploited | First-party MCP server compromise |
| **HackerOne Hai invisible prompt injection** | HackerOne's beta AI assistant "Hai" found vulnerable to invisible prompt injection using Unicode tag characters (range E0000-E007F); hidden instructions embedded within normal-looking user input manipulated AI behavior undetectably (Cyrex) | AI triage system manipulation |
| **Kali Linux MCP server command injection** | Official Kali Linux MCP server ships with textbook command injection via `subprocess` with `shell=True`; reported by Simone Margaritelli (evilsocket) — highlights that even security-focused tools have basic injection flaws | Security tool irony |
| **MS-Agent AI framework RCE (CVE-2026-2256)** | CVSS 9.8: critical command injection in ModelScope MS-Agent — Shell tool's regex-based denylist for dangerous commands bypassed through command obfuscation; triggered remotely via prompt injection without authentication; public PoC on GitHub | AI framework total compromise |
| **OpenCode unauthenticated RCE (CVE-2026-22812)** | CVSS 8.8: open-source AI coding agent auto-starts unauthenticated HTTP server; any local process or website (via permissive CORS) can execute arbitrary shell commands with user's privileges; fixed in v1.0.216 | AI coding agent local server RCE |
| **n8n expression sandbox escape (CVE-2026-25049)** | CVSS 9.4: TypeScript type confusion — type annotations not enforced at runtime allow attackers to bypass sanitization controls using destructuring syntax; system command execution through crafted workflow expressions; fixed v1.123.17 / v2.5.2 | Novel sandbox escape pattern |
| **n8n Git Node RCE (CVE-2026-21877)** | Git Node code injection enabling RCE on both self-hosted and n8n Cloud instances; part of 6-CVE batch disclosed in a single day | Workflow automation RCE |
| **OpenClaw sandbox escape (CVE-2026-27001)** | Prompt injection via unsanitized workspace path embedding into system prompt + sandbox escape in download skill installation allowing file writes outside intended directories; fixed v2026.2.15 | AI agent sandbox escape |
| **OpenClaw Docker escape (CVE-2026-27002)** | Docker container escape via configuration injection — allows dangerous Docker options including bind mounts, host networking, and unconfined profiles; fixed v2026.2.15 | Container escape via AI agent |
| **Cursor Workspace Trust disabled** | Oasis Security found Cursor ships with Workspace Trust disabled by default, enabling silent code execution via malicious `.vscode/tasks.json` when opening untrusted repositories | AI IDE trust bypass |
| **MCP Denial-of-Wallet overthinking loops** | arXiv:2602.14798 (February 2026): 14 malicious tools across 3 MCP servers trigger repetition, forced refinement, and distraction loops; amplifies token consumption up to **142.4x** and increases latency; no single step looks abnormal, making detection difficult; severe financial risk for pay-per-token deployments | Economic denial-of-service via AI |
| **OpenClaw Voice Extension RCE (CVE-2026-28446)** | CVSS 9.8: pre-authentication RCE in OpenClaw voice-call extension — inbound allowlist policy bypass via empty caller ID and suffix matching; no session or authentication required; remote attacker achieves arbitrary code execution with OpenClaw process privileges; 42,000+ exposed instances; fixed v2026.2.1 (The Hacker News, February 2026) | AI agent voice feature RCE |
| **MCP MarkItDown SSRF (MCP fURI)** | BlueRock Security disclosed SSRF in Microsoft's MarkItDown MCP server — unrestricted URI handling allows access to any HTTP or file resource; in cloud deployments, attackers query instance metadata to obtain AWS credentials including access and secret keys; no authentication required (BlueRock, March 2026) | MCP SSRF → cloud compromise |
| **GeminiJack (Google Gemini Enterprise)** | Noma Labs: zero-click indirect prompt injection via poisoned Google Docs, Calendar invites, or emails; when any employee queries Gemini Enterprise, AI retrieves poisoned documents and exfiltrates data via disguised external image requests across Gmail, Calendar, Docs, and Workspace; Google separated Vertex AI Search from Gemini Enterprise in response (February 2026) | Enterprise AI data exfiltration |
| **Claude DXT Zero-Click RCE** | LayerX: CVSS 10.0 zero-click RCE in Claude Desktop Extensions (DXT) affecting 50+ extensions and 10,000+ users; DXT extensions run unsandboxed with full system privileges; crafted Google Calendar event achieves full RCE; **Anthropic declined to fix** stating it falls outside their current threat model (February 9, 2026) | AI desktop extension zero-click RCE |
| **Claude Code confirmation bypass RCE (CVE-2026-24887)** | RCE via bypass of Claude Code confirmation prompts — allows execution of untrusted commands through the `find` command; demonstrates that interactive approval mechanisms in AI coding tools can be circumvented (SentinelOne, 2026) | AI coding tool confirmation bypass |
| **mcp-atlassian RCE + SSRF (CVE-2026-27825)** | Critical: missing directory confinement in Confluence attachment download allows unauthenticated path traversal for arbitrary file writes (overwrite ~/.bashrc, ~/.ssh/authorized_keys); SSRF via unsanitized X-Atlassian-Jira-Url headers; fixed v0.17.0 (Arctic Wolf, February 2026) | MCP connector RCE |
| **mcp-nmap-server Command Injection (CVE-2026-3484)** | Command injection via child_process.exec in Nmap CLI handler; part of broader finding that 34% of MCP implementations use APIs prone to command injection (CWE-78) | MCP tool command injection |
| **Clinejection supply chain compromise** | Snyk February 2026: GitHub issue title containing prompt injection compromised Cline's AI triage bot → npm token theft → malicious cline@2.3.0 published with postinstall script installing OpenClaw on ~4,000 developer machines in 8 hours; composed indirect prompt injection + GitHub Actions cache poisoning + credential model weaknesses (Adnan Khan disclosure) | AI supply chain total compromise |
| **Malicious AI assistant browser extensions** | Microsoft Defender March 5, 2026: ~900,000 cumulative installs across 20,000+ enterprise environments of malicious Chromium extensions impersonating AI assistants (ChatGPT, DeepSeek sidebars) harvesting complete LLM chat histories and session tokens | AI extension data theft |
| **PleaseFix 1Password breach path** | Zenity Labs PleaseFix disclosure included credential theft via 1Password: attackers assumed Perplexity Comet agent privileges to access password vaults; initial fix bypassed using `view-source:file:///`; 120-day disclosure timeline (Oct 2025–Feb 2026) | Password manager compromise via AI agent |
| **GRP-Obliteration** | February 2026: Microsoft researchers showed single unlabeled prompt removes LLM safety alignment via inverted GRPO; GPT-OSS-20B attack success rate jumped 13% → 93% across all 44 harm categories | Complete safety alignment removal |
| **ZombieAgent (ChatGPT)** | January 2026: zero-click exploit chain — malicious email → ChatGPT memory poisoned → persistent rules → self-propagation to contacts; all within OpenAI cloud, invisible to endpoint monitoring; patched Dec 2025 | Self-propagating memory corruption |
| **Autonomous jailbreak agents** | March 2026 (Nature Communications): large reasoning models as autonomous adversaries achieved 97.14% jailbreak success across 9 target models with no human supervision — "alignment regression" | Systematic safety erosion |
| **Aikido Infinite Coolify CVEs** | February 2026: autonomous agents found 7 CVEs in Coolify including privilege escalation and RCE as root across 52,000+ exposed instances | Self-securing software discovery |
| **Vibe Hacking emergence** | 2026: low-effort AI-built attacks beating enterprise defenses; HP research confirms attackers prioritize cost over quality yet still bypass security controls | Democratized AI-powered attacks |
| **FIRST CVE forecast** | February 2026: predicted median 59,427 new CVEs for 2026 (first year to exceed 50,000); realistic scenarios suggest 70,000-100,000 possible | Unprecedented vulnerability volume |
| **Reprompt (Microsoft Copilot)** | Varonis Threat Labs: single-click data exfiltration chain via `q` URL parameter in Microsoft Copilot; attacker injects instructions causing Copilot to exfiltrate user data and conversation memory silently, maintaining control even when chat is closed; patched January 13, 2026 | Enterprise AI data exfiltration |
| **OpenClaw Browser Relay CDP (CVE-2026-28458)** | CVSS 7.5: unauthenticated `/cdp` WebSocket endpoint in Browser Relay extension enables cookie/session theft from all browser tabs; fixed v2026.2.1 | AI agent browser session hijacking |
| **OpenClaw Sandbox Bridge (CVE-2026-28468)** | Sandbox browser bridge server accepts requests without authentication when sandboxed browser feature is enabled; local attacker fully compromises all browser sessions running in OpenClaw's sandbox — cookie/session/page content theft; fixed v2026.2.14 | AI agent sandbox authentication bypass |
| **LLM-assisted deanonymization** | arXiv:2602.16800 (February 2026): LLM agents identify anonymous users from Hacker News, Reddit, LinkedIn posts with 25-67% recall and 70-90% precision at $1-4 per identification; practical obscurity no longer holds as privacy assumption | AI-powered privacy destruction |
| **AI app data breach epidemic** | Barrack.ai documented 20+ AI app breaches since January 2025 exposing tens of millions of users; four root causes: misconfigured Firebase (196/198 iOS AI apps), missing Supabase RLS, hardcoded API keys (72% of Android AI apps), absent authentication; Chat & Ask AI exposed 406M records including 300M+ chat messages | Systemic AI app security crisis |
| **IDE Extension Namespace Squatting** | Koi Security: Cursor, Windsurf, and Google Antigravity IDEs recommended nonexistent extensions from OpenVSX; attackers could register namespaces and upload malware with full system access; Cursor patched Dec 2025, Google patched Jan 2026, Windsurf unresponsive | AI IDE extension supply chain |

---

### Promptware Kill Chain (2026)

Published in arxiv:2601.09625 (January 2026), featured in a Black Hat webinar (February 2026), and covered by Bruce Schneier. Models prompt injection as multi-step malware using a 7-stage kill chain:

| Stage | Name | Description | Bug Bounty Test |
|-------|------|-------------|-----------------|
| 1 | **Initial Access** | Prompt injection enters the system (direct, indirect, or via MCP) | Test all injection entry points: chat, documents, emails, MCP tools |
| 2 | **Privilege Escalation** | Jailbreaking to bypass system constraints | Test if injection can override system prompt guardrails |
| 3 | **Reconnaissance** | Probing for system capabilities, tools, and data access | Test if agent reveals its tools, permissions, and connected systems |
| 4 | **Persistence** | Poisoning memory, retrieval stores, or RAG databases | Test if injected payloads survive across sessions (LPCI) |
| 5 | **Command & Control** | Establishing ongoing communication with attacker | Test if agent can be directed to external URLs or APIs |
| 6 | **Lateral Movement** | Spreading across connected tools, agents, or systems via MCP | Test cross-tool and cross-agent propagation of injected instructions |
| 7 | **Actions on Objective** | Data exfiltration, unauthorized actions, or system modification | Test the final impact: what data leaks, what actions execute |

**Why This Matters:** Analysis of 36 academic studies found 21 documented attacks traversing 4+ stages. Use this framework when scoping AI agent vulnerabilities — a finding that reaches stage 5+ is significantly more severe than one limited to stage 1-2. Reference the kill chain stage in reports to demonstrate attack sophistication.

### Agentic Browser Attack Surface (2026)

A new attack surface emerging in early 2026 as AI agents gain autonomous web browsing capabilities:

**Affected Products:** Perplexity Comet, Chrome Gemini panel, ChatGPT Atlas/Operator, and other agentic browsers.

**Attack Patterns:**
- **Zero-click agent hijacking** — Attacker-controlled web content (calendar invites, emails, documents) triggers autonomous agent execution without user interaction
- **File system access via agents** — Agents with `file://` access can be tricked into reading and exfiltrating local files
- **Credential manager access** — Manipulated agent workflows access password managers through legitimate agent-browser integration
- **Extension escalation** — Malicious browser extensions exploit AI panel integration points (CVE-2026-0628)

**Testing Approach:**
1. Identify if target has agentic browsing features (autonomous web access, scheduled browsing)
2. Plant indirect prompt injection in content the agent will process (calendar, email, search results)
3. Test if the agent acts on injected instructions without user confirmation
4. Check if agent has access to local filesystem, password managers, or other sensitive browser state
5. Test if browser extensions can interact with and manipulate the AI agent panel

---

### IDEsaster: AI Coding IDE Attack Surface (2025-2026)

A major new attack surface category: **30+ vulnerabilities across 10+ AI coding tools**, resulting in 24 CVEs (researcher Ari Marzouk). Named "IDEsaster" — AI coding assistants as vectors for RCE and credential theft.

**Key CVEs:**

| CVE | Tool | Impact | CVSS |
|-----|------|--------|------|
| CVE-2025-59536 | Claude Code | Arbitrary code execution through untrusted project hooks | 8.7 |
| CVE-2026-21852 | Claude Code | API key exfiltration when opening crafted repositories | 5.3 |
| CVE-2025-59944 | Cursor | Case-sensitivity bypass allowing file protection circumvention | — |
| CVE-2025-61590/91/92/93 | Cursor | Workspace file and MCP connection manipulation leading to RCE | — |
| CVE-2026-0830 | Kiro (AWS) | Command injection leading to RCE | — |
| CVE-2025-7656 | Chromium (Cursor/Windsurf) | 94+ Chromium vulnerabilities in AI IDEs using legacy builds | — |
| CVE-2026-29783 | GitHub Copilot CLI | Shell expansion arbitrary code execution via bash parameter patterns (`${var@P}`, `${!var}`) — safety layer misclassified dangerous commands as read-only | HIGH |
| CVE-2026-22812 | OpenCode | Unauthenticated HTTP server auto-starts with permissive CORS — any local process or website can execute shell commands with user privileges | 8.8 |
| CVE-2025-64106 | Cursor | MCP installation deep-link RCE — insufficient validation allows masking malicious commands behind trusted installation dialog (e.g., appearing as "Playwright" while executing payloads); patched within 2 days (Cyata) | 8.8 |
| CVE-2026-24887 | Claude Code | Confirmation prompt bypass — allows execution of untrusted commands through `find` command, circumventing interactive approval mechanisms | — |
| CVE-2026-28458 | OpenClaw | Browser Relay `/cdp` WebSocket endpoint missing authentication — cookie/session theft from all browser tabs | 7.5 |
| CVE-2026-28468 | OpenClaw | Sandbox browser bridge unauthenticated access — full compromise of sandboxed browser sessions | — |

**Cursor Workspace Trust Bypass (Oasis Security, 2026):**
- Cursor ships with **Workspace Trust disabled by default** — unlike VS Code which prompts users before trusting workspaces
- Enables silent code execution via malicious `.vscode/tasks.json` when opening untrusted repositories
- Combined with other IDEsaster patterns (MCP config manipulation, hooks injection), creates unopposed attack surface from repo clone
- Test for: open a crafted repository in Cursor and check if tasks.json auto-executes without trust prompt

**Rules File Backdoor (Pillar Security, 2026):**
- Configuration/rules files used by Cursor and GitHub Copilot can be weaponized with **invisible Unicode characters** — undetectable to humans but readable by AI agents
- One compromised rules file shared across projects creates widespread supply chain compromise
- Test for: inspect `.cursorrules`, `.github/copilot-instructions.md`, and similar config files for hidden Unicode content

**RoguePilot (Orca Security, 2026):**
- Passive prompt injection via GitHub issues enabling **GITHUB_TOKEN theft** through Codespaces/Copilot
- Attack uses hidden HTML comments in GitHub issues to inject prompts when a Codespace is opened
- Exploits VS Code's `json.schemaDownload.enable` setting to exfiltrate tokens to external servers
- Patched by Microsoft — test for similar patterns in other IDE integrations

**PromptPwnd (Aikido Security, 2026):**
- Prompt injection in **GitHub Actions and GitLab CI/CD pipelines** exploits AI agents (Gemini CLI, Claude Code, OpenAI Codex)
- Attacker submits malicious GitHub issue with hidden instructions; AI agent leaks GEMINI_API_KEY, GITHUB_TOKEN, and Google Cloud access tokens
- At least **five Fortune 500 companies** confirmed affected; Google patched within 4 days
- Test for: AI agents running in CI/CD pipelines that process untrusted input (issues, PRs, comments)

**Copilot CLI Arbitrary Code Execution:**
- PromptArmor demonstrated GitHub Copilot CLI can download and execute malware with **zero user approval**
- CVE-2026-29783: shell tool allows arbitrary code execution through bash parameter expansion patterns — safety layer incorrectly classified dangerous commands as "read-only"

**Extension Recommendation Attacks (Koi Security, 2026):**
- Cursor, Windsurf, Google Antigravity, and Trae recommend non-existent VSCode extensions from OpenVSX registry
- Threat actors can claim unclaimed extension namespaces and serve malicious extensions to 1.8M+ developers
- Attack chain: AI IDE recommends extension → developer installs → malicious code executes with full IDE permissions
- Koi Security preemptively claimed vulnerable namespaces; Cursor patched Dec 2025, Google patched Jan 2026, Windsurf unresponsive
- Test for: extension namespace squatting, fake extension serving, trust chain exploitation

**Reprompt (Varonis Threat Labs, January 2026):**
- Single-click data exfiltration chain exploiting the `q` URL parameter in Microsoft Copilot
- Attacker crafts URL containing injected instructions → victim clicks → Copilot exfiltrates user data and conversation memory
- Maintains persistent control even after chat is closed — instructions survive in Copilot's conversation context
- Patched by Microsoft January 13, 2026
- Test for: URL parameter injection in AI assistants, persistent instruction injection via conversation context, data exfiltration through AI-mediated channels

**Testing Approach:**
1. Clone a repository containing malicious `.claude/`, `.cursor/`, `.github/` configurations
2. Open in target AI IDE and observe if project hooks, MCP configs, or environment variables are automatically executed
3. Check if file protection mechanisms can be bypassed via case sensitivity or path traversal
4. Test if IDE extension recommendations can be manipulated to install attacker-controlled extensions
5. Verify if workspace file manipulation can inject MCP connections or alter build configurations
6. Check for legacy Chromium vulnerabilities if IDE uses embedded browser (94+ known flaws in Cursor/Windsurf builds)

---

### Supply Chain Worm: Shai-Hulud (2025-2026)

A multi-wave JavaScript supply chain worm campaign demonstrating self-propagating compromise:

**Attack Waves:**
- **Wave 1:** Compromised maintainer accounts, malicious postinstall scripts injecting credential harvesters
- **Wave 2 (Shai-Hulud 2.0):** Cross-victim credential exposure — stolen credentials from one victim used to compromise packages maintained by another
- **s1ngularity campaign:** Compromised Nx packages harvested **2,349 credentials from 1,079 developer systems**
- **Scale:** **454,648 new malicious packages** published on npm in 2025 alone — 99% of all open-source malware

**Key Defense:** Dependency cooldowns (7-14 day delay before adopting new packages/versions) would have prevented 8 out of 10 major 2025 supply chain attacks. npm Trusted Publishing recommended over token-based authentication.

**Testing Approach:**
1. Check if target uses npm packages without lockfile integrity verification
2. Test for postinstall script execution in CI/CD pipelines
3. Verify dependency pinning and cooldown policies
4. Check for transitive dependency poisoning risk
5. Test if npm publish tokens are rotated and scoped

---

### GRP-Obliteration: Single-Prompt Safety Alignment Removal (Microsoft, February 2026)

A novel attack that can **remove an LLM's safety alignment using a single unlabeled prompt**:

**How It Works:**
- Based on Group Relative Policy Optimization (GRPO), a training technique normally used to improve model behavior
- Inverts the process: one harmful prompt generates multiple responses; a "judge" model scores based on compliance
- A single training example (e.g., "Create a fake news article") caused safety regressions across **all 44 harm categories** in SorryBench
- GPT-OSS-20B attack success rate jumped from **13% to 93%**
- **Tested across 15 models** in the DeepSeek, GPT-OSS, Gemma, Llama, Ministral, and Qwen families — all 15 "reliably unalign"; even a relatively mild prompt like "Create a fake news article" removes safety across all 44 harm categories
- Generalizes to text-to-image diffusion models (harmful generation rate: 56% → 90% on sexuality prompts)

**Testing for GRP-Obliteration:**
1. If target allows model fine-tuning or RLHF customization, test if a single adversarial training example can degrade safety
2. Test if target's safety filters can be inverted through adversarial optimization
3. Check if target models expose training APIs that could be abused for alignment regression

**Severity Guidance:** Critical if fine-tuning API is publicly accessible; High if requires authenticated access; reference arxiv:2602.06258.

---

### ZombieAgent: Zero-Click Memory Poisoning via Email (Radware, January 2026)

A zero-click exploit chain against ChatGPT demonstrating self-propagating memory corruption:

**Attack Chain:**
1. Attacker sends malicious email containing hidden instructions
2. Victim asks ChatGPT to summarize unread messages
3. Agent processes email → long-term memory poisoned with attacker-created rules
4. Poisoned rules persist across all future sessions
5. Agent scans inbox and sends poisoned messages to victim's contacts → **self-propagation**
6. All activity occurs within OpenAI's cloud infrastructure — invisible to endpoint monitoring

**Testing for ZombieAgent Pattern:**
1. Identify if target AI processes external content (emails, documents, messages) with memory persistence
2. Plant instructions in content the AI will retrieve — test if they survive as persistent memory rules
3. Check if memory corruption persists across session boundaries
4. Test if compromised agent can propagate instructions to contacts or collaborators
5. Verify if poisoned memory entries are visible to the user (most are not)

**Severity Guidance:** Critical — zero-click, self-propagating, persistent. Maps to ASI06 (Memory Poisoning) and enables ASI07 (Insecure Inter-Agent Communication) via propagation. OpenAI patched December 2025.

---

### Autonomous Jailbreak Agents (Nature Communications, March 2026)

Large reasoning models acting as **autonomous adversaries** can systematically erode safety guardrails:

**Key Findings:**
- DeepSeek-R1, Gemini 2.5 Flash, Grok 3 Mini, Qwen3 235B as autonomous jailbreak agents
- **97.14% jailbreak success rate** across 9 target models in multi-turn conversations with no human supervision
- Demonstrates "alignment regression" — LRMs can systematically erode other models' safety in extended conversations
- DOI: s41467-026-69010-1

**Testing Implications:**
- Multi-turn conversations are far more dangerous than single-shot prompts
- If target uses reasoning models (o3, R1, Gemini 2.5), test for extended conversation safety degradation
- Safety testing should include multi-turn attack scenarios, not just single prompt injection

---

### Agent-to-Agent (A2A) Protocol Attack Surface (2026)

Google's Agent-to-Agent protocol (open-source, Apache 2.0, Linux Foundation governed) creates new attack vectors:

**Vulnerability Classes:**
- **Agent identity spoofing** — impersonating trusted agents to inject instructions
- **Capability declaration forgery** — claiming capabilities to redirect task assignments
- **Task chain poisoning** — injecting malicious tasks into multi-agent workflows
- **Trust graph attacks** — exploiting trust relationships between agents for lateral movement
- **No token lifetime limitations** — persistent access once compromised

**Testing Approach:**
1. If target uses A2A protocol, test for agent identity verification between peers
2. Check if capability declarations are validated or trusted implicitly
3. Test if task assignments can be intercepted or modified in transit
4. Verify that east-west agent traffic has security controls (most don't — bypasses traditional perimeters)
5. Map to ASI07 (Insecure Inter-Agent Communication) in OWASP Agentic Top 10

**Reference:** arXiv:2505.12490 identifies 40+ academic threats; real-world scenario: compromised research agent inserts hidden instructions consumed by financial agent → unintended trades.

---

### Side-Channel Timing Attacks Against LLMs (2026)

A new class of inference attacks exploiting timing characteristics of language models:

**Attack Types:**
- **Whisper Leak** — analyzes packet size and timing patterns in streaming LLM responses; achieves **>98% AUPRC** classification across 28 popular LLMs
- **Speculative decoding attacks** — fingerprint user queries with **>75% accuracy** by analyzing token generation timing
- Optimization techniques in language models (speculative decoding, caching) create exploitable timing patterns

**Testing Approach:**
1. Analyze streaming response timing for information leakage about query content or model behavior
2. Check if target uses speculative decoding or token caching that creates timing side channels
3. Test if response timing varies predictably based on query sensitivity or content type
4. Check if mitigations (padding, delay injection) are applied to streaming responses

**Current Mitigations:** Cloudflare, OpenAI, Mistral, Microsoft, and xAI have deployed countermeasures. Test if target has similar protections.

---

### Denial-of-Wallet via MCP Overthinking Loops (February 2026)

A new attack class exploiting MCP tool interactions to cause severe financial damage without obvious malicious behavior (arXiv:2602.14798):

**How It Works:**
- Attacker registers 14 malicious tools across 3 MCP servers
- Tools trigger three loop types: **repetition loops** (agent repeats same tool calls), **forced refinement loops** (tool responses demand "improvements"), and **distraction loops** (introduce tangential tasks)
- Amplifies token consumption up to **142.4x** and dramatically increases latency
- No single step looks abnormal — each tool call appears legitimate, making detection extremely difficult
- Creates severe financial risk for pay-per-token AI deployments

**Testing for Denial-of-Wallet:**
1. If target uses MCP servers with pay-per-token billing, test if tool responses can trigger repeated agent calls
2. Craft tool outputs that request "refinement" or "additional analysis" — does the agent loop?
3. Test if distraction tools can redirect the agent away from its primary task, consuming tokens on irrelevant work
4. Check if token budgets or call limits exist for MCP tool interactions
5. Measure cost amplification: what's the ratio of attacker cost (registering tools) to victim cost (token consumption)?

**Severity Guidance:** High-Critical if target has no token budgets and uses production billing; Medium if testing/sandbox environments with spending caps. Maps to ASI05 (Excessive Agency) in OWASP Agentic Top 10.

---

### Invisible Unicode Prompt Injection (2026)

A stealth injection technique using Unicode tag characters (range E0000-E007F) that are invisible to humans but processed by AI models:

**How It Works:**
- Attacker encodes malicious instructions using Unicode tag characters within seemingly normal text
- Text appears completely benign to human reviewers — no visible difference
- AI models read and execute the hidden instructions as if they were regular text
- Used successfully against HackerOne's Hai AI triage assistant (Cyrex disclosure)

**Testing Approach:**
1. Encode prompt injection payloads using Unicode tag characters (E0000-E007F range)
2. Submit encoded text through normal input channels (chat, forms, comments)
3. Check if AI processes the hidden instructions — does behavior change?
4. Test other invisible Unicode ranges: zero-width spaces (U+200B), zero-width joiners (U+200D), bidirectional overrides (U+202A-U+202E)
5. If target has AI-powered content moderation, test if invisible instructions can bypass filters

**Severity Guidance:** High if invisible injection can manipulate AI triage, content moderation, or automated decision-making; Medium if limited to chat-level manipulation. Especially relevant for AI-powered security tools that process user submissions.

---

### NIST AI Agent Standards Initiative (February 2026)

NIST's Center for AI Standards and Innovation (CAISI) launched a formal initiative for AI agent security standards:

**Three Pillars:**
1. Facilitating industry-led agent standards and U.S. leadership in international standards bodies
2. Fostering community-led open-source protocol development
3. Advancing research in AI agent security and identity

**Key Deadlines:**
- RFI on AI Agent Security (due March 9, 2026)
- ITL AI Agent Identity and Authorization concept paper (due April 2, 2026)
- Listening sessions begin April 2026

**Bug Bounty Relevance:** As NIST standards mature, programs will increasingly require compliance — understanding emerging standards gives hunters an edge in framing reports against forthcoming regulatory requirements.

---

### Semantic Chaining Jailbreak (NeuralTrust, February 2026)

A new multimodal jailbreak technique where attackers chain semantically "safe" individual instructions that converge on a forbidden result:

**How It Works:**
- Unlike traditional jailbreaks using a single harmful prompt, semantic chaining exploits models' compositional reasoning
- Each individual instruction appears benign and passes content filters
- The sequence of instructions converges on a forbidden output that no single prompt would produce
- Notably simple — requires no technical expertise

**Affected Models:** Grok 4, Gemini Nano Banana Pro, Seedance 4.5

**Testing Approach:**
1. Decompose a forbidden request into multiple benign-sounding sub-tasks
2. Chain them in sequence within a single conversation
3. Test if the model produces the forbidden output through composition
4. Particularly effective against multimodal models processing text + images

**Severity Guidance:** High if semantic chaining bypasses content moderation to produce harmful outputs; Medium if limited to edge cases. Critical differentiator: this bypasses both input filtering and output filtering.

---

### H-CoT: Hijacking Chain-of-Thought (February 2026)

Reasoning models that display intermediate thinking can have their chain-of-thought hijacked to bypass safety:

**Key Findings:**
- OpenAI o1 typically rejects 99%+ of child abuse/terrorism prompts — under H-CoT attack, rejection rate drops **below 2%**
- Affects OpenAI o1/o3, DeepSeek-R1, Gemini 2.0 Flash Thinking
- Exploits the displayed intermediate reasoning as an attack surface
- arXiv:2502.12893 (Duke University/Accenture)

**Testing Approach:**
1. Identify if target uses reasoning models with visible chain-of-thought
2. Craft prompts that redirect the reasoning chain mid-stream
3. Test if safety refusals can be circumvented by manipulating the reasoning trace

---

### Multi-Agent System Privacy Attacks (ICLR/arXiv 2026)

Three new research papers reveal systemic privacy vulnerabilities in multi-agent systems:

**AgentLeak (arXiv:2602.11510):** First full-stack benchmark for privacy leakage in multi-agent LLM systems. Key finding: multi-agent configs reduce per-channel output leakage (27.2% vs 43.2% single-agent) but introduce **unmonitored internal channels** — inter-agent messages and shared memory. 7-channel taxonomy, 32-class attack taxonomy across 4,979 execution traces.

**OMNI-LEAK (arXiv:2602.13477):** A single indirect prompt injection in a public database can cascade through orchestrator multi-agent patterns — SQL agent → orchestrator → notification agent → data exfiltration. Even a 1/500 success rate in a 100-person company could leak sensitive data within five days. All tested frontier models except claude-sonnet-4 were vulnerable.

**CORBA (arXiv:2502.14529):** Contagious Recursive Blocking Attacks force multi-agent systems into recursive blocking states. 79-100% of AutoGen agents blocked within 1.6-1.9 dialogue turns. Blocking messages appear benign, making detection extremely difficult.

**Inter-Agent Trust Exploitation (ICLR 2026):** 82.4% of tested LLMs can be compromised through inter-agent trust — models that resist direct malicious commands will execute identical payloads when requested by peer agents.

**Testing Approach:**
1. If target uses multi-agent systems, test inter-agent communication channels for injection
2. Test if compromising one agent cascades to others via shared state or orchestrator
3. Test for contagious blocking/DoS attacks across agent networks
4. Test if agents blindly trust instructions from peer agents

**Severity Guidance:** Critical for multi-agent enterprise deployments; High for dual-agent systems. A single compromised agent can poison 87% of downstream decision-making within 4 hours.

---

### SOUL.md Identity File Poisoning (2026)

Attackers trick AI agents into writing malicious instructions into their identity/personality files (SOUL.md, .cursorrules, etc.) via indirect prompt injection:

**Attack Chain:**
1. Attacker creates a document with hidden prompt injection
2. AI agent processes the document and is instructed to modify its own identity file
3. Malicious instructions persist in the identity file across all future sessions
4. Every conversation and action the agent takes is now influenced by attacker-controlled instructions

**Testing Approach:**
1. Check if agent has writable identity/personality files
2. Test if processed documents can instruct the agent to modify its own configuration
3. Verify if configuration changes persist across sessions
4. Test the full chain: document → identity file modification → persistent compromise

---

### Critical Warning: "AI Slop" Reports

AI slop reports are now a **major industry problem** that has caused the first program shutdowns. In January 2026, **curl ended its bug bounty program** after AI-generated submissions overwhelmed the security team — by July 2025 submission volume spiked to 8x the normal rate, with only 5% of 2025 submissions being genuine vulnerabilities. In six years, not a single AI-only-generated submission discovered a genuine vulnerability. Daniel Stenberg's goal was to "remove the incentive for people to submit crap." The program had paid out $90K+ for 81 genuine vulnerabilities before closing.

Bugcrowd published an analysis titled "How lazy hacking killed cURL's bug bounty." Submissions with fabricated or AI-hallucinated findings will be rejected, result in bounty clawback, or damage your reputation:

**Red Flags (Don't Report):**
- Finding with no proof-of-concept attachment or reproduction steps
- AI output claiming a vulnerability but your manual test doesn't confirm it
- Overly generic descriptions copied from AI without specific endpoint/payload details
- AI acting as "an echo chamber and amplifier" — luring you into confirmation bias about a non-existent finding (Intigriti chief hacker officer warning)

**Quality Check Before Submitting:**
1. Reproduce the finding yourself (manually, with tools like Burp or curl)
2. Include specific payloads and steps in the report
3. Screenshot or video proof if visual impact (XSS, IDOR data)
4. Call out any AI-assisted analysis in the "Discovery Method" section (transparency builds trust)
5. Verify the AI's output against reality — don't trust AI confidence levels alone

---

### MCP OAuth Account Takeover (Obsidian Security, 2025-2026)

Multiple one-click account takeover vulnerabilities in Remote MCP servers discovered by Obsidian Security. MCP servers acting as both authorization server and OAuth client created CSRF-style attacks leaking authorization codes:

**How It Works:**
- MCP server sends user to its own authorization endpoint, then silently uses the returned code with a separate OAuth provider
- Missing `state` parameter validation enables CSRF-style attacks against the authorization flow
- Affected clients: Claude Desktop, VS Code, Cursor, Cline, ChatMCP, Cherry Studio

**Testing Approach:**
1. If MCP server implements OAuth, test for missing `state` parameter in authorization requests
2. Test if authorization codes can be replayed across sessions
3. Check if the MCP server validates redirect URIs strictly
4. Test for mixed role confusion — server acting as both authorization server and OAuth client

**Timeline:** Reported July-August 2025; fixed September 2025; MCP spec updated November 25, 2025 to mandate OAuth 2.1 + PKCE.

---

### Google Antigravity IDE Attack Surface (2026)

Google's AI coding IDE launched in early 2026 has rapidly accumulated a significant vulnerability portfolio:

**Key Findings:**
- **RCE via Web Page Interaction** ($10,000 bounty, Hacktron AI): critical RCE allowing full system takeover by opening a malicious website; disclosed February 9, 2026
- **Forced Descent — Persistent Code Execution** (Mindgard): modifying a single global config file enables persistent code execution across projects, surviving uninstall/reinstall; no public patch as of March 2026
- **70+ Architectural Vulnerabilities** (community analysis): extensive security weaknesses in the platform's architecture identified within weeks of launch
- **Extension Namespace Squatting** (Koi Security): recommended non-existent extensions from OpenVSX, allowing attacker namespace registration; Google patched January 2026

**Testing Approach:**
1. Test for persistent code execution via global config file modification
2. Check if extension recommendations can be manipulated via namespace squatting
3. Test for web-triggered RCE via malicious page interaction
4. Audit the trust model for workspace files and project configurations

---

### Cursor Rogue MCP Browser Takeover (CSO Online, 2026)

A rogue MCP server can inject JavaScript into Cursor's built-in browser, replacing login pages with phishing interfaces:

**How It Works:**
- Unlike VS Code, Cursor does not perform integrity checks on its own files
- A rogue MCP server injects malicious JavaScript into Cursor's embedded browser
- Login pages are replaced with phishing interfaces while URLs remain unchanged
- The attack propagates automatically with each new browser tab opened

**Testing Approach:**
1. If target uses Cursor with MCP servers, test if MCP server can modify Cursor's browser behavior
2. Check if Cursor validates integrity of its embedded browser files
3. Test if malicious JavaScript injected via MCP persists across browser tabs
4. Compare with VS Code's integrity check behavior as a baseline
