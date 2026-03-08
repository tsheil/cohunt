# IDE & Supply Chain Attack Patterns

Attack patterns targeting AI coding IDEs, desktop extensions, and software supply chain vectors. Extracted from agent-attack-patterns.md for progressive disclosure.

> **Related files:** [agent-attack-patterns.md](agent-attack-patterns.md) for OWASP Agentic Top 10, multi-agent attacks, novel techniques | [mcp-playbooks.md](mcp-playbooks.md) for MCP test procedures

---

## Table of Contents

- [IDEsaster: AI Coding IDE Attack Surface](#idesaster-ai-coding-ide-attack-surface)
- [Claude DXT Zero-Click RCE](#claude-dxt-zero-click-rce)
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

**RoguePilot (Orca Security, 2026):**
- Passive prompt injection via GitHub issues enabling **GITHUB_TOKEN theft** through Codespaces/Copilot
- Attack uses hidden HTML comments in GitHub issues to inject prompts when a Codespace is opened
- Exploits VS Code's `json.schemaDownload.enable` setting to exfiltrate tokens to external servers
- Patched by Microsoft — test for similar patterns in other IDE integrations

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
- CVE-2026-29783: shell tool allows arbitrary code execution through bash parameter expansion patterns

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

**Testing Approach:**
1. If target uses Claude Desktop Extensions, check if DXT execution is sandboxed
2. Test if content from connected services (calendar, email, docs) can trigger DXT actions
3. Check for privilege boundaries between DXT execution and host system
4. Test if DXT tool calls are gated by user confirmation

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
