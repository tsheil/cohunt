---
name: ai-hunting
description: AI-assisted bug hunting workflows and AI/LLM-specific vulnerability patterns. Use AI tools to accelerate recon, code review, and vulnerability discovery. Also covers testing AI-powered features for prompt injection, insecure output handling, and other OWASP LLM Top 10 issues. Trigger with "use AI to hunt", "AI hunting workflow", "test LLM features", "prompt injection", "AI vulnerability", "LLM security", "test the chatbot", "AI-assisted", "MCP security", "agent security", "AI agent attack", "tool poisoning", "RAG poisoning", "model extraction", "AI red team", or when target has AI/ML features, chatbots, copilots, or MCP integrations. For traditional web vuln classes (XSS, SSRF, SQLi), use vuln-patterns. For auth bypass on AI-gated features, use auth-testing. For AI IDE supply chain attacks, see reference/ide-supply-chain.md.
---

# AI-Assisted Bug Hunting & LLM Vulnerability Discovery

## Area 1: Using AI to Accelerate Your Hunting

### The Bionic Hacker Approach

82% of hackers now use AI in their workflows (Bugcrowd 2026). The key is **AI amplification, not replacement** — LLMs handle pattern matching and code analysis while you focus on logic, chaining bugs, and manual verification. 72% of hackers find more critical vulnerabilities when collaborating in teams. Pair AI-augmented workflows with team-based hunting for maximum impact.

### AI OPSEC — Protecting Your Hunting Data

**Critical:** Sharing target source code, internal endpoints, or proprietary API schemas with public AI services (ChatGPT, Claude, Gemini) risks violating bug bounty program terms and can get you **banned**. Programs increasingly monitor for unauthorized data disclosure.

| Risk | Mitigation |
|------|------------|
| **Source code leakage** | Use local/private LLMs (Ollama, LM Studio) for code analysis of in-scope targets |
| **Endpoint disclosure** | Strip target-identifying info before pasting into public LLMs |
| **Program ban** | Check program policy on AI tool usage before sharing any target data |
| **Confirmation bias** | AI acts as an echo chamber — if you think you found something, the LLM will agree. Always validate with manual testing before reporting |
| **Training data risk** | Your prompts may become training data. Never paste API keys, tokens, or credentials into public LLMs |

### Precise Prompt Crafting for Hunting

Vague prompts produce vague results. Use surgical prompts targeting specific vulnerability patterns:

| Bad Prompt | Good Prompt |
|-----------|-------------|
| "Find bugs in this code" | "Analyze this Express route handler for SSRF via the `url` parameter — check if the URL is validated before `fetch()` is called" |
| "Is this secure?" | "Check if this JWT verification uses `algorithms: ['HS256']` with the RSA public key, enabling algorithm confusion (CVE-2015-9235)" |
| "Review for vulnerabilities" | "Trace all paths from `req.body` to database queries in this controller — list any that skip parameterized queries" |
| "Check this API" | "Given this OpenAPI spec, generate 5 test cases for mass assignment on the `PATCH /users/{id}` endpoint targeting admin-only fields" |

**Pattern:** Specify the vulnerability class, the input source, the dangerous sink, and the specific technique. Include CVE references when testing for known patterns.

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
| **CAI** | CTF-level challenges, structured offensive tasks | Top-30 Spain, top-500 worldwide HTB; 3,600x faster than humans in CTF benchmarks; 156x cost reduction; non-professionals find CVSS 4.3-7.5 bugs at expert rates |
| **AISLE** | Deep C/C++ memory safety bugs at scale | 100+ CVEs across 30+ projects (OpenSSL, Linux kernel, glibc, Chromium, Firefox, WebKit, Apache HTTPd, GnuTLS, OpenVPN, Samba, NASA CryptoLib); 15 OpenSSL CVEs including CVE-2025-15467 (CVSS 9.8 RCE); found bugs dating back 25-27 years |
| **Codex Security** | Continuous source code monitoring | 792 critical + 10,561 high findings, 1.2M commits scanned |
| **Google Big Sleep** | LLM-guided variant analysis of open-source C/C++ | 20 zero-days in popular OSS (SQLite CVE-2025-6965 CVSS 7.2, FFmpeg, ImageMagick); DeepMind + Project Zero collab; found SQLite memory corruption missed by years of fuzzing; human verification step in loop |
| **Claude Code Security** | Deep codebase analysis, variant finding | 500+ vulns in OSS; 22 Firefox vulns (14 high-severity) in 2 weeks scanning 6,000 C++ files; 112 unique bug reports to Mozilla; first AI-authored browser exploit (CVE-2026-2796, CVSS 9.8); $4K API credits vs $3K-$20K per vuln bounty |
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

