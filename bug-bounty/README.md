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

All commands work **standalone** (web search + bash tools) and get **supercharged** with MCP connectors.

## Skills

Domain knowledge Claude uses automatically when relevant:

| Skill | Description |
|---|---|
| `target-recon` | Recon a web target — fingerprint tech stack, enumerate subdomains, detect WAF/CDN, map the attack surface |
| `program-research` | Research a bug bounty program — scope, rewards, response times, disclosed reports, and what gets paid |
| `ai-hunting` | AI-assisted hunting workflows and AI/LLM vulnerability patterns — use AI tools to accelerate testing, and hunt bugs in AI-powered features |
| `vuln-patterns` | Vulnerability testing patterns and checklists for common bug classes including AI/LLM vulnerabilities |
| `source-code-audit` | Audit source code for security vulnerabilities — trace data flows, find auth gaps, spot injection sinks |
| `api-security` | API-specific security testing for REST, GraphQL, WebSocket, and gRPC architectures |
| `auth-testing` | Authentication and authorization testing — OAuth, JWT, BOLA, BFLA, privilege escalation |
| `report-writing` | Write high-quality bug bounty reports that get triaged fast and paid well |
| `cloud-security` | Cloud misconfiguration patterns for AWS, GCP, and Azure |
| `mobile-security` | Mobile application security testing for iOS and Android |
| `http-desync` | HTTP request smuggling, web cache poisoning, and race condition testing |

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

See [CONNECTORS.md](CONNECTORS.md) for the full list of supported integrations.

## What's New in v0.16.0

- **IDEsaster campaign coverage** — 30+ AI coding IDE vulnerabilities (24 CVEs), extension recommendation attacks, Chromium flaws in Cursor/Windsurf
- **Supply chain worm patterns** — Shai-Hulud npm supply chain worm, dependency cooldown defenses, 454K malicious packages
- **Cross-agent privilege escalation** — Second-order prompt injection patterns (ServiceNow Now Assist), multi-agent exploitation
- **React2Shell (CVE-2025-55182)** — CVSS 10.0 React Server Components RCE, Flight protocol deserialization
- **Microsoft Entra ID (CVE-2025-55241)** — CVSS 10.0 Global Admin takeover via Actor Tokens
- **Expanded tool coverage** — Burp AI features (Explore Issue, Explainer, BAC filtering), ZAP-MCP integration
- **Updated market intelligence** — Exploitation speed data, API security detection gap stats, updated program payouts
- **36 vulnerability test patterns** — Added AI IDE supply chain, cross-agent injection, supply chain worms, cloud identity token abuse
