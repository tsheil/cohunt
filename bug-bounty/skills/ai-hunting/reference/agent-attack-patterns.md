# Agent & AI System Attack Patterns

Architectural attack patterns targeting AI agents, coding assistants, multi-agent systems, and agentic browsers. Covers OWASP Agentic Top 10, supply chain attacks, IDE exploitation, and autonomous agent threats. For individual jailbreak/injection techniques (LPCI, ZombieAgent, Policy Puppetry, etc.), see [llm-attack-techniques.md](llm-attack-techniques.md).

> **Related files:** [llm-attack-techniques.md](llm-attack-techniques.md) for 18 novel attack techniques with test procedures | [mcp-playbooks.md](mcp-playbooks.md) for MCP-specific test procedures | [ai-case-studies.md](ai-case-studies.md) for real-world incident case studies

---

## Table of Contents

- [OWASP Top 10 for Agentic Applications (ASI01-ASI10)](#owasp-top-10-for-agentic-applications)
- [Agent Skill Supply Chain Attacks](#agent-skill-supply-chain-attacks)
- [IDE & Supply Chain Attack Surface (routing)](#ide--supply-chain-attack-surface)
- [Agentic Browser Attack Surface](#agentic-browser-attack-surface)
- [Multi-Agent System Attacks](#multi-agent-system-attacks)
- [Agent-to-Agent (A2A) Protocol Attack Surface](#agent-to-agent-a2a-protocol-attack-surface)
- [Promptware Kill Chain](#promptware-kill-chain)
- [Novel Attack Techniques (routing)](#novel-attack-techniques)
- [Full Schema Poisoning (FSP)](#full-schema-poisoning-fsp)
- [Supply Chain Worm: Shai-Hulud](#supply-chain-worm-shai-hulud)
- [Phantom: Template-Based Agent Hijacking](#phantom-template-based-agent-hijacking)
- [AI-as-Exfiltration-Channel](#ai-as-exfiltration-channel-unified-pattern-class)

---

## OWASP Top 10 for Agentic Applications

A new separate list from the LLM Top 10 (December 2025), with input from 100+ security researchers. Covers risks specific to **autonomous AI agents** that take actions:

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

**Testing Agentic Applications — Procedures by Risk:**

| # | ASI Risk | Test Procedure |
|---|----------|---------------|
| T1 | ASI01: Goal Hijack | Inject "Ignore prior instructions" in content agent retrieves (docs, emails, issues, wiki, GitHub issues) |
| T2 | ASI02: Tool Misuse | Shell metacharacters/SQLi in tool params; inject extra API params ("Also set admin=true") |
| T3 | ASI03: Privilege Escalation | Map agent credentials; attempt cross-scope/cross-system actions beyond granted token scope |
| T4 | ASI04: Supply Chain | Audit plugins for known vulns; check auto-install from untrusted sources; rug pull detection |
| T5 | ASI05: Excessive Agency | Request destructive actions; verify confirmation gates; observe unrequested scope creep |
| T6 | ASI06: Memory Poisoning | Inject persistent instructions → new session activation; test cross-user shared memory/RAG |
| T7 | ASI07: Inter-Agent Comms | Cross-agent instruction injection; forge messages from trusted agents; check identity validation |
| T8 | ASI08-10: Cascade/Trust/Rogue | Error propagation; confirmation fatigue; self-directed behavior; audit log discrepancies |

**Compounding effects:** MCP tool poisoning can trigger ASI01 (goal hijack) + ASI02 (tool misuse) simultaneously. Memory poisoning (ASI06) combined with excessive agency (ASI05) creates persistent automated compromise. Always test combinations, not just individual risks.

> **OWASP MCP Top 10 (protocol-layer risks):** See [mcp-playbooks.md](mcp-playbooks.md)

---

## Agent Skill Supply Chain Attacks

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
- **36% contain prompt injection**; **1,467 malicious payloads** found across **76 confirmed malicious skills**
- 13.4% (534 skills) contain critical-level issues
- 100% of confirmed malicious skills contain malicious code patterns; 91% simultaneously employ prompt injection
- 2.9% of skills dynamically fetch and execute content from external endpoints at runtime — attackers can modify behavior at any time
- SKILL.md to shell access achievable in **three lines of markdown**

**Clinejection (Snyk, 2026):**
- Prompt injection used to turn AI coding bots into supply chain attack vectors through GitHub Actions pipelines
- Malicious PR content -> AI coding agent processes -> injected code merged or pipeline compromised

**SANDWORM_MODE MCP Injection (Socket, February 2026):**
- npm supply chain worm using 19 typosquatting packages with "McpInject" module
- Deploys malicious MCP server into AI coding assistant configs (Claude Code, Cursor, Windsurf)
- Rogue MCP tools embed prompt injections to steal SSH keys, AWS credentials, .env files
- 48-hour delayed activation with per-machine jitter
- Pattern: npm install -> MCP injection -> credential theft

**ContextCrush (Noma Labs, February 2026):**
- Supply chain vulnerability in Context7 MCP server providing library documentation
- Attackers inject malicious instructions through "Custom Rules" feature
- Rules served verbatim through MCP server with no sanitization
- Pattern: trusted documentation -> MCP delivery -> agent execution

**Testing Agent Skills:**
1. Before installing any third-party skill, scan with at least one of the scanning tools (see [mcp-playbooks.md](mcp-playbooks.md#mcp-security-scanning-tools))
2. Read the SKILL.md file manually — look for environment variable references, external URL fetching, trigger conditions
3. Check if skill dynamically fetches content from external endpoints (2.9% of ClawHub skills do this)
4. Test if skill instructions can be manipulated by content the agent processes
5. Monitor for post-installation behavior changes (rug pull pattern)

---

## IDE & Supply Chain Attack Surface

> **Full coverage (28+ CVEs, IDEsaster, Claude DXT RCE, AI-as-C2 proxy, Google Antigravity):** See [ide-supply-chain.md](ide-supply-chain.md)

Key patterns: workspace trust bypass (Cursor), rules file backdoor (invisible Unicode), RoguePilot (GitHub issue → symlink + $schema exfil → repo takeover), CVE-2026-29783 (Copilot CLI bash parameter expansion RCE), PromptPwnd (CI/CD pipeline injection), CamoLeak (CVE-2025-59145 CVSS 9.6 — zero-click private code exfil via Camo proxy), extension namespace squatting, Claude DXT zero-click RCE (CVSS 10.0), AI web-browsing as C2 channel.

---

## Agentic Browser Attack Surface

A new attack surface emerging in early 2026 as AI agents gain autonomous web browsing capabilities:

**Affected Products:** Perplexity Comet, Chrome Gemini panel, ChatGPT Atlas/Operator, and other agentic browsers.

**Trail of Bits Trust Zone Taxonomy (February 2026):**
A simplified trust zone violation model for agentic browsers, from Trail of Bits' audit of Perplexity Comet (4 prompt injection techniques found):
- **INJECTION** — untrusted input injected into AI agent context
- **CTX_IN** — sensitive browsing data added to chat context (e.g., Gmail contents reaching the AI)
- **CTX_OUT** — chat context leaked into external requests (e.g., AI includes private data in web requests)
- **REV_CTX_IN** — chat context affects browsing origins (e.g., AI modifies which pages are visited based on injected instructions)

Use this taxonomy when categorizing agentic browser vulnerabilities in reports — maps directly to CWE categories and helps triagers understand the trust boundary violation.

**Attack Patterns:**
- **Zero-click agent hijacking** — Attacker-controlled web content (calendar invites, emails, documents) triggers autonomous agent execution without user interaction
- **File system access via agents** — Agents with `file://` access can be tricked into reading and exfiltrating local files
- **Credential manager access** — Manipulated agent workflows access password managers through legitimate agent-browser integration
- **Extension escalation** — Malicious browser extensions exploit AI panel integration points (CVE-2026-0628)
- **PleaseFix zero-click exfiltration** — Zenity Labs (March 2026): two exploit paths in Perplexity Comet: (1) zero-click file system exfiltration via calendar invite triggers, (2) credential theft via 1Password manipulation through agent-authorized workflows. Initial fix bypassed with `view-source:file:///`. Pattern: routine workflow triggers (calendar invites) can weaponize agentic browsers without user interaction. Key insight: no exploit, no user clicks, no explicit request for sensitive actions — the agent's own permissions are the attack surface
- **OpenAI Lockdown Mode** (February 2026): OpenAI acknowledged prompt injection in AI browsers "may never be fully patched" and launched Lockdown Mode for ChatGPT — confirms the fundamental architectural vulnerability

**Adaptive Prompt Injection (March 2026):**
Research demonstrates adaptive attacks bypass 12 recent prompt injection defenses with **90%+ success** using gradient descent, RL, random search, and human-guided exploration. Techniques: cross-modal attacks (hidden instructions in images accompanying benign text), context poisoning (gradual manipulation of conversation history for delayed activation), steganographic injection (invisible text in metadata fields). Implication: any defense relying on static filtering or pattern matching will be bypassed.

**Testing Approach:**
1. Identify if target has agentic browsing features (autonomous web access, scheduled browsing)
2. Plant indirect prompt injection in content the agent will process (calendar, email, search results)
3. Test if the agent acts on injected instructions without user confirmation
4. Check if agent has access to local filesystem, password managers, or other sensitive browser state
5. Test if browser extensions can interact with and manipulate the AI agent panel
6. Test adaptive injection: embed instructions in images, metadata, and conversation context — not just text

---

## Multi-Agent System Attacks

Three research papers reveal systemic privacy vulnerabilities in multi-agent systems:

**AgentLeak (arXiv:2602.11510):** First full-stack benchmark for privacy leakage in multi-agent LLM systems. Key finding: multi-agent configs reduce per-channel output leakage (27.2% vs 43.2% single-agent) but introduce **unmonitored internal channels** — inter-agent messages and shared memory. 7-channel taxonomy, 32-class attack taxonomy across 4,979 execution traces.

**OMNI-LEAK (arXiv:2602.13477):** A single indirect prompt injection in a public database can cascade through orchestrator multi-agent patterns — SQL agent -> orchestrator -> notification agent -> data exfiltration. Even a 1/500 success rate in a 100-person company could leak sensitive data within five days. All tested frontier models except claude-sonnet-4 were vulnerable.

**Agents of Chaos (arXiv:2602.20021, February 2026):** First red-team study of autonomous agents deployed in a live lab with persistent memory, email, Discord, file systems, and shell access. Over 2 weeks, 20 AI researchers interacted under benign and adversarial conditions. **11 representative failure modes** documented:
- **Synonym-based PII bypass** — agent refused to "share" SSNs/bank data but complied when asked to "forward" them
- **Self-destructive actions** — one agent destroyed its own mail server
- **Infinite loops** — two agents stuck in a 9-day recursive loop
- **False completion reporting** — agents reported task success while system state contradicted claims
- **Unauthorized compliance** — agents obeyed instructions from non-owners without verification

**CORBA (arXiv:2502.14529):** Contagious Recursive Blocking Attacks force multi-agent systems into recursive blocking states. 79-100% of AutoGen agents blocked within 1.6-1.9 dialogue turns. Blocking messages appear benign, making detection extremely difficult.

**Inter-Agent Trust Exploitation (ICLR 2026):** 82.4% of tested LLMs can be compromised through inter-agent trust — models that resist direct malicious commands will execute identical payloads when requested by peer agents.

**Testing Approach:**
1. If target uses multi-agent systems, test inter-agent communication channels for injection
2. Test if compromising one agent cascades to others via shared state or orchestrator
3. Test for contagious blocking/DoS attacks across agent networks
4. Test if agents blindly trust instructions from peer agents

**Severity Guidance:** Critical for multi-agent enterprise deployments; High for dual-agent systems. A single compromised agent can poison 87% of downstream decision-making within 4 hours.

---

## Agent-to-Agent (A2A) Protocol Attack Surface

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

---

## Promptware Kill Chain

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

---

## Novel Attack Techniques

> **Full coverage (18 techniques with testing procedures):** See [llm-attack-techniques.md](llm-attack-techniques.md)

Key patterns: LPCI (persistent memory-store payloads surviving across sessions), ForcedLeak (CRM agent form injection, CVSS 9.4), ZombieAgent (zero-click self-propagating memory corruption), SOUL.md identity file poisoning, GRP-Obliteration (single-prompt safety alignment removal across 15 models), Policy Puppetry (universal jailbreak across all frontier models — also bypasses detection guardrails), H-CoT (chain-of-thought hijacking dropping refusal from 99% to 2%), image-based prompt injection (64% ASR under stealth constraints), autonomous jailbreak agents (97.14% multi-turn success).

---

## Full Schema Poisoning (FSP)

An evolution beyond tool description poisoning where attackers compromise entire tool schema definitions at the structural level (Adversa AI, March 2026):

**How FSP Differs from Tool Poisoning:**
- Traditional tool poisoning hides instructions in tool **descriptions** — FSP modifies the **schema structure itself**
- Hidden parameters, altered return types, or malicious default values affect all subsequent tool invocations
- Poisoned schemas appear legitimate to monitoring systems that only scan descriptions
- Description-only scanners (most current MCP security tools) miss FSP entirely

**Attack Vectors:**
1. **Hidden parameters** — Add undocumented parameters to `inputSchema` that the LLM discovers and uses (e.g., a `command` parameter accepting shell input not shown in documentation)
2. **Altered return types** — Modify response schemas to include fields that instruct the LLM to take additional actions
3. **Malicious defaults** — Set default parameter values that execute dangerous operations when the LLM calls the tool without specifying all parameters
4. **Type confusion** — Change parameter types (e.g., string → object) to enable injection through structured data

**Testing for FSP:**
1. Inspect `inputSchema` fields for parameters not visible in tool documentation or UI
2. Compare schema definitions to tool documentation — undocumented parameters are suspicious
3. Check default values for dangerous operations (file paths, shell commands, URLs)
4. Test if schema modifications survive across tool invocations without re-approval
5. Verify if the target's MCP scanner checks schema structure, not just descriptions

**Severity Guidance:** High-Critical when FSP enables code execution or data exfiltration through structural schema manipulation that bypasses description-only security tools. Maps to MCP06 (Tool Poisoning) in OWASP MCP Top 10.

---

## Supply Chain Worm: Shai-Hulud

**454,648 malicious npm packages** in 2025 (99% of all open-source malware). s1ngularity harvested 2,349 credentials from 1,079 developer systems. **Test:** lockfile integrity, postinstall scripts, dependency pinning, npm token rotation.

## Phantom: Template-Based Agent Hijacking

Role confusion jailbreak exploiting chat template tokens (arXiv:2602.16958, Feb 2026). Template Autoencoder (TAE) maps structural patterns into latent space; Bayesian optimization finds adversarial vectors. **79.76% ASR** across 7 closed-source agents (GPT-4.1, Gemini-3), **70+ vendor vulnerabilities**. Operates at **structural/token level** — model-agnostic, bypasses instruction-tuning. **Test:** inject system/assistant role markers within user messages; test token bleeding across turns; check role confusion persistence in multi-turn.

---

## AI-as-Exfiltration-Channel: Unified Pattern Class

**The pattern:** attacker-controlled content → privileged AI agent context → exfiltration sink. Subsumes ForcedLeak, Reprompt, DXT confused deputy, ZombieAgent, SpAIware, and CVE-2026-26144. All are variants of the same trust boundary violation.

**Canonical chain:** low-privilege data source (calendar, email, form, web page, Excel file) → AI agent with privileged access (CRM, filesystem, credentials, Copilot) → exfiltration via markdown image injection, constructed URLs, memory persistence, or agent-mediated network requests.

**Real-world instances:**

| Variant | Entry Point | AI Agent | Exfil Method | CVE/Source |
|---------|------------|----------|--------------|------------|
| ForcedLeak | Web-to-Lead form | Salesforce Agentforce | Markdown image injection via CSP gap | Varonis March 2026 |
| Reprompt | Shared document link | Microsoft Copilot Personal | P2P URL parameter injection + double-request | Varonis, patched Jan 2026 |
| DXT Confused Deputy | Google Calendar invite | Claude DXT (Desktop Commander) | Direct code execution, zero-click | LayerX, CVSS 10.0, Feb 2026 |
| ZombieAgent char exfil | Email | ChatGPT with memory | Character-at-a-time URL construction | Radware, Jan 2026 |
| SpAIware | Any processed content | ChatGPT persistent memory | Memory save → ongoing surveillance | ScienceDirect 2026 |
| Copilot Agent exfil | Excel file (zero-click) | Microsoft Copilot Agent mode | Unintended network egress from Excel | CVE-2026-26144, Critical |
| Agentic Collapse | Public chat (zero-click) | AI RAG agent → osTicket | Semantic coercion bypasses WAF — agent executes vuln tool | CVE-2026-22200, Penligent |
| ShadowLeak | Crafted email (zero-click) | ChatGPT Deep Research + Gmail | Server-side exfil, no UI indicator | Radware, Sept 2025 |

**Reportability:** All five must be present — (1) victim-accessible content placement, (2) privileged tool/data behind agent, (3) exfil sink (URL/email/API), (4) second-user harm, (5) missing trust boundary the AI should have enforced.

**Quick tests:** calendar→RCE | email→file exfil | form→CRM data | Slack→API abuse | Excel→network egress | issue→cred theft | RSS→backdoor | **chat→legacy-tool-RCE** (agentic collapse). Maps to ASI01 + ASI02 + ASI05. **Full DXT details:** [ide-supply-chain.md](ide-supply-chain.md)