**XBOW reached #1 on HackerOne US leaderboard** in 90 days with 1,060 submissions (54 critical, 242 high), cumulative 1,400+ findings. Architecture: coordinator → multiple solver agents with isolated attack machines — runs up to 80x faster than manual teams. Raised $75M but currently operating in the red (compute > bounties).

**Where autonomous agents excel:** Mass SSRF scanning, subdomain enumeration, simple SQLi/XSS, structured exploitation tasks, code analysis at scale.

**Where humans still win:** Context-dependent bugs, multi-step business logic, authentication-gated scenarios, chaining bugs across components, creative attack paths.

**Your role:** Use autonomous tools for coverage; manually verify and chain findings to create reportable exploits. Both Anthropic (Claude Code Security) and OpenAI (Codex Security) launched enterprise-grade autonomous scanning the same week (March 6, 2026) — human hunters must differentiate with chains, business logic, and agent-specific attack patterns that scanning tools cannot replicate. **Wiz Cyber Model Arena** (March 2026) confirmed: AI solved 9/10 directed challenges at under $50 but fails at broad enumeration — human direction + AI execution is the winning model.

> **Full competitive analysis and benchmarks:** See [reference/tools-landscape.md](reference/tools-landscape.md)

---

## Key Attack Surfaces

### MCP (Model Context Protocol)

MCP is AI's fastest-growing attack surface: **50+ CVEs by March 2026** (30 in 60 days alone), **42,665 exposed instances** (5,194 actively vulnerable), 38% of servers lack authentication, 43% have command injection flaws. **Pynt quantified the risk**: 10 MCP plugins = **92% exploit probability** (281 configs analyzed); Enkrypt AI scanned 1,000+ servers and found **33% had critical vulnerabilities**. ChatGPT can now connect to any remote MCP server, expanding the attack surface to all Enterprise/Business users. New attack vectors in early 2026: **Schema Drift** (silent tool schema expansion between versions), **Context Pivoting** (lateral movement via shared agent context), and **Full Schema Poisoning** (structural schema compromise bypassing description-only scanners). Endor Labs analysis confirmed MCP inherits classical CWE-22/CWE-77 at scale: 82% of 2,614 implementations vulnerable to path traversal.

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

> **62+ MCP test procedures, OWASP MCP Top 10, Checkmarx 11 risks, Pynt quantified risk model, and vulnerability stats:** See [reference/mcp-playbooks.md](reference/mcp-playbooks.md)
> **Real-world MCP incidents and case studies:** See [reference/ai-case-studies.md](reference/ai-case-studies.md)

### Agent Skill Supply Chain

**ClawHavoc** (Feb 2026): 1,184+ malicious skills on ClawHub (~1 in 5 packages). **ToxicSkills** (Snyk): 3,984 skills audited — 36% contain security flaws, 76 confirmed malicious payloads, **100% of malicious skills combine code-level attacks with prompt injection** (dual-vector); mcp-scan detection engine achieves 90-100% recall with 0% false positives. SKILL.md to shell access in 3 lines of markdown. **Clinejection**: prompt injection in GitHub issue → AI triage bot → npm token theft → malicious package on 4,000 machines.

**Claude Code CVE-2026-21852** (CVSS 5.3): information disclosure in project-load flow — malicious repository exfiltrates data including Anthropic API keys; fixed in v2.0.65 (Jan 2026). Pattern: AI coding tool trust boundary violations via crafted repos.

**Scan before installing any third-party skill** with Cisco MCP Scanner, Snyk Agent Scan, mcp-scan, or Repello SkillCheck.

### IDEsaster: AI Coding IDE Attack Surface

30+ vulnerabilities across 10+ AI coding tools, 24 CVEs. Key patterns: workspace trust bypass (Cursor), rules file backdoor (invisible Unicode), RoguePilot (GitHub issue → token theft), PromptPwnd (CI/CD pipeline injection), extension namespace squatting.

### Agentic Browser Attack Surface

Zero-click agent hijacking via calendar invites/emails/documents. File system exfiltration, credential manager access, extension escalation. Affects Perplexity Comet, Chrome Gemini panel, ChatGPT Atlas.

### AI Inference Server Supply Chain

