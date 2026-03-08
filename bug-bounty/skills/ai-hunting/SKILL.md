---
name: ai-hunting
description: AI-assisted bug hunting workflows and AI/LLM-specific vulnerability patterns. Use AI tools to accelerate recon, code review, and vulnerability discovery. Also covers testing AI-powered features for prompt injection, insecure output handling, and other OWASP LLM Top 10 issues. Trigger with "use AI to hunt", "AI hunting workflow", "test LLM features", "prompt injection", "AI vulnerability", "LLM security", "test the chatbot", "AI-assisted", or when target has AI/ML features.
---

# AI-Assisted Bug Hunting & LLM Vulnerability Discovery

## Area 1: Using AI to Accelerate Your Hunting

### The Bionic Hacker Approach

82% of hackers now use AI in their workflows (Bugcrowd 2026). The key is **AI amplification, not replacement** — LLMs handle pattern matching and code analysis while you focus on logic, chaining bugs, and manual verification. 72% of hackers find more critical vulnerabilities when collaborating in teams. Pair AI-augmented workflows with team-based hunting for maximum impact.

### AI-Assisted Recon Workflows

| Phase | AI Task | Output |
|-------|---------|--------|
| **Subdomain Generation** | Generate wordlist combinations from target domain patterns | 100+ candidates for Subfinder/Assetfinder |
| **JS Bundle Analysis** | Decompile/identify API endpoints, auth tokens, hardcoded creds | Annotated findings with exploit chain suggestions |
| **Recon Synthesis** | Aggregate Shodan, GitHub leaks, public bug reports | Prioritized attack paths |
| **Endpoint Mapping** | Parse Swagger/OpenAPI docs, generate edge cases | Test cases and parameter combinations |

### AI-Assisted Code Review

**Top tools by use case:**

| Tool | Best For | Key Metric |
|------|----------|------------|
| **Vulnhalla** | CodeQL + LLM-guided questioning | 96% false positive reduction |
| **Hound** | Deep logic bugs, multi-file chains via knowledge graphs | Language-agnostic |
| **Shannon** | Autonomous white-box web app pentesting | 96.15% on XBOW benchmark |
| **CAI** | CTF-level challenges, structured offensive tasks | Top-20 worldwide CTF, 11x faster |
| **AISLE** | Deep C/C++ memory safety bugs at scale | 100+ CVEs, 12/12 OpenSSL zero-days |
| **Codex Security** | Continuous source code monitoring | 14 CVEs found, 1.2M commits scanned |
| **Claude Code Security** | Deep codebase analysis, variant finding | 500+ vulns, 22 Firefox vulns in 2 weeks |
| **ZAST.AI** | Zero-false-positive source code scanning | 119 CVEs assigned |
| **Strix** | Autonomous web app testing with browser + proxy | Used by top 1% HackerOne hunters |

> **Full tool catalog (40+ tools):** See [reference/tools-landscape.md](reference/tools-landscape.md)

### AI-Assisted Fuzzing & Payload Generation

| Use Case | Expected Gain |
|----------|---------------|
| **Wordlist Expansion** — feed known payloads, ask for WAF bypass variations | 3-5x more test cases |
| **Protocol Fuzzing** — describe protocol, generate malformed packets | Faster parser bug discovery |
| **Context-Aware Payloads** — show tech stack + recent CVEs | Targeted vs. spray-and-pray |
| **AI PoC Generation** — PoCGen framework | 77% success rate (432/560 vulns) |

**Warning — "PoC Pollution":** AI-generated PoCs are flooding repositories with broken exploits. Always validate any AI-generated PoC before using it.

### The XBOW/Autonomous Agent Reality Check

**XBOW reached #1 on HackerOne US leaderboard** in 90 days with 1,060 submissions (54 critical, 242 high). Architecture: coordinator → multiple solver agents with isolated attack machines. Raised $75M but currently operating in the red (compute > bounties).

**Where autonomous agents excel:** Mass SSRF scanning, subdomain enumeration, simple SQLi/XSS, structured exploitation tasks, code analysis at scale.

**Where humans still win:** Context-dependent bugs, multi-step business logic, authentication-gated scenarios, chaining bugs across components, creative attack paths.

**Your role:** Use autonomous tools for coverage; manually verify and chain findings to create reportable exploits. Both Anthropic (Claude Code Security) and OpenAI (Codex Security) launched enterprise-grade autonomous scanning the same week (March 6, 2026) — human hunters must differentiate with chains, business logic, and agent-specific attack patterns that scanning tools cannot replicate.

