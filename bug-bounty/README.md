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

## Where Do I Start?

| Your Situation | Use This | What Happens |
|---------------|----------|-------------|
| **New to a target** | `/start-hunt <target>` | Routes you to the right workflow based on your goal |
| **Want a full hunt setup** | `hunt-session` agent | End-to-end: research → recon → scope → plan |
| **Know the target, need a plan** | `/hunt-plan <target>` | Prioritized test cases from recon + research data |
| **Have a specific endpoint** | `/quick-test <vuln> <url>` | Focused test plan with exact requests |
| **Stuck or blocked mid-hunt** | `/pivot` | Next 3 highest-value test angles |
| **Found something, need a report** | `/write-report <finding>` | Submission-ready report with CVSS + PoC |
| **Multiple findings to rank** | `/triage-findings` | Priority ranking by reportability and payout |
| **Want to chain findings** | `/chain` | Document and score a multi-step attack path |
| **Report ready for QA** | `report-review` agent | Validates PoC, checks CVSS, catches N/A triggers |
| **Testing AI/LLM features** | Ask about prompt injection, AI security | `ai-hunting` skill triggers automatically |
| **Testing mobile app** | Ask about iOS/Android testing | `mobile-security` skill triggers automatically |
| **Need a payload** | `/generate-payload <type> <context>` | WAF-aware, encoded, ready-to-use payloads |

For full release history, see [CHANGELOG.md](CHANGELOG.md).