**vLLM CVE-2026-22807**: AI inference engines loading Hugging Face `auto_map` dynamic modules without `trust_remote_code` gating — attacker-controlled Python in model repos executes at server startup before any request handling. Pattern applies to any inference server processing model configs as code. **Code generators** (Orval CVE-2026-23947) also vulnerable: untrusted OpenAPI specs inject code via unescaped `x-enumDescriptions` → generated TypeScript executes on import.

### Multi-Agent System Attacks

**OMNI-LEAK**: single injection cascades through orchestrator → 82.4% of LLMs compromised via inter-agent trust. **CORBA**: 79-100% of agents blocked within 1.6 turns. **ZombieAgent**: zero-click self-propagating memory corruption via email.

> **OWASP Agentic Top 10 (19 test procedures), agent supply chain, agentic browsers, novel techniques:** See [reference/agent-attack-patterns.md](reference/agent-attack-patterns.md)
> **IDEsaster (28+ CVEs), Claude DXT, AI-as-C2, Google Antigravity:** See [reference/ide-supply-chain.md](reference/ide-supply-chain.md)
> **50+ real-world incidents and case studies:** See [reference/ai-case-studies.md](reference/ai-case-studies.md)

---

## Quick Start — Testing an AI Feature (Without Getting N/A'd)