> **Full competitive analysis and benchmarks:** See [reference/tools-landscape.md](reference/tools-landscape.md)

---

## Key Attack Surfaces

### MCP (Model Context Protocol)

MCP is AI's fastest-growing attack surface: **30+ CVEs in 60 days**, 38% of servers lack authentication, 43% have command injection flaws. New attack vectors in early 2026: **Schema Drift** (silent tool schema expansion between versions), **Context Pivoting** (lateral movement via shared agent context), and **Full Schema Poisoning** (structural schema compromise bypassing description-only scanners).

**Top vulnerability classes:**

| Class | Description | Severity |
|-------|-------------|----------|
| Tool Poisoning | Hidden instructions in tool descriptions manipulate AI behavior | High-Critical |
| Indirect Prompt Injection via MCP | Instructions planted in retrieved content (issues, docs, emails) | Critical |
| Over-Privileged Tokens | Broad PATs/API keys enable cross-system chaining | Critical |
| eval() Epidemic | User input to eval()/exec() — 7 RCE CVEs in Feb 2026 alone | Critical |
| Supply Chain (SANDWORM_MODE) | npm packages inject rogue MCP servers into AI coding configs | Critical |
| Tool Shadowing | Similar-named tools intercept calls to legitimate ones | High |
| SDK Cross-Client Leak | Reused server instances leak responses across clients | High-Critical |
| MCP OAuth Account Takeover | CSRF-style attacks via server acting as both authz and client | Critical |

**OWASP MCP Top 10 (2026):** MCP01 (Token Mismanagement) through MCP10 (Insufficient Logging) — use risk IDs when framing MCP findings.

**Where to hunt:** Any product integrating MCP servers (Claude Desktop, Cursor, Windsurf, VS Code), enterprise AI agent platforms, open-source `mcp-server-*` repos, huntr platform.

> **59 MCP test procedures, OWASP MCP Top 10, and vulnerability stats:** See [reference/mcp-playbooks.md](reference/mcp-playbooks.md)
> **Real-world MCP incidents and case studies:** See [reference/ai-case-studies.md](reference/ai-case-studies.md)

### Agent Skill Supply Chain

**ClawHavoc** (Feb 2026): 1,184+ malicious skills on ClawHub (~1 in 5 packages). **ToxicSkills** (Snyk): 36% of 3,984 audited skills contain prompt injection; SKILL.md to shell access in 3 lines of markdown. **Clinejection**: prompt injection in GitHub issue → AI triage bot → npm token theft → malicious package on 4,000 machines.

**Scan before installing any third-party skill** with Cisco MCP Scanner, Snyk Agent Scan, or Repello SkillCheck.

### IDEsaster: AI Coding IDE Attack Surface

30+ vulnerabilities across 10+ AI coding tools, 24 CVEs. Key patterns: workspace trust bypass (Cursor), rules file backdoor (invisible Unicode), RoguePilot (GitHub issue → token theft), PromptPwnd (CI/CD pipeline injection), extension namespace squatting.

### Agentic Browser Attack Surface

Zero-click agent hijacking via calendar invites/emails/documents. File system exfiltration, credential manager access, extension escalation. Affects Perplexity Comet, Chrome Gemini panel, ChatGPT Atlas.

### Multi-Agent System Attacks

**OMNI-LEAK**: single injection cascades through orchestrator → 82.4% of LLMs compromised via inter-agent trust. **CORBA**: 79-100% of agents blocked within 1.6 turns. **ZombieAgent**: zero-click self-propagating memory corruption via email.

> **Agent supply chain, agentic browsers, novel techniques, OWASP Agentic Top 10:** See [reference/agent-attack-patterns.md](reference/agent-attack-patterns.md)
> **IDEsaster (28+ CVEs), Claude DXT, AI-as-C2, Google Antigravity:** See [reference/ide-supply-chain.md](reference/ide-supply-chain.md)
> **50+ real-world incidents and case studies:** See [reference/ai-case-studies.md](reference/ai-case-studies.md)

---

## Area 2: Hunting Bugs in AI/LLM Features

When a target has AI/LLM features (chatbots, AI assistants, code generators, content modifiers), test against the **OWASP LLM Top 10 2025**:

| Risk | Name | Quick Test |
|------|------|------------|
| LLM01 | Prompt Injection | Direct: override system prompt. Indirect: inject via retrieved content |
| LLM02 | Sensitive Info Disclosure | Ask about training data, system internals, PII in responses |
| LLM03 | Supply Chain | Check dependencies, model provenance, plugin integrity |
| LLM04 | Data/Model Poisoning | Test if user-contributed content influences model outputs |
| LLM05 | Improper Output Handling | Check if LLM output rendered unsanitized (XSS, SQLi, command injection) |
| LLM06 | Excessive Agency | Test if LLM can execute actions beyond intended scope |
| LLM07 | System Prompt Leakage | Extract system prompt via encoding tricks, translation, roleplay |
| LLM08 | Vector/Embedding Weaknesses | Test RAG poisoning, embedding collision attacks |
| LLM09 | Misinformation | Test if LLM provides harmful advice in critical domains |
| LLM10 | Unbounded Consumption | Test for denial-of-wallet via large prompts or recursive tool calls |

**Also test against:**
- **OWASP Agentic Top 10** (ASI01-ASI10): Goal hijacking, tool misuse, privilege escalation, memory poisoning, cascading failures
- **OWASP AIVSS**: AI-specific vulnerability scoring extending CVSS for agentic capabilities

> **Full testing patterns for each LLM Top 10 category + practical workflow checklists:** See [reference/llm-testing.md](reference/llm-testing.md)

---

## Critical Warning: "AI Slop" Reports

**curl ended its bug bounty** (Jan 2026) after AI submissions overwhelmed the team — first program shutdown attributed to AI slop. In six years, not a single AI-only-generated submission discovered a genuine vulnerability.

**Before submitting:**
1. Reproduce the finding yourself (Burp, curl, manual testing)
2. Include specific payloads and reproduction steps
3. Screenshot or video proof
4. Call out AI-assisted analysis transparently
5. Verify AI output against reality — don't trust confidence levels alone

---

## Program Scoping & Bounty Notes

### Key Market Stats (2025-2026)

- **Business logic vulnerabilities** = **45% of all bounty awards** industrywide (Intigriti 2026) — highest-value human edge
- **1,121 programs** on HackerOne include AI in scope (270% YoY increase)
- **Bug bounty market**: $2.06B (2026), projected $7.74B by 2035 (CAGR 15.94%)
- **XBOW**: #1 HackerOne with 1,060 submissions; **AISLE**: 100+ CVEs including all 12 OpenSSL zero-days
- **40+ MCP CVEs** in Q1 2026; 38% of servers lack auth; 43% have command injection
- **3+ million AI agents** in corporations; 88% reported security incidents; 47% not monitored
- **Only 10% of AI-generated code** is secure (Endor Labs)
- **Enterprise AI gap**: 83% plan agentic AI, only 29% ready to secure it
- **EU AI Act deadline**: August 2, 2026 (penalties up to 35M EUR or 7% of global turnover)
- **Apple max bounty $5M**; Google $250K Chrome record; OpenAI raised to $100K; Samsung $1M mobile
- **AI agent attacks**: 66-84% success when testing prompt injection against auto-execution systems

> **Full market context database (120+ metrics):** See [reference/market-context.md](reference/market-context.md)

### How to Scope AI Vulnerabilities with Programs

**Before reporting, check:**
1. **Is AI/ML in scope?** Look for: "AI", "LLM", "chatbot", "generative", "machine learning"
2. **Are OWASP LLM Top 10 items explicitly in scope?**
3. **What's the risk tolerance?** Some programs consider prompt injection low priority if output isn't executed

