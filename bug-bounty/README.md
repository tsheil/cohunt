# Bug Bounty Plugin

A bug bounty hunting plugin primarily designed for [Cowork](https://claude.com/product/cowork), Anthropic's agentic desktop application — though it also works in Claude Code. Covers the full hunting lifecycle: recon targets, research programs, test for vulnerabilities, write reports, and plan engagements. Works standalone with curl, dig, and web search — supercharged when you connect your platform and asset discovery tools.

## Installation

```bash
claude plugins add tsheil/cohunt/bug-bounty
```

## Commands

Explicit workflows you invoke with a slash command:

| Command | Description |
|---|---|
| `/recon` | Run reconnaissance on a target — fingerprint tech stack, enumerate subdomains, detect WAF/CDN, map the attack surface |
| `/hunt-plan` | Build a hunting plan for a target — combine recon and program intel into a prioritized action plan |
| `/scope-check` | Quick scope check — verify if a target or finding is in scope before you invest time |
| `/triage-findings` | Rank and prioritize your findings — estimate severity, assess reportability, and decide what to report first |
| `/write-report` | Write a submission-ready bug bounty report — structured, scored, and formatted for the platform |
| `/compare-programs` | Compare bug bounty programs side-by-side — rewards, scope, response times, competition — and get a recommendation |
| `/methodology` | Generate a testing methodology checklist tailored to a target's tech stack, attack surface, and your focus area |
| `/session-notes` | Track findings, observations, and progress during a hunting session — log as you go, export when done |
| `/asset-inventory` | Track discovered assets during a session — subdomains, endpoints, technologies, findings — organized and exportable |
| `/quick-test` | Rapid single-vulnerability check — give a target and vuln class, get exact requests to send and what to look for |
| `/chain` | Document and score a vulnerability chain — combine multiple findings into a higher-severity attack path |

All commands work **standalone** (web search + bash tools) and get **supercharged** with MCP connectors.

## Agents

Autonomous workflows that orchestrate multiple skills:

| Agent | Description |
|---|---|
| `hunt-session` | Run a complete hunting session — research the program, recon targets, analyze scope, and build a prioritized plan in one go |
| `report-review` | Review a draft report before submission — check completeness, validate CVSS, audit reproduction steps, and get a go/no-go recommendation |

## Skills

Domain knowledge Claude uses automatically when relevant:

| Skill | Description |
|---|---|
| `target-recon` | Recon a web target — fingerprint tech stack, enumerate subdomains, detect WAF/CDN, map the attack surface |
| `program-research` | Research a bug bounty program — scope, rewards, response times, disclosed reports, and what gets paid |
| `report-writing` | Write high-quality bug bounty reports that get triaged fast and paid well |
| `vuln-patterns` | Vulnerability testing patterns and checklists for common bug classes, tailored to the target's tech stack |
| `source-code-audit` | Audit source code for security vulnerabilities — trace data flows, find auth gaps, spot injection sinks |
| `http-desync` | HTTP request smuggling, web cache poisoning, race conditions — protocol-level attacks against proxies and caches |
| `cloud-security` | Cloud misconfiguration patterns for AWS, GCP, and Azure — storage, metadata, IAM, serverless, and Kubernetes |
| `mobile-security` | Mobile app security testing — API key extraction, certificate pinning, deep links, local storage, WebView, and platform-specific patterns for iOS and Android |
| `auth-testing` | Authentication and authorization testing — OAuth, JWT, session management, IDOR, privilege escalation, BOLA, BFLA, SSO bypass, MFA bypass, and access control patterns |
| `api-security` | API architecture-specific testing — REST versioning abuse, GraphQL introspection and query attacks, WebSocket hijacking, gRPC reflection, batch endpoint abuse, and rate limiting bypass |

## Example Workflows

### Run a Hunt Session

The fastest way to get started on a new target:
```
I want to hunt on Shopify this weekend
```

The `hunt-session` agent kicks off automatically — researches the program, recons the targets, checks scope, and delivers a complete session brief with prioritized test cases.

### Recon a Target

Use the command or ask naturally:
```
/recon example.com
```

Gives you HTTP fingerprinting, subdomain inventory, security headers assessment, and attack surface summary.

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

### Check Scope

```
/scope-check staging.shopify.com for Shopify
```

Quickly verify whether a target or vulnerability type is in scope before you test or report.

### Get Testing Patterns

```
How do I test for IDOR on this REST API?
```

The `vuln-patterns` skill triggers and provides concrete test cases, payloads, and bypasses for the vulnerability class — tailored to the tech stack if recon data is available.

### Audit Source Code

```
Audit this Express middleware for auth bypasses
```

The `source-code-audit` skill triggers and traces data flows, checks authentication logic, identifies injection sinks, and flags framework-specific anti-patterns.

### Write a Report

```
/write-report IDOR on api.example.com/users/{id} — can read other users' profiles
```

Or ask naturally — "Help me write up this SSRF I found on api.example.com" — and the `report-writing` skill triggers automatically. Both produce a complete, submission-ready report with structured reproduction steps, CVSS scoring, and impact assessment.

### Compare Programs

Deciding where to hunt:
```
/compare-programs Shopify vs GitHub — I specialize in API bugs and have 10 hours
```

Side-by-side comparison of rewards, scope, competition, and a recommendation tailored to your skills and time.

### Test HTTP-Level Bugs

```
This target uses Cloudflare + nginx — what desync attacks should I try?
```

The `http-desync` skill triggers with smuggling, cache poisoning, and race condition patterns specific to the proxy/cache stack.

### Generate a Testing Methodology

Get a step-by-step testing checklist tailored to your target:
```
/methodology api.example.com --focus API
```

Or let it build from recon data:
```
/methodology shopify.com
```

Produces a phased testing plan with concrete test cases, time estimates, tool recommendations, and quick wins — customized to the detected tech stack.

### Test Cloud Security

When a target is cloud-hosted:
```
This app is on AWS — check for S3 misconfigs and metadata exposure
```

The `cloud-security` skill triggers with storage enumeration, SSRF-to-metadata paths, IAM testing patterns, and serverless-specific checks for the detected cloud provider.

### Review a Report Before Submitting

Get a pre-submission quality check:
```
Review my report before I submit it
```

The `report-review` agent validates your CVSS scoring, audits reproduction steps, checks for common N/A triggers, and gives a go/no-go recommendation with specific improvements.

### Triage Your Findings

```
/triage-findings I found an IDOR on /api/users, reflected XSS on /search, and an open redirect on /login
```

Ranks your findings by severity, reportability, and duplicate risk — tells you what to report first and what needs more work.

### Test Mobile Apps

When the scope includes mobile applications:
```
The scope includes their iOS app — what should I test?
```

The `mobile-security` skill triggers with static analysis patterns, certificate pinning bypass techniques, deep link abuse vectors, local storage checks, and mobile-specific API patterns for the target platform.

### Chain Vulnerabilities

Turn multiple findings into a higher-severity chain:
```
/chain Open redirect on /oauth/authorize + OAuth code theft → full account takeover
```

Or describe what you found:
```
/chain I found SSRF on /api/preview, cloud metadata is accessible, and AWS creds are in the response
```

Maps the attack flow, scores the chain as a single finding (CVSS 3.1 + 4.0), flags weak links, and produces a cohesive report.

### Test API Architecture

When the target is API-heavy:
```
This target has a GraphQL API at /graphql — what should I test?
```

The `api-security` skill triggers with introspection checks, query depth attacks, alias-based batching, subscription abuse, and authorization bypass patterns specific to GraphQL. Also covers REST versioning abuse, WebSocket hijacking, and gRPC reflection.

### Test Authentication & Authorization

The highest-paid bug class:
```
How do I test OAuth on this target? The login uses Google SSO
```

The `auth-testing` skill triggers with OAuth misconfigurations, token theft patterns, account linking flaws, and SSO-specific bypass techniques. Also covers BOLA/BFLA, JWT attacks, MFA bypass, and privilege escalation.

### Quick Vulnerability Check

Fast, focused test for a single vulnerability:
```
/quick-test api.example.com/users/{id} IDOR
```

Get exact curl commands to copy-paste, what response confirms the bug, how to rule out false positives, and next steps if confirmed.

### Track Discovered Assets

Keep a running inventory of everything you find:
```
/asset-inventory add subdomain api.staging.example.com
/asset-inventory add endpoint POST /api/v2/users
/asset-inventory add finding IDOR on /api/users/{id}
/asset-inventory view
/asset-inventory export
```

Organized by type — subdomains, endpoints, technologies, credentials, findings, and notes.

### Track Your Session

Keep running notes while hunting:
```
/session-notes add Found IDOR on /api/users/{id} — needs more testing
/session-notes add Tested auth bypass on /login — rate limiting blocks it
/session-notes view
/session-notes export
```

Log findings, observations, and tested areas as you go. Export a clean summary when you're done — ready for triage or follow-up.

### Full Workflow

The quick way — let the agent handle setup:
```
Set me up to hunt on GitHub
```

Or step by step:
```
/compare-programs GitHub vs GitLab — I'm good at API bugs
Research the GitHub bug bounty program
/recon github.com
/scope-check api.github.com for GitHub
/asset-inventory add subdomain api.github.com
Audit this controller for auth issues [paste code]
How do I test for IDOR on GitHub's API?
/quick-test api.github.com/users/{id} IDOR
This uses Cloudflare + nginx — any smuggling or cache bugs to try?
/hunt-plan github.com
[... hunt and find bugs ...]
/asset-inventory view
/triage-findings [describe what you found]
/chain [if findings connect into a higher-severity attack path]
/write-report IDOR on api.github.com/users/{id} — can read other users' data
```

Compare programs, pick one, research it, recon the targets, verify scope, track assets, audit source code, get testing patterns, run quick tests, test HTTP-level bugs, build a hunt plan, review your asset inventory, triage your findings, chain related bugs, then write reports.

## Standalone + Supercharged

Every command and skill works without any integrations:

| What You Can Do | Standalone | Supercharged With |
|-----------------|------------|-------------------|
| Recon targets | curl, dig, web search | Asset discovery MCP (e.g. Shodan, Censys) |
| Research programs | Web search | Bug bounty platform MCP (e.g. HackerOne) |
| Build hunt plans | Web search + user input | Platform + asset discovery MCPs |
| Detect tech stacks | curl, page source analysis | Vulnerability DB MCP for CVE mapping |
| Enumerate subdomains | crt.sh, DNS lookups | Asset discovery MCP for comprehensive inventory |
| Check scope | Web search for program page | Platform API for live scope data |
| Test for vulns | Built-in patterns and payloads | Vulnerability DB for CVE-informed testing |
| Triage findings | Severity estimation, reportability assessment | Platform API for duplicate checking |
| Audit source code | Data flow tracing, framework patterns | Code search MCP for cross-repo analysis |
| Write reports | Structured templates, CVSS 3.1 + 4.0 scoring | Platform API for auto-submission |
| Compare programs | Web search, disclosed report analysis | Platform API for live stats and payouts |
| Test HTTP desync | Built-in smuggling, cache, and race patterns | Web scanner for automated detection |
| Test cloud security | curl, DNS, web search | Asset discovery + vulnerability DB MCPs |
| Generate methodology | Heuristic prioritization from recon | Platform + asset discovery + vulnerability DB MCPs |
| Review reports | Built-in checklist, CVSS validation | Platform API for program-specific guidance |
| Test mobile apps | Static analysis, API extraction, deep link testing | Code search MCP for cross-version analysis |
| Test auth & authorization | BOLA, BFLA, OAuth, JWT, MFA, SSO patterns | Platform API for scope-aware auth testing |
| Quick vulnerability checks | Copy-paste curl commands, confirmation steps | — |
| Test API architectures | REST, GraphQL, WebSocket, gRPC patterns | Asset discovery + code search MCPs |
| Chain vulnerabilities | Attack flow mapping, combined CVSS scoring | Platform API for duplicate checking |
| Track discovered assets | Categorized inventory, structured export | — |
| Track session progress | In-conversation notes, categorized findings | — |

## MCP Integrations

> If you see unfamiliar placeholders or need to check which tools are connected, see [CONNECTORS.md](CONNECTORS.md).

Connect your tools for a richer experience:

| Category | Examples | What It Enables |
|---|---|---|
| **Bug bounty platform** | HackerOne, Bugcrowd, Intigriti | Live scope, real-time rewards, report statistics |
| **Asset discovery** | Shodan, Censys, SecurityTrails | Comprehensive subdomain and service inventory |
| **Vulnerability database** | NVD, Snyk, VulnDB | CVE mapping for detected technology stacks |
| **DNS intelligence** | SecurityTrails, PassiveTotal | Historical DNS, related domains, zone data |
| **Code search** | GitHub, GitLab, Sourcegraph | Cross-repo pattern matching, commit history search |

See [CONNECTORS.md](CONNECTORS.md) for the full list of supported integrations.