1. **Classify surface** — Chat / RAG / tool-using agent / MCP / IDE plugin / code generator?
2. **Identify attacker-controlled input** — What data does the AI process that you can influence?
3. **Identify higher-privileged data or tool** — What can the AI access that you cannot? (other users' data, internal APIs, file system, email)
4. **Plant payload** — Inject via the attacker-controlled input (document, issue, email, shared content).
5. **Trigger from second session/victim context** — Does the AI execute your payload when a *different* user or session processes it?
6. **Capture unauthorized read/action** — Did the AI exfiltrate data, execute an action, or access something it shouldn't?
7. **Verify cross-boundary harm** — Is someone other than yourself harmed? (If no: not a bounty — move on.)
8. **Test persistence** — Only if the first hit lands: does it persist across sessions or affect future users?
9. **Stop if** — It's only a self-session jailbreak, prompt leak without secrets, or hallucination.
10. **Run `/reportability-check`** before `/write-report`. If blocked or patched, run `/variant-hunt`.

**Quick severity filter — what pays vs. what doesn't:**

| Pays Bounty | Does NOT Pay |
|-------------|-------------|
| Prompt injection → accessing *other users'* data | "I made the AI say something bad" (pure jailbreak) |
| Tool executes unauthorized action (email, delete, modify) | System prompt leak with no sensitive data |
| Indirect injection exfiltrates secrets from victim | Prompt injection with no downstream effect |
| MCP tool poisoning → RCE or credential theft | Self-XSS via AI output in own chat |
| Memory poisoning affects future sessions/other users | AI hallucinated wrong information |

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
- **OWASP Top 10 for Agentic Applications 2026** (ASI01-ASI10): Globally peer-reviewed framework developed with 100+ experts; covers goal hijacking, tool misuse, privilege escalation, memory poisoning, cascading failures — **19 test procedures** in [reference/agent-attack-patterns.md](reference/agent-attack-patterns.md)
- **OWASP AIVSS**: AI-specific vulnerability scoring extending CVSS for agentic capabilities

> **Full testing patterns for each LLM Top 10 category + practical workflow checklists:** See [reference/llm-testing.md](reference/llm-testing.md)

---

## Critical Warning: "AI Slop" Reports

**curl ended its bug bounty** (Jan 2026) after AI submissions overwhelmed the team — first program shutdown attributed to AI slop. Over 6 years, the program paid $90K+ for 81 genuine vulnerabilities, but by 2025 ~20% of submissions were AI slop and only 5% of all submissions were genuine. Not a single AI-only-generated submission discovered a real vulnerability. Submission volume spiked 8x while quality cratered — a cautionary signal for all programs.

**Platform response:** HackerOne launched **Hai Insight Agent** (March 2026) to filter hallucinated hackbot reports before triage. Programs are now using AI validation to detect and auto-close AI slop — submitting unverified AI output wastes your reputation.

**Before submitting:**
1. Reproduce the finding yourself (Burp, curl, manual testing)
2. Include specific payloads and reproduction steps
3. Screenshot or video proof
4. Call out AI-assisted analysis transparently
5. Verify AI output against reality — don't trust confidence levels alone
6. Guard against **confirmation bias** — if you asked an LLM "is this a vulnerability?" and it said yes, that means nothing. Test it manually

---

## Program Scoping & Bounty Notes

### Hunter Implications (Key Stats)

- **Hunter implication — AI is the #1 growth surface:** 1,121 programs include AI in scope (270% YoY increase); 50+ MCP CVEs in 60 days; 38% of servers lack auth. Hunt MCP and agentic features.
- **Hunter implication — Autonomous tools are competitors:** XBOW is #1 on HackerOne with 1,400+ findings and 80x speed. Focus on business logic (45% of awards) and multi-step chains that agents can't replicate.
- **Hunter implication — Bounties are rising:** Apple $2M max, OpenAI $100K, Samsung $1M. Bug bounty market $2.06B (2026). AI-specific programs paying Critical for cross-boundary prompt injection.
- **Hunter implication — Regulatory deadlines create urgency:** EU AI Act (Aug 2026), EU CRA (Sep 2026) — programs must fix AI vulns or face 7% global turnover penalties.

> **Full market context database (150+ metrics):** See [reference/market-context.md](reference/market-context.md)

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
| [reference/tools-landscape.md](reference/tools-landscape.md) | Full AI security tools catalog (40+ tools), Promptfoo (OpenAI acquisition), Sage ADR, security MCP servers, red teaming frameworks, Pynt/Noma research | ~496 |
| [reference/mcp-playbooks.md](reference/mcp-playbooks.md) | 70 MCP test procedures, OWASP MCP Top 10, Checkmarx 11 risks, Pynt quantified risk model, Enkrypt scanner findings, OAuth attacks, SDK flaws, Schema Drift, Context Pivoting, Sampling attacks | ~491 |
| [reference/agent-attack-patterns.md](reference/agent-attack-patterns.md) | OWASP Agentic Top 10 + 19 test procedures, agent supply chain, agentic browsers, multi-agent attacks, Full Schema Poisoning, Phantom TAE jailbreak, novel techniques (LPCI, salami slicing, H-CoT, ZombieAgent, GRP-Obliteration) | ~498 |
| [reference/ide-supply-chain.md](reference/ide-supply-chain.md) | IDEsaster CVE table (28+ CVEs), Claude DXT zero-click RCE (Feb 2026), AI-as-C2 proxy, Google Antigravity IDE, CI/CD pipeline injection | ~295 |
| [reference/ai-case-studies.md](reference/ai-case-studies.md) | 60+ real-world incidents, Phantom/Sage ADR/SACR UADP, MCP ecosystem risk quantification, Microsoft AI as Tradecraft, platform AI policy updates, red teaming tools, AI bug bounty platforms, NIST standards | ~471 |
| [reference/llm-testing.md](reference/llm-testing.md) | OWASP LLM Top 10 testing patterns, system prompt extraction, indirect injection workflows, data exfiltration tests | ~364 |
| [reference/llm-attack-techniques.md](reference/llm-attack-techniques.md) | Advanced jailbreak techniques: JBF, TAO-Attack, autonomous jailbreak agents (97.14%), Unit42 IDPI catalog, encoding/multilingual/visual bypasses, semantic chaining, hybrid XSS+PI | ~189 |
| [reference/market-context.md](reference/market-context.md) | Full market statistics, program developments, industry metrics, competitive intelligence (150+ data points), Phantom/Sage ADR/SACR UADP stats | ~384 |

**Quick search** — find specific content without loading full files:
```
grep -n "OWASP\|Agentic Top 10\|MCP Top 10\|ASI0" ${CLAUDE_SKILL_DIR}/reference/agent-attack-patterns.md
grep -n "tool poisoning\|name collision\|sampling\|OAuth\|schema" ${CLAUDE_SKILL_DIR}/reference/mcp-playbooks.md
grep -n "XBOW\|Shannon\|Codex Security\|ZeroPath\|Ghost Security" ${CLAUDE_SKILL_DIR}/reference/tools-landscape.md
grep -n "IDEsaster\|DXT\|Cursor\|Copilot\|VS Code" ${CLAUDE_SKILL_DIR}/reference/ide-supply-chain.md
grep -n "prompt injection\|jailbreak\|system prompt\|indirect injection" ${CLAUDE_SKILL_DIR}/reference/llm-testing.md
grep -n "CVE-\|CVSS\|incident\|breach\|exploit" ${CLAUDE_SKILL_DIR}/reference/ai-case-studies.md
```

---

## Related Skills

- **vuln-patterns**: Common vulnerability patterns and how to spot them
- **source-code-audit**: In-depth code review workflows and tools
- **report-writing**: Crafting high-quality, reportable vulnerability submissions
- **target-recon**: Reconnaissance techniques and tooling for mapping attack surface
