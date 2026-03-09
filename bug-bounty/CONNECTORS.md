# Connectors

## How tool references work

Plugin files use `~~category` as a placeholder for whatever tool the user connects in that category. For example, `~~bug bounty platform` might mean HackerOne, Bugcrowd, or any other platform with an MCP server.

Plugins are **tool-agnostic** — they describe workflows in terms of categories (bug bounty platform, vulnerability database, asset discovery, etc.) rather than specific products. The `.mcp.json` pre-configures specific MCP servers, but any MCP server in that category works.

## Connectors for this plugin

| Category | Placeholder | Included servers | Other options |
|----------|-------------|-----------------|---------------|
| Bug bounty platform | `~~bug bounty platform` | bug-bounty-mcp (disabled) | HackerOne, Bugcrowd, Intigriti, YesWeHack |
| Vulnerability database | `~~vulnerability database` | Snyk (disabled) | NVD, CVE.org, VulnDB |
| Asset discovery | `~~asset discovery` | Shodan (disabled), Censys (disabled) | SecurityTrails, FullHunt |
| DNS intelligence | `~~dns intelligence` | — | SecurityTrails, PassiveTotal, VirusTotal |
| Code search | `~~code search` | GitHub (disabled) | GitLab, Sourcegraph |
| Code security scanner | `~~code scanner` | Semgrep (disabled) | SonarQube, CodeQL |
| Web application scanner | `~~web scanner` | Burp Suite MCP (disabled) | Nuclei, OWASP ZAP |
| Penetration testing | `~~pentest tools` | Kali Linux MCP (disabled) | Individual tool MCPs (Nmap, SQLMap) |

## Enabling a connector

Each connector in `.mcp.json` is disabled by default. To enable one:

1. Add your API key to the relevant environment variable
2. Set `"disabled": false` for that server
3. Restart your session

For example, to enable Shodan:

```json
{
  "shodan": {
    "command": "uvx",
    "args": ["mcp-shodan"],
    "env": {
      "SHODAN_API_KEY": "your-key-here"
    },
    "disabled": false
  }
}
```

In Cowork, connectors can also be enabled through the plugin settings UI.
