# Bug Bounty Plugin

A bug bounty hunting plugin primarily designed for [Cowork](https://claude.com/product/cowork), Anthropic's agentic desktop application — though it also works in Claude Code. Helps with target reconnaissance, program research, and hunt planning. Works with any bug bounty workflow — standalone with curl, dig, and web search, supercharged when you connect your platform and asset discovery tools.

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

The `program-research` skill triggers and provides scope analysis, reward structure, disclosed report patterns, and a go/no-go recommendation.

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

## MCP Integrations

> If you see unfamiliar placeholders or need to check which tools are connected, see [CONNECTORS.md](CONNECTORS.md).

Connect your tools for a richer experience:

| Category | Examples | What It Enables |
|---|---|---|
| **Bug bounty platform** | HackerOne, Bugcrowd, Intigriti | Live scope, real-time rewards, report statistics |
| **Asset discovery** | Shodan, Censys, SecurityTrails | Comprehensive subdomain and service inventory |
| **Vulnerability database** | NVD, Snyk, VulnDB | CVE mapping for detected technology stacks |
| **DNS intelligence** | SecurityTrails, PassiveTotal | Historical DNS, related domains, zone data |

See [CONNECTORS.md](CONNECTORS.md) for the full list of supported integrations.
