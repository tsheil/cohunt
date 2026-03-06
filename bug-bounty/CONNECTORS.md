# Connectors

## How tool references work

Plugin files use `~~category` as a placeholder for whatever tool the user connects in that category. For example, `~~bug bounty platform` might mean HackerOne, Bugcrowd, or any other platform with an MCP server.

Plugins are **tool-agnostic** — they describe workflows in terms of categories (bug bounty platform, vulnerability database, asset discovery, etc.) rather than specific products. The `.mcp.json` pre-configures specific MCP servers, but any MCP server in that category works.

## Connectors for this plugin

| Category | Placeholder | Included servers | Other options |
|----------|-------------|-----------------|---------------|
| Bug bounty platform | `~~bug bounty platform` | — | HackerOne, Bugcrowd, Intigriti, YesWeHack |
| Vulnerability database | `~~vulnerability database` | — | NVD, CVE.org, VulnDB, Snyk |
| Asset discovery | `~~asset discovery` | — | Shodan, Censys, SecurityTrails, FullHunt |
| DNS intelligence | `~~dns intelligence` | — | SecurityTrails, PassiveTotal, VirusTotal |
| Code search | `~~code search` | — | GitHub, GitLab, Sourcegraph |
| Web application scanner | `~~web scanner` | — | Burp Suite, Nuclei, OWASP ZAP |
