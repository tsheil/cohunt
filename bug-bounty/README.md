# Bug Bounty Plugin

A bug bounty hunting plugin primarily designed for [Cowork](https://claude.com/product/cowork), Anthropic's agentic desktop application — though it also works in Claude Code. Helps with target reconnaissance, program research, AI-assisted hunting, and hunt planning. Works with any bug bounty workflow — standalone with curl, dig, and web search, supercharged when you connect your platform and asset discovery tools.

## Installation

```bash
claude plugins add tsheil/cohunt/bug-bounty
```

## Commands

Explicit workflows you invoke with a slash command:

| Command | Description |
|---|---|
| `/hunt-plan` | Build a hunting plan for a target — combine recon and program intel into a prioritized action plan |
| `/recon` | Run reconnaissance on a target — fingerprint tech stack, enumerate subdomains, detect WAF/CDN, map attack surface |
| `/scope-check` | Verify if a target, vulnerability type, or finding is in scope before investing time |
| `/compare-programs` | Compare bug bounty programs side-by-side — rewards, scope, response times, competition |
| `/write-report` | Write a submission-ready bug bounty report — structured, scored, and formatted for the platform |
| `/quick-test` | Rapid single-vulnerability check — focused test plan with exact requests to send |
| `/chain` | Document and score a vulnerability chain — combine findings into a higher-severity attack path |
| `/triage-findings` | Rank bugs by reportability, estimate severity and payout, decide what to report first |
| `/asset-inventory` | Track discovered assets during a hunting session — subdomains, endpoints, technologies, findings |
| `/session-notes` | Track findings, observations, and progress during a hunt session |
| `/methodology` | Generate a testing methodology checklist tailored to a target's tech stack |

All commands work **standalone** (web search + bash tools) and get **supercharged** with MCP connectors.

## Skills

Domain knowledge Claude uses automatically when relevant:

| Skill | Description |
|---|---|
| `target-recon` | Recon a web target — fingerprint tech stack, enumerate subdomains, detect WAF/CDN, map the attack surface |
| `program-research` | Research a bug bounty program — scope, rewards, duplicate patterns, competition, go/no-go recommendation |
| `ai-hunting` | AI-assisted hunting workflows and AI/LLM vulnerability patterns — MCP, agents, IDE extensions, prompt injection |
| `vuln-patterns` | Web vulnerability testing patterns — IDOR, XSS, SSRF, auth bypass, injection, business logic by vertical |
| `source-code-audit` | Audit source code for security vulnerabilities — trace data flows, find auth gaps, spot injection sinks |
| `api-security` | API architecture security — design flaws in REST, GraphQL, WebSocket, and gRPC |
| `auth-testing` | Authentication and authorization flaws — BOLA, BFLA, privilege escalation, IDOR, SSO/MFA bypass |
| `report-writing` | Write high-quality bug bounty reports that get triaged fast and paid well |
| `cloud-security` | Cloud misconfiguration patterns for AWS, GCP, and Azure |
| `mobile-security` | Mobile application security testing for iOS and Android |
| `http-desync` | HTTP request smuggling, web cache poisoning, and race condition testing |

## Agents

Autonomous workflows that run multi-step hunting tasks:

| Agent | Description |
|---|---|
| `hunt-session` | End-to-end hunting orchestrator — prioritizes targets, runs recon, tests vulnerabilities, validates findings |
| `report-review` | Pre-submission quality assurance — validates PoC, checks for AI slop, assesses duplicate risk, platform-specific formatting |

## Example Workflows

### Recon a Target

Just ask naturally:
```
Recon example.com
```

The `target-recon` skill triggers automatically and gives you HTTP fingerprinting, subdomain inventory, security headers assessment, and attack surface summary.

### Research a Program

```
Research the Shopify bug bounty program
```

The `program-research` skill triggers and provides scope analysis, reward structure, disclosed report patterns, AI/automation landscape assessment, and a go/no-go recommendation.

### Hunt AI/LLM Vulnerabilities

```
Test the AI chatbot on example.com for prompt injection
```

The `ai-hunting` skill triggers and walks you through OWASP LLM Top 10 testing patterns — prompt injection, system prompt extraction, output injection, excessive agency, and more.

### Use AI to Hunt Faster

```
Help me use AI to find vulnerabilities in this codebase
```

The `ai-hunting` skill covers AI-assisted workflows — using LLMs for code review (Vulnhalla-style guided questioning), recon synthesis, and payload generation.

### Build a Hunt Plan

```
/hunt-plan shopify.com
```

Combines recon data and program research into prioritized hunt targets, specific test cases, and time budget allocation.

### Full Workflow

```
Research the GitHub bug bounty program
Recon github.com
/hunt-plan github.com
```

Research the program, recon the targets, then generate a complete hunting plan.

## Standalone + Supercharged

Every command and skill works without any integrations:

| What You Can Do | Standalone | Supercharged With |
|-----------------|------------|-------------------|
| Recon targets | curl, dig, web search | Asset discovery MCP (e.g. Shodan, Censys) |
| Research programs | Web search | Bug bounty platform MCP (e.g. HackerOne) |
| Build hunt plans | Web search + user input | Platform + asset discovery MCPs |
| Detect tech stacks | curl, page source analysis | Vulnerability DB MCP for CVE mapping |
| Enumerate subdomains | crt.sh, DNS lookups | Asset discovery MCP for comprehensive inventory |
| AI-assisted hunting | Built-in LLM workflows | Code search MCP for cross-repo analysis |
| Test AI/LLM features | Built-in OWASP LLM Top 10 patterns | Web scanner MCP for automated validation |

## MCP Integrations

> If you see unfamiliar placeholders or need to check which tools are connected, see [CONNECTORS.md](CONNECTORS.md).

Connect your tools for a richer experience:

| Category | Examples | What It Enables |
|---|---|---|
| **Bug bounty platform** | HackerOne, Bugcrowd, Intigriti | Live scope, real-time rewards, report statistics |
| **Asset discovery** | Shodan, Censys, SecurityTrails | Comprehensive subdomain and service inventory |
| **Vulnerability database** | NVD, Snyk, VulnDB | CVE mapping for detected technology stacks |
| **DNS intelligence** | SecurityTrails, PassiveTotal | Historical DNS, related domains, zone data |
| **Code search** | GitHub, GitLab, Sourcegraph | Cross-repo pattern matching for code audit |
| **Web scanner** | Burp Suite, Nuclei, OWASP ZAP | Automated vulnerability validation |
| **Pentest tools** | Kali Linux MCP | AI-driven penetration testing with security tool orchestration |

See [CONNECTORS.md](CONNECTORS.md) for the full list of supported integrations.

## What's New in v0.46.0

- **web-vulns.md structural split** — Split critically oversized `web-vulns.md` (498 lines, 2-line buffer) into `web-vulns.md` (329 lines, API/web patterns) + new `infrastructure-vulns.md` (229 lines, platform/infrastructure patterns), creating ~170 lines of headroom for future growth
- **MSHTML Mark-of-the-Web bypass chain** — CVE-2026-21513 (CVSS 8.8, APT28 active exploitation) + CVE-2026-21510/21514/21519/21533 cluster; 5 test patterns for MotW bypass detection in new `infrastructure-vulns.md`
- **3 new AI security tools** — Repello AI SkillCheck (browser-based skill security scanner), Noma MCP Server Security (MCP deployment discovery), Crust (agent security infrastructure with runtime behavioral analysis) added to `tools-landscape.md`
- **Krebs on Security OpenClaw exposure** — 42,665 exposed admin interfaces; pattern: scan for exposed admin → credential harvest → lateral movement; added to case studies and hunt-session agent
- **Apple iOS zero-day** — CVE-2026-20700 (CVSS 7.8, Google TAG discovery) dyld zero-day exploit chain added to case studies and market context
- **Hunt-session agent updated** — Windows MotW bypass chain and exposed AI agent admin panel vectors added to human-vs-AI edge matrix; infrastructure-vulns.md routing added
- **Market intelligence** — Cycode 62% exploitable LLM code stat, February 2026 Windows MotW bypass cluster, 4 new notable incident entries

## What's New in v0.44.0

- **Advanced recon techniques** — Favicon hash enumeration (Shodan/Censys infrastructure discovery), origin IP discovery via Certificate Transparency logs (WAF bypass), copyright notice mining for related domains, and virtual host fuzzing for hidden services in `target-recon`
- **DockerDash MCP attack pattern** — Docker's Ask Gordon AI compromised via single malicious image metadata label; two attack paths (RCE + exfiltration) via MCP Gateway; new section in `ide-supply-chain.md` with testing approach
- **MDM pre-auth RCE section** — Ivanti EPMM CVE-2026-1281/1340 (both CVSS 9.8) mass exploitation pattern; new section in `web-vulns.md` covering enterprise MDM attack surfaces
- **GraphQL CSRF via Content-Type switching** — Cross-origin mutation exploitation when GraphQL endpoints accept form-encoded requests; added to `web-vulns.md` GraphQL section
- **Burp Suite MCP + Kali Linux MCP connectors** — Two new MCP integrations for hands-on testing added to `.mcp.json` and `CONNECTORS.md`; Burp for proxy history + HTTP request manipulation, Kali for AI-driven penetration testing
- **5 new case study entries** — Ivanti EPMM mass exploitation, XBOW #1 globally, Apple $2M max bounty, Microsoft Zero Day Quest $1.6M payout, plus market intelligence updates

## What's New in v0.43.0

- **WeKnora 5-CVE multi-tenant compromise** — CVE-2026-30855 (CVSS 8.8) broken access control in Tencent's LLM framework; open registration → cross-tenant ATO + LLM API key leakage from tenant configs; plus DNS rebinding SSRF (CVE-2026-30858) and cross-tenant KB cloning (CVE-2026-30857). Pattern: multi-tenant AI platforms with open registration are goldmines
- **RustDesk 7+ CVE cluster** — Session replay auth bypass (CVE-2026-30789), missing authorization, MitM, cleartext transmission, broken crypto, plus pre-auth SSRF enabling internal port scanning before password verification. New self-hosted remote desktop attack section in `web-vulns.md`
- **Trail of Bits Claude Code security skills** — Battle-tested audit methodology from Testing Handbook as Claude Code plugin marketplace; `claude-code-config` for opinionated sandboxing defaults. Added to `tools-landscape.md`
- **Cisco Skill Scanner** — Companion to MCP Scanner; static + behavioral analysis for AI agent skill files (OpenClaw, Claude Skills, Codex skills); VirusTotal integration
- **Adversa AI MCP ecosystem growth** — 1,412 MCP servers indexed (232% increase in 6 months); updated in market context
- **6 new case study entries** — WeKnora multi-tenant, RustDesk cluster, Trail of Bits skills marketplace, plus market intelligence updates across both market-context files and program-research

## What's New in v0.38.0

- **OWASP Agentic Top 10 test procedures** — 19 concrete test procedures (T1-T19) for ASI01-ASI10 risks in `agent-attack-patterns.md`, covering goal hijacking, tool misuse, privilege escalation, memory poisoning, inter-agent impersonation, cascading failures, trust exploitation, and rogue agent detection
- **Microsoft "AI as Tradecraft" case study** — Coral Sleet fully AI-enabled malware workflow, LLM jailbreaking for code gen, AI code fingerprints (emojis as markers, conversational comments), first documented agentic AI experimentation by state-sponsored threat actors
- **Codex Security Research Preview stats** — Updated with March 6, 2026 launch data: 792 critical + 10,561 high findings from 1.2M commits, 50%+ FP reduction, 90% severity over-reporting reduction, sandboxed validation, project-specific threat models
- **Checkmarx 11 MCP Risks taxonomy** — Cross-referenced in `mcp-playbooks.md` alongside OWASP MCP Top 10, adding client-side and deserialization vectors
- **Market intelligence** — Codex Security launch, OWASP Agentic Top 10 finalization, Checkmarx MCP risk taxonomy, Microsoft AI threat actor tradecraft

## What's New in v0.37.0

- **Edge framework path normalization bypass** — CVE-2026-29045 (Hono) auth bypass via `decodeURI` vs `decodeURIComponent` mismatch, with 5 test patterns in `web-vulns.md`
- **SSRF via webhook/notification/import cluster** — 5-CVE cluster (Soft Serve 9.1, Ghostfolio 9.3, Wallos 8.8, Plane 8.5, PinchTab 7.5) with incomplete IP validation patterns
- **GraphQL WebSocket subscription depth bypass** — CVE-2026-30241 (Mercurius) transport-inconsistent depth limiting pattern
- **OAuth first-party app trust abuse (ConsentFix)** — Azure CLI trust bypass for MFA circumvention
- **HTTP/3 race conditions (QUICker)** — First HTTP/3-specific race condition testing patterns
- **Langflow CSV Agent RCE** — CVE-2026-27966 (CVSS 9.8) hardcoded dangerous defaults pattern in `ai-mcp-vulns.md`
- **MCP Security Bench stats** — 40.71% average attack success rate (ICLR 2026), 30 MCP CVEs in 60 days
- **PleaseFix zero-click exfiltration** — Zenity Labs agentic browser calendar invite → 1Password credential theft pattern
- **Unit42 in-the-wild IDPI catalog** — 22 documented indirect prompt injection techniques including first ad review evasion
- **Advanced jailbreak techniques** — Jailbreak Foundry (papers-to-attacks), TAO-Attack two-stage optimization, LRM autonomous jailbreak (97.14% success)
- **6 new AI security tools** — Penetrify, GHOSTCREW, OpenAnt, Jailbreak Foundry, QUICker, Simbian added to tools landscape
- **Market intelligence** — Bugcrowd FedRAMP, HackerOne AI Safe Harbor, Wallarm +400% AI API threats, curl bounty shutdown

## What's New in v0.28.0

- **TypeScript runtime type erasure pattern** — New critical hunt pattern in `framework-patterns.md` for sandbox escapes via TS type confusion (CVE-2026-25049, n8n, CVSS 9.8) with code examples and audit guidance
- **Content-Type confusion pattern** — Added n8n Ni8mare CVE-2026-21858 (CVSS 10.0) request handler exploitation pattern to framework patterns
- **11 new IDEsaster CVEs** — Cursor CVE-2026-22708/26268, Copilot CVE-2026-21257/21523, OpenClaw CVE-2026-28485/28462/28466/28478/29610 added to agent attack patterns
- **Claude DXT zero-click RCE** — LayerX disclosure (CVSS 10.0, Anthropic declined to fix) and AI-as-C2 proxy (Check Point Research) patterns added
- **MCP OAuth expansion** — Root cause analysis, 6 additional affected clients, 7 new test procedures for Obsidian Security MCP OAuth CSRF findings
- **7 new MCP vulnerability classes** — OpenClaw browser control, exec bypass, PATH hijack, symlink escape; MCP Go SDK case sensitivity (CVE-2026-27896)
- **6 new MCP test procedures** (#47-52) — Browser control auth bypass, exec approval gating, PATH hijacking, symlink sandbox escape, DXT RCE, AI-as-C2
- **VoidLink AI malware** — First AI-generated advanced malware (88K lines, Check Point Research) added to case studies and market context
- **Reference file consolidation** — Merged `advanced-web-vulns.md` into `web-vulns.md` (eliminated duplicate GraphQL/JWT/OAuth/rate limiting content); SKILL.md routing updated
- **n8n `with` sandbox escape** — CVE-2026-1470 (CVSS 9.9) JavaScript deprecated `with` statement scope chain manipulation added across case studies and framework patterns

## What's New in v0.27.0

- **source-code-audit progressive disclosure** — Extracted framework-specific patterns to `reference/framework-patterns.md` (187 lines), reducing SKILL.md from 469→353 lines with room to grow
- **WeKnora CVE patterns** — Added MCP stdio blacklist bypass (CVE-2026-30861, CVSS 9.9) and PostgreSQL array expression SQLi bypass (CVE-2026-30860, CVSS 9.9) test procedures to `mcp-playbooks.md`
- **New MCP vulnerability classes** — MCP Stdio Blacklist Bypass, SQL Expression Tree Bypass, JavaScript `with` Sandbox Escape (CVE-2026-1470) added to playbooks and framework patterns
- **Hacktron AI BeyondTrust RCE** — CVE-2026-1731 (CVSS 9.9) pre-auth RCE discovery added to case studies, tools landscape, and program-research
- **Anthropic mcp-server-git chain** — CVE-2025-68145/68143/68144 RCE chain added to case studies and disclosed vulnerabilities
- **MCP SDK cross-client data leak** — New test procedure (#46) for stateless mode session isolation failures
- **TOC and cross-references** — Added Table of Contents to `tools-landscape.md` and related-files headers across reference files
- **XBOW stats reconciled** — Clarified "1,060 reports in ~90 days" vs "cumulative 1,400+" across all references