**Red Flags (Don't Report):**
- "AI features out of scope" or "we're aware of known LLM limitations"
- "Prompt injection is by design"
- No explicit AI mention — contact the program first

**Best Approach:** If unsure, ask: "Are LLM vulnerabilities (prompt injection, output injection, training data poisoning) in scope for your AI features?"

### Bounty Tier Expectations

| Severity | OWASP LLM Issue | Typical Range | Notes |
|----------|---|---|---|
| **Critical** | LLM06 + financial impact; LLM02 (PII); LLM01 → Code Execution | $5K-$25K+ | Real exploitation, not "LLM said something weird" |
| **High** | LLM01 + data disclosure; LLM05 → XSS; LLM07 with internal APIs | $2K-$10K | Verified exploitation with clear impact |
| **Medium** | LLM08; LLM04 non-critical; LLM09 critical advice | $500-$2K | Valid but requires multiple steps or lower impact |
| **Low** | Prompt injection with no impact; System prompt non-sensitive info | $100-$500 | Informational |

### What Gets Paid vs. What Gets Rejected

The #1 mistake in AI bug reports: demonstrating a capability without proving security impact. Programs pay for **cross-trust-boundary impact**, not "the AI did something unexpected."

**Pays bounties (crosses a trust boundary):**

| Finding | Why It Pays | Typical Severity |
|---------|-------------|------------------|
| Prompt injection → accessing *other users'* data | Cross-user data access | High-Critical |
| Prompt injection → tool executes unauthorized action (send email, delete data, modify records) | Unauthorized side effects beyond attacker's session | High-Critical |
| Indirect injection via retrieved content → exfiltrates secrets | Attacker-controlled data causes victim's agent to leak PII/tokens | Critical |
| MCP tool poisoning → RCE or credential theft | Trust boundary violation: tool description → code execution | Critical |
| Memory poisoning → affects future sessions or other users | Persistent compromise beyond single interaction | High-Critical |
| Denial-of-wallet → measurable financial impact | Resource exhaustion with quantifiable cost | Medium-High |
| System prompt leak containing API keys, internal URLs | Sensitive data exposure enabling further exploitation | High |

**Does NOT pay (stays within user's own session):**

| Finding | Why It's Rejected |
|---------|-------------------|
| "I made the AI say something bad" (pure jailbreak) | No security impact — content moderation issue, not vulnerability |
| System prompt leak (no sensitive data) | Low/informational — most programs consider system prompts non-secret |
| Prompt injection with no downstream effect | Capability without impact — "so what?" test fails |
| Self-XSS via AI output in own chat | No victim — attacker can only attack themselves |
| AI hallucinated wrong information | Inherent LLM limitation, not a vulnerability |
| "AI follows my instructions" without privilege escalation | Expected behavior — AI is designed to follow instructions |

**The "So What?" Test:** Before submitting, ask: *"If an attacker does this, who is harmed besides themselves?"* If the answer is "no one," it's not a bounty-worthy finding. Chain it with a cross-boundary impact or move on.

### Common Rejection Reasons

- "AI sometimes hallucinates" — inherent limitation, not a bug
- "LLM was tricked by prompt" — only reportable if it leads to real exploitation
- "AI slop report" — vague, no PoC, no reproduction steps
- "Prompt injection stays in chat" — only reportable if it escapes chat or affects other users

---

## Reference Files

This skill uses progressive disclosure. Detailed reference material is available on demand:

| File | Contents | Lines |
|------|----------|-------|
| [reference/tools-landscape.md](reference/tools-landscape.md) | Full AI security tools catalog (40+ tools), security MCP servers, red teaming frameworks | ~377 |
| [reference/mcp-playbooks.md](reference/mcp-playbooks.md) | MCP test procedures (59), OWASP MCP Top 10, vulnerability classes, OAuth attacks, SDK flaws, Schema Drift, Context Pivoting, scanning tools | ~420 |
| [reference/agent-attack-patterns.md](reference/agent-attack-patterns.md) | OWASP Agentic Top 10, agent supply chain, agentic browsers, multi-agent attacks, Full Schema Poisoning, novel techniques (LPCI, salami slicing, H-CoT, ZombieAgent, GRP-Obliteration) | ~480 |
| [reference/ide-supply-chain.md](reference/ide-supply-chain.md) | IDEsaster CVE table (28+ CVEs), Claude DXT zero-click RCE, AI-as-C2 proxy, Google Antigravity IDE, CI/CD pipeline injection | ~163 |
| [reference/ai-case-studies.md](reference/ai-case-studies.md) | 50+ real-world incidents, platform AI policy updates, red teaming tools, AI bug bounty platforms, CTFs, NIST standards, AI slop warning | ~241 |
| [reference/llm-testing.md](reference/llm-testing.md) | OWASP LLM Top 10 testing patterns, system prompt extraction, indirect injection workflows, data exfiltration tests | ~406 |
| [reference/market-context.md](reference/market-context.md) | Full market statistics, program developments, industry metrics, competitive intelligence (120+ data points) | ~161 |

---

## Related Skills

- **vuln-patterns**: Common vulnerability patterns and how to spot them
- **source-code-audit**: In-depth code review workflows and tools
- **report-writing**: Crafting high-quality, reportable vulnerability submissions
- **target-recon**: Reconnaissance techniques and tooling for mapping attack surface
