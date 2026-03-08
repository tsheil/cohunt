# MCP Security Playbooks — Test Procedures & Vulnerability Patterns

Actionable test procedures for hunting vulnerabilities in Model Context Protocol (MCP) implementations. Covers vulnerability classes, OWASP MCP Top 10 mapping, 43+ concrete test patterns, OAuth attack vectors, SDK-level flaws, and MCP-specific tooling.

> **Related files:** [agent-attack-patterns.md](agent-attack-patterns.md) for agent supply chain and novel attack techniques | [ai-case-studies.md](ai-case-studies.md) for real-world MCP incident case studies

---

## Table of Contents

- [MCP Vulnerability Classes](#mcp-vulnerability-classes)
- [OWASP MCP Top 10 (2026)](#owasp-mcp-top-10-2026)
- [55+ MCP Test Procedures](#43-mcp-test-procedures)
- [MCP OAuth Account Takeover](#mcp-oauth-account-takeover)
- [MCP Real-World Attack Examples](#mcp-real-world-attack-examples)
- [MCP Implementation Vulnerability Stats](#mcp-implementation-vulnerability-stats)
- [Denial-of-Wallet via MCP Overthinking Loops](#denial-of-wallet-via-mcp-overthinking-loops)
- [Cursor Rogue MCP Browser Takeover](#cursor-rogue-mcp-browser-takeover)
- [Security MCP Servers for Bug Bounty Workflows](#security-mcp-servers-for-bug-bounty-workflows)
- [MCP Security Scanning Tools](#mcp-security-scanning-tools)
- [Schema Drift: Silent MCP Attack Surface Expansion](#schema-drift-silent-mcp-attack-surface-expansion)
- [Context Pivoting: Lateral Movement via Shared Agent Context](#context-pivoting-lateral-movement-via-shared-agent-context)
- [Where to Hunt MCP Vulnerabilities](#where-to-hunt-mcp-vulnerabilities)

---

## MCP Vulnerability Classes

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
| **MCP Tool Name Collision** | CVE-2026-30856 (CVSS 5.9): by exploiting ambiguous `mcp_{service}_{tool}` naming convention in MCP clients, attacker registers a tool that overwrites a legitimate one (e.g., `tavily_extract`), redirecting LLM execution flow for system prompt exfiltration, context theft, and tool execution with user privileges. CWE-706 (Use of Incorrectly-Resolved Name or Reference). Fixed in WeKnora v0.3.0 | Medium-High |
| **Exec-Approvals Shell Expansion Gap** | CVE-2026-28463: allowlist checks validate pre-expansion argv tokens, but execution uses real shell expansion — safe bins like `head`, `tail`, `grep` can read arbitrary files via glob patterns. Pattern: any AI tool validating commands before executing them has a gap between validation input and execution input | High |
| **ClawJacked (WebSocket Hijacking)** | Malicious websites hijack local AI agent instances (e.g., OpenClaw) via WebSocket connections. Attacker page connects to localhost-bound WebSocket, gains remote control of local agent with all its permissions and tool access | Critical |
| **"Rug Pull" Attacks** | MCP servers modify tool definitions between sessions, presenting different capabilities than initially approved. Exploits dynamic nature of MCP tool definitions for post-deployment modification | High |
| **Schema Drift** | Silent expansion of MCP server attack surface between version updates — new parameters, tools, or capabilities added in patch releases without changelog entries; previous audits become outdated (ecap0/AgentAudit, March 2026) | High |
| **Full Schema Poisoning (FSP)** | Structural compromise of tool schema definitions — hidden parameters, altered return types, malicious default values that affect all tool invocations while appearing legitimate to description-only scanners (Adversa AI, March 2026) | High-Critical |
| **Context Pivoting** | MCP lateral movement via shared agent context — a malicious server manipulates the agent into calling other connected servers' tools with attacker-controlled parameters; MCP equivalent of network pivoting (ecap0/AgentAudit, February 2026) | Critical |
| **Command Injection in MCP Packages** | CVE-2025-6514: critical OS command injection in `mcp-remote` (437K+ downloads) — malicious MCP servers send booby-trapped authorization_endpoint for RCE. Effectively a supply-chain backdoor | Critical |
| **Sandbox Escape** | Anthropic's own Filesystem-MCP server had sandbox escape + symlink bypass enabling arbitrary file access and code execution | Critical |
| **Token Passthrough Anti-Pattern** | MCP server accepts tokens from client without validating they were properly issued to the server — fundamental auth architecture flaw | High |
| **Claude Code Project File Exploitation** | CVE-2025-59536 / CVE-2026-21852: RCE and API token exfiltration through Claude Code project files (Check Point Research) | Critical |
| **Git MCP Server RCE** | CVE-2025-68145, CVE-2025-68143, CVE-2025-68144: Three CVEs in Anthropic's Git MCP server enabling RCE via prompt injection | Critical |
| **MCP Server SSRF -> Cloud Compromise** | Unpatched SSRF in Microsoft MarkItDown MCP server — compromises AWS EC2 instances via metadata service exploitation | Critical |
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
| **OpenClaw Browser Control Auth Bypass** | CVE-2026-28485: versions 2026.1.5 to 2026.2.12 fail to enforce mandatory authentication on `/agent/act` browser-control HTTP route — unauthorized local callers invoke privileged operations | Critical |
| **OpenClaw Browser Control Path Traversal** | CVE-2026-28462: browser control API accepts user-supplied output paths without constraining writes to temporary directories — file write to arbitrary locations | High |
| **OpenClaw Exec Approval Bypass** | CVE-2026-28466: gateway fails to sanitize internal approval fields in `node.invoke` parameters — authenticated clients bypass exec approval gating for `system.run` commands | High |
| **OpenClaw Webhook Handler DoS** | CVE-2026-28478: webhook handlers buffer request bodies without strict byte or time limits — remote unauthenticated attackers send oversized JSON payloads for denial of service | Medium |
| **OpenClaw PATH Command Hijacking** | CVE-2026-29610: command hijacking via PATH environment variable manipulation — execute unintended binaries by prepending attacker-controlled directory to PATH | High |
| **OpenClaw Symlink Sandbox Escape** | GHSA-M8V2-6WWH-R4GC: logic error in path validation — bind mounts for non-existent files within symlinked parent directories trick validation into accepting restricted host paths | Critical |
| **MCP SDK Go Case-Insensitive JSON** | CVE-2026-27896: Go's `encoding/json.Unmarshal` matches field names case-insensitively — crafted responses with "Method" vs "method" bypass validation. Fixed in Go MCP SDK v1.3.1 | High |
| **MCP Stdio Blacklist Bypass** | CVE-2026-30861 (CVSS 9.9): LLM-powered frameworks whitelisting `npx`/`uvx` but failing to block flag injection — `-p` flag with `npx node` bypasses command blacklist entirely. Only requires registering an account (unrestricted registration). Pattern: any MCP stdio config with command allowlists | Critical |
| **SQL Expression Tree Bypass** | CVE-2026-30860 (CVSS 9.9): SQL injection protection that fails to recursively inspect child nodes within PostgreSQL array/row expressions — dangerous functions smuggled inside `ARRAY[]`/`ROW()` constructs chain with large object operations for full RCE. Pattern: any LLM-powered query builder with non-recursive sanitization | Critical |
| **JavaScript `with` Sandbox Escape** | CVE-2026-1470 (CVSS 9.9): deprecated `with` statement in sandboxed JavaScript expression engines enables scope chain manipulation to break sandbox boundaries and execute arbitrary code. Pattern: workflow automation and expression engines using `with`-based scoping | Critical |

---

## OWASP MCP Top 10 (2026)

A dedicated security framework specifically for Model Context Protocol risks, published by OWASP in early 2026. Complements the Agentic Top 10 (see [agent-attack-patterns.md](agent-attack-patterns.md)) by focusing on the protocol layer:

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

---

## 55+ MCP Test Procedures

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
24. Test for AI framework regex denylist bypass: if target uses AI agent frameworks with command execution (Shell tools, code interpreters), can command obfuscation bypass regex-based denylists? (CVE-2026-2256, ModelScope MS-Agent, CVSS 9.8 — prompt injection -> full system compromise)
25. Test for unauthenticated AI agent local servers: does the AI coding tool auto-start an HTTP server on localhost? Is CORS permissive? Can any website trigger command execution? (CVE-2026-22812, OpenCode — any local process or website could execute arbitrary shell commands)
26. Test for workspace trust bypass: does the AI IDE disable workspace trust by default? Can malicious `.vscode/tasks.json` auto-execute without user confirmation? (Oasis Security — Cursor ships with Workspace Trust disabled)
27. Test for sandbox/container escape via AI agent config: can AI agent installation or skill download flows write files outside intended directories or inject dangerous Docker options? (CVE-2026-27001/27002, OpenClaw — sandbox escape + Docker container escape)
28. Test for TypeScript type confusion sandbox escape: if target uses sandboxed expression evaluation, can TypeScript type annotations (not enforced at runtime) bypass sanitization via destructuring syntax? (CVE-2026-25049, n8n, CVSS 9.4 — novel sandbox escape pattern)
29. Test for MCP server SSRF via unrestricted URI handling: does the MCP server accept arbitrary URIs without validation? In cloud deployments, can instance metadata (169.254.169.254) be queried to steal credentials? (BlueRock "MCP fURI" — 36.7% of 7,000+ MCP servers exposed; Microsoft MarkItDown MCP SSRF -> AWS credential theft)
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
44. Test for MCP stdio command blacklist bypass (WeKnora pattern): if target whitelists `npx`/`uvx` commands but allows flags, test if `-p` flag with `npx node` or similar bypasses the blacklist to execute arbitrary commands. (CVE-2026-30861, CVSS 9.9 — only requires registering an account; fixed in WeKnora v0.2.10)
45. Test for PostgreSQL array/row expression SQL injection bypass: if target sanitizes SQL queries, test if recursive inspection covers child nodes within `ARRAY[]` and `ROW()` expressions. Smuggle dangerous functions (large object operations, library loading) inside nested PostgreSQL expressions. (CVE-2026-30860, CVSS 9.9 — chains with `lo_import`/`lo_get` for file read or `COPY ... FROM PROGRAM` for RCE; fixed in WeKnora v0.2.12)
46. Test for MCP SDK cross-client data leak in stateless mode: if target runs a single `McpServer` instance with `StreamableHTTPServerTransport` across multiple clients, test if responses from one client are visible to another. No-auth servers in multi-client configs leak to anyone. (CVE-2026-25536, CVSS 7.1 — affects SDK v1.10.0-1.25.3; fixed v1.26.0)
47. Test for OpenClaw browser control auth bypass: does the AI agent platform expose browser-control HTTP routes (e.g., `/agent/act`) without mandatory authentication? Can unauthenticated local callers invoke privileged browser operations? (CVE-2026-28485 — versions 2026.1.5 to 2026.2.12)
48. Test for OpenClaw exec approval gating bypass: can authenticated clients bypass execution approval checks by injecting internal approval fields into `node.invoke` parameters? (CVE-2026-28466 — gateway fails to sanitize approval fields, allowing `system.run` without approval)
49. Test for AI agent PATH command hijacking: can environment variable manipulation (PATH prepending) cause the AI agent to execute unintended binaries? Test by setting PATH to include attacker-controlled directory before legitimate paths. (CVE-2026-29610 — affects process execution in agent sandboxes)
50. Test for symlink-based sandbox escape: request bind mounts for non-existent files within symlinked parent directories — does path validation resolve symlinks before checking? (GHSA-M8V2-6WWH-R4GC — validation accepts restricted host paths through symlinked parents)
51. Test for Claude DXT zero-click RCE: if target uses Claude Desktop Extensions, can content from connected services (calendar, email, docs) trigger DXT execution without user interaction? DXT runs with full host privileges, no sandboxing. (LayerX, March 2026 — Anthropic declined to fix; 10,000+ active DXT users)
52. Test for AI-as-C2 proxy: can the AI agent's web browsing capability be used as a bidirectional C2 channel? Post encoded commands on attacker-controlled pages, instruct AI to browse and extract them. No API keys or accounts needed — all traffic appears as normal AI browsing. (Check Point Research, January 2026)
53. Test for MCP tool name collision: does the MCP client use `{service}_{tool}` naming without namespace isolation? Can an attacker register a tool that overwrites a legitimate one (e.g., `tavily_extract`) to redirect LLM execution flow? Test for system prompt exfiltration and context theft via colliding tool names. (CVE-2026-30856, WeKnora, CVSS 5.9 — CWE-706; fixed v0.3.0)
54. Test for exec-approvals shell expansion gap: if the AI tool validates commands against an allowlist before executing, does validation use pre-expansion argv tokens while execution uses real shell expansion? Test if safe binaries like `head`, `tail`, `grep` can read arbitrary files via glob patterns. (CVE-2026-28463, OpenClaw — allowlist checks pre-expansion, execution uses shell expansion)
55. Test for voice webhook verification bypass: if the AI agent has voice/call integration, can attacker forge forwarded headers (e.g., `X-Forwarded-For`) to bypass caller allowlist verification? (CVE-2026-28465, OpenClaw — voice-call plugin webhook bypass)
56. Test for schema drift between MCP server versions: compare `tools/list` responses across minor/patch updates — do new parameters, tools, or capabilities appear without changelog entries? Check if "patch" updates silently introduce shell command parameters or cross-server influence instructions. (ecap0/AgentAudit, March 2026)
57. Test for context pivoting in multi-server MCP deployments: if target connects multiple MCP servers to one agent, can a poisoned response from Server A instruct the agent to call Server B's tools with attacker-controlled parameters? Test cross-server data exfiltration via shared agent context. (ecap0/AgentAudit, February 2026)
58. Test for full schema poisoning (FSP): beyond tool description poisoning, can an attacker modify structural schema elements — hidden parameters, altered return types, malicious default values — that affect all tool invocations while passing description-only scanners? Test if `inputSchema` contains parameters not visible in the tool's UI or documentation. (Adversa AI, March 2026)

---

## MCP OAuth Account Takeover

Multiple one-click account takeover vulnerabilities in Remote MCP servers discovered by Obsidian Security (expanded March 2026). MCP servers acting as both authorization server and OAuth client created CSRF-style attacks leaking authorization codes:

**Root Cause:** The MCP spec treats an MCP server as a resource server, but most are implemented as API wrappers/proxies. MCP servers act as both an authorization server for MCP clients AND as a single OAuth client with one shared static `client_id` to the existing authorization server — the server can't tell who initiated the authorization request versus who completed it.

**How It Works:**
- MCP server sends user to its own authorization endpoint, then silently uses the returned code with a separate OAuth provider
- Missing `state` parameter validation enables CSRF-style attacks against the authorization flow
- Attacker initiates OAuth flow, gets authorization URL, sends to victim — victim completes auth, attacker gets the tokens
- **Affected clients:** Claude Desktop, VS Code, Cursor, Cline, ChatMCP, Cherry Studio, Gemini-CLI, MCP Inspector, Windsurf, Smithery.ai, Lutra.ai, Glue.ai

**Testing Approach:**
1. If MCP server implements OAuth, test for missing `state` parameter in authorization requests
2. Test if authorization codes can be replayed across sessions
3. Check if the MCP server validates redirect URIs strictly
4. Test for mixed role confusion — server acting as both authorization server and OAuth client
5. Check if `state` parameter is bound to a session cookie (the correct fix)
6. Test if the consent flow can be completed by a different user than the one who initiated it (CSRF)
7. Check if MCP server uses a shared static `client_id` — this is the architectural root cause

**Fix Pattern:** Bind the state to a session cookie — set a session cookie when user approves consent, generate state and bind it server-side, verify state matches session on callback.

**Timeline:** Reported July-August 2025; fixed September 2025; MCP spec updated November 25, 2025 to mandate OAuth 2.1 + PKCE. Obsidian Security expanded disclosure March 2026 with additional affected clients.

---

## MCP Real-World Attack Examples

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

4. **Clawdbot/Moltbot Incident (January 2026):** Open-source AI agent framework went viral over the weekend of Jan 24-25. Within 72 hours: exposed admin panels, critical RCE vulnerabilities, and active infostealer campaigns (RedLine, Lumma, Vidar) targeting its configuration directories. 2,000+ exposed gateways visible on Shodan.

5. **ContextCrush (February 2026):** Noma Labs discovered supply chain vulnerability in Context7 (operated by Upstash) — a popular MCP server providing library documentation to AI coding assistants. Attackers could inject malicious instructions through Context7's "Custom Rules" feature. Pattern: trusted documentation -> MCP delivery -> agent execution.

6. **ClawJacked WebSocket Hijack (February 2026):** Malicious websites exploited WebSocket connections to hijack local OpenClaw AI agent instances. Any webpage could connect to the localhost-bound WebSocket and gain full remote control of the local agent.

7. **eval() Epidemic — 7 MCP RCE CVEs in One Month (February 2026):** Seven remote code execution vulnerabilities published in February 2026 alone, all sharing the same root cause: user-controlled input reaching dangerous execution functions (`eval()`, `exec()`, `execAsync()`) without sanitization. Pattern: arbitrary execution is the feature; the question is whether that execution is controllable by adversarial input.

8. **LangGrinch (CVE-2025-68664, CVSS 9.3):** Critical serialization injection in LangChain Core where untrusted LLM-influenced metadata could be rehydrated as objects, enabling secret exfiltration. Impacts ~847M total downloads. LangChain awarded $4,000 — maximum ever for the project.

9. **SANDWORM_MODE Supply Chain Worm (February 2026):** Socket discovered a self-propagating npm worm using 19 typosquatting packages. Key MCP-specific capability: an "McpInject" module deploys a malicious MCP server and injects it into AI coding assistant configs. Pattern: npm install -> MCP injection -> credential theft.

10. **DockerDash Meta-Context Injection (February 2026):** Noma Security disclosed that a single malicious metadata label in a Docker image could compromise the entire Docker environment through Docker's Ask Gordon AI assistant. Noma coined "Meta-Context Injection" for this class where MCP gateways cannot distinguish descriptive metadata from pre-authorized instructions.

11. **Copilot CLI Malware Download (March 2026):** Two days after GitHub Copilot CLI hit general availability, PromptArmor demonstrated `env curl | env sh` bypasses all protections — downloading and executing malware with zero human approval. CVE-2026-29783.

> **Full incident table with 50+ entries:** See [ai-case-studies.md](ai-case-studies.md)

---

## MCP Implementation Vulnerability Stats

- **30+ MCP CVEs filed in just 60 days** — MCP is now AI's fastest-growing attack surface (MCP Security Research, early 2026)
- **43% of MCP servers vulnerable to command execution** (Adversa AI March 2026 aggregate across 500+ servers; 3 demonstrated attack classes with PoC code: external prompt injection, tool prompt injection, cross-tool hijacking)
- **38% of 500+ scanned MCP servers** completely lack authentication (2026 scan)
- **30% permitted unrestricted URL fetching** (SSRF-prone)
- **8,000+ MCP servers** found publicly exposed in February 2026, with 492 identified as vulnerable (lacking authentication or encryption)
- **Attack surface spans three layers:** MCP servers themselves, protocol implementation libraries (SDKs), and the machines running MCP clients — a vulnerability in any layer compromises the entire chain
- **CVE-2025-6514** rated CVSS 10.0 — critical command injection in mcp-remote npm package
- **CVE-2026-27896**: MCP Go SDK vulnerability — JSON parser handles field names case-insensitively
- **CVE-2025-53109**: critical symlink bypass vulnerability enabling modification of privileged files
- **Palo Alto Unit42** published new research on prompt injection attack vectors through MCP sampling — agents with long conversation histories are significantly more vulnerable
- **Adversa AI published MCP Security TOP 25** — definitive catalog of 25 MCP vulnerability categories
- **11 MCP vulnerability classes** systematically cataloged, including supply chain typosquatting and cross-server context abuse
- **MCP Security Bench (MSB)** accepted at ICLR 2026 as academic benchmark for MCP attacks
- VulnerableMCP.info tracks MCP-specific vulnerability database
- **OWASP MCP Top 10** (2026): dedicated security framework for MCP-specific risks
- **CoSAI MCP Security Whitepaper** (January 27, 2026): Coalition for Secure AI released comprehensive taxonomy identifying **12 core threat categories** and **~40 distinct threats**
- **MCP Auth Security**: 88% of MCP servers require credentials, but 53% rely on insecure long-lived static secrets; modern OAuth adoption only 8.5% (Astrix State of MCP Security 2025)
- **Enkrypt AI scan of top 1,000 MCP servers**: 33% had critical vulnerabilities, averaging **5.2 vulnerabilities per server**
- **MCPTox benchmark results**: tool poisoning attack success rates exceed **60%** on models like GPT-4o-mini, o1-mini, DeepSeek-R1, and Phi-4 across 1,312 malicious test cases
- **CyberArk advanced tool poisoning variants**: malicious instructions embedded not just in descriptions but in function names, parameter types, required fields arrays, and default values — bypass description-only scanning
- **MCP protocol now mandates OAuth 2.1 + PKCE** for authentication; Protected Resource Metadata (PRM) mandatory; Resource Indicators required (2025-11-25 spec update)
- **BlueRock MCP Trust Registry** (March 2026): analyzed **7,000+ MCP servers** — **36.7% exposed to SSRF** vulnerabilities
- **Docker MCP Defender + Gateway**: Docker published "MCP Horror Stories" series; MCP Defender provides runtime detection of tool poisoning and data exfiltration; Docker MCP Gateway provides infrastructure-level protection with sandboxed execution
- **Log-To-Leak Framework** (ICLR 2026 submission): new class of prompt-level privacy attacks targeting tool invocation — covertly forces agents to invoke a malicious logging tool to exfiltrate data. Evaluated across 5 real-world MCP servers and 4 LLM agents
- **AgentAudit registry growth** (March 2026): **270+ packages** audited with **247 vulnerabilities** found using multi-agent consensus auditing — growth from 194 packages in early March demonstrates expanding MCP ecosystem audit coverage
- **MCP server audit ecosystem maturing**: 194→270+ packages with systematic scanning; most common patterns remain `child_process.exec()` command injection, credential leakage in error messages/logs, and excessive filesystem permissions
- **PoisonedRAG** (USENIX Security 2025): first knowledge corruption attack on RAG systems — attackers inject semantically meaningful poisoned texts into RAG databases
- **"Lethal Trifecta" Pattern**: repeated pattern across real-world MCP incidents — privileged access + untrusted input + external communication channel
- **5.5% of MCP servers** in the wild exhibit active tool poisoning attacks; **33% allow unrestricted network access** (arXiv research on 2,614 MCP implementations)
- **Endor Labs MCP Analysis** (2,614 implementations): 82% use file system operations prone to Path Traversal (CWE-22), 67% use sensitive APIs related to Code Injection (CWE-94), 34% use APIs related to Command Injection (CWE-78)
- **ETDI** (Enhanced Tool Definition Interface, arXiv:2506.01333): proposed mitigation for tool squatting and rug pull attacks
- **MCP Sampling Attacks** (Palo Alto Unit42, 2026): MCP's sampling feature creates new prompt injection angles — a malicious server becomes an "active prompt author"
- **MCPJam Inspector RCE** (CVE-2026-23744): unauthenticated HTTP endpoint can install arbitrary MCP servers; listens on 0.0.0.0 by default
- **Tenable Cloud & AI Security Risk Report** (2026): 70% of organizations integrated AI/MCP third-party packages without centralized security oversight; 18% granted AI services administrative permissions rarely audited
- **Gravitee State of AI Agent Security 2026**: **3+ million AI agents** operating in corporations; **88% of organizations** reported security incidents; **47% of agents** not actively monitored
- **Endor Labs MCP shell=True pattern**: 34% of 2,614 studied MCP implementations use APIs prone to command injection (CWE-78)
- **Comprehensive MCP server audit** (March 2026): 194 packages audited across 211 independent security reports — **118 findings across 68 packages**. Most common patterns: command injection via unsanitized `child_process.exec()`, credential leakage in error messages/logs/LLM context, excessive filesystem permissions, and missing input validation
- **Elastic Security Labs**: published MCP attack/defense recommendations covering tool poisoning, orchestration injection, and rug-pull redefinitions — actionable defensive guidance for understanding target mitigations
- **Microsoft Spotlighting technique**: protecting against indirect prompt injection in MCP by using delimiters and data marking to help models distinguish instructions from data — potential defense to test against
- **ICON defense mechanism** (March 2026): two-stage defense for indirect prompt injection — detects injection via attention collapse patterns, then uses a Mitigating Rectifier to steer attention away from adversarial tokens. Significantly reduces attack success rates. Test if target deployments use ICON or similar attention-based defenses
- **n8n 6-CVE batch disclosure** (February 2026): six CVEs in a single day covering RCE, command injection, arbitrary file access, and XSS
- **MINJA** (Memory Injection Attack): research demonstrating **95%+ injection success rates** against production AI agents with persistent memory

---

## Denial-of-Wallet via MCP Overthinking Loops

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

## Cursor Rogue MCP Browser Takeover

A rogue MCP server can inject JavaScript into Cursor's built-in browser, replacing login pages with phishing interfaces (CSO Online, 2026):

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

---

## Security MCP Servers for Bug Bounty Workflows

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

---

## MCP Security Scanning Tools

| Scanner | Type | What It Catches | Setup |
|---------|------|-----------------|-------|
| **Cisco MCP Scanner v4.0.1** | CLI (Python, 900+ stars) | YAML/YARA static analysis, bytecode analysis, behavioral dataflow tracing, LLM-as-judge semantic evaluation | `pip install mcp-scanner` |
| **Snyk Agent Scan** | CLI | Prompt injection, tool poisoning, tool shadowing, toxic flows, rug pulls, malware payloads, hardcoded secrets | Auto-discovers Claude Code/Desktop, Cursor, Gemini CLI, Windsurf configs |
| **Repello SkillCheck** | Browser-based | Prompt injection variants, env var exfiltration, policy violations, payload delivery indicators | Upload skill zip, get score (0-100) |
| **MCPGuard** (VirtueAI) | Agent-based | Purpose-built fine-tuned models for MCP-specific patterns; analyzed 700+ servers, found critical vulns in 78% | Maintains scan memory across codebase |
| **BlueRock MCP Trust Registry** | Web platform | Security analysis of 7,000+ MCP servers; SSRF, auth, and tool poisoning detection; provides trust scores | mcp-trust.com |
| **Akto** | Platform | 1,000+ real-world agent exploits; continuous red teaming + runtime guardrails; MCP discovery | Enterprise deployment |
| **AgentAudit** (ecap0) | Registry + CLI | MCP server security registry; multi-agent consensus-based auditing; schema drift detection; 194 packages audited, 118 findings across 68 packages; covers npm, pip, and AgentSkills | agentaudit.dev |
| **Agent-Wiz** (Repello AI) | CLI | Threat modeling and visualization for LangGraph, AutoGen, CrewAI agents | GitHub: `Repello-AI/Agent-Wiz` |

**MCP Security Tools:** mcp-scan v0.4.2 (Invariant Labs, acquired by Snyk Feb 2026 — standard MCP scanner for tool poisoning, rug pulls, cross-origin escalation; static + proxy modes), MCPTox (tool poisoning benchmark), MCPGuard (automated vuln detection), MCP Golf Testing (offensive toolkit), Semgrep MCP Server (code scanning via MCP), Escape ASM (discovers unauthenticated MCP endpoints)

---

## Schema Drift: Silent MCP Attack Surface Expansion

A new MCP attack vector identified by ecap0 (AgentAudit, February-March 2026) where tool schemas silently expand between version updates:

**How It Works:**
- With each MCP server update, tool schemas (descriptions, parameters, capabilities) can change without changelog mention
- v1.0.0 ships with 3 clean tools; v1.0.1 "patch" silently adds a `command` parameter accepting shell commands "for advanced search"
- v1.1.0 adds an `execute_script` tool with descriptions containing instructions that influence LLM behavior with other connected servers
- Previous security audits become outdated with each silent update — organizations running audited MCP servers may unknowingly have expanded attack surfaces

**Testing for Schema Drift:**
1. Compare tool definitions between MCP server versions — diff `tools/list` responses across minor/patch updates
2. Look for new parameters that accept arbitrary strings, shell commands, or file paths added in patch releases
3. Check if new tool descriptions contain cross-server influence instructions
4. Test if "patch" updates introduce entirely new tools with dangerous capabilities
5. Verify if the target has version-pinning or schema validation for MCP servers

**Related Concepts:**
- **Full Schema Poisoning (FSP)**: attackers compromise entire tool schema definitions at the structural level — injecting hidden parameters, altered return types, or malicious default values that affect all subsequent tool invocations while appearing legitimate to monitoring systems. Goes beyond description-only tool poisoning by modifying the schema structure itself
- Maps to MCP06 (Tool Poisoning) + MCP04 (Supply Chain) in OWASP MCP Top 10

---

## Context Pivoting: Lateral Movement via Shared Agent Context

A new MCP attack vector (ecap0/AgentAudit, February 2026) — the MCP equivalent of network lateral movement:

**How It Works:**
- When multiple MCP servers connect to the same AI agent, they share the same execution context — same conversation, same model, same trust boundary
- A malicious MCP server doesn't need to compromise other servers directly — it manipulates the agent into using them
- One poisoned tool response can instruct the agent to call tools from other connected MCP servers with attacker-controlled parameters
- Pattern: `Malicious Server A → poisons agent context → agent calls Server B's tools with attacker input → data exfiltration via Server A`

**Testing for Context Pivoting:**
1. If target connects multiple MCP servers to one agent, test if one server's responses can influence tool calls to other servers
2. Inject instructions in Tool A's response that direct the agent to call Tool B with attacker-controlled parameters
3. Test cross-server data exfiltration: can data from Server B be returned through Server A's channel?
4. Check if servers have isolated execution contexts or share a single trust boundary
5. Verify if the agent validates that tool call parameters match the original user intent, not injected instructions

**Severity Guidance:** Critical in multi-server enterprise deployments where MCP servers have different trust levels (e.g., public documentation server + private database server). Maps to ASI07 (Insecure Inter-Agent Communication) + MCP06 (Tool Poisoning).

---

## Where to Hunt MCP Vulnerabilities

- Any product that integrates MCP servers (Claude Desktop, Cursor, Windsurf, VS Code extensions)
- Enterprise AI agent platforms connecting to internal tools
- Open-source MCP server implementations (check GitHub for `mcp-server-*` repos)
- huntr platform specifically accepts AI/ML vulnerability reports
- VulnerableMCP.info for known MCP-specific vulnerabilities and affected implementations
