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

## Best connector by skill

Which connectors supercharge which skills:

| Skill | Must-Have Connector | Nice-to-Have |
|-------|-------------------|--------------|
| **target-recon** | Asset discovery (Shodan/Censys) | DNS intelligence, Code search |
| **program-research** | Bug bounty platform | — |
| **vuln-patterns** | Web scanner (Burp/ZAP) | Vulnerability database |
| **source-code-audit** | Code search (GitHub/GitLab) | Code scanner (Semgrep) |
| **api-security** | Web scanner | Code search |
| **auth-testing** | Web scanner | Bug bounty platform (disclosed patterns) |
| **cloud-security** | Cloud security scanner | Asset discovery |
| **mobile-security** | Mobile scanner (MobSF) | Code search |
| **ai-hunting** | Code search | MCP scanner |
| **client-side-security** | Web scanner | Code search |
| **supply-chain-security** | Code search (GitHub) | Vulnerability database |
| **http-desync** | Web scanner (Burp) | — |

## Additional connector recipes

These connectors are not pre-configured in `.mcp.json` but work with the plugin. Add them manually:

### DNS Intelligence (SecurityTrails)

```json
{
  "securitytrails": {
    "command": "npx",
    "args": ["-y", "securitytrails-mcp"],
    "env": {
      "SECURITYTRAILS_API_KEY": "your-key-here"
    },
    "disabled": true
  }
}
```

Supercharges: target-recon (DNS history, related domains, passive DNS, zone data)

### Mobile Security (MobSF)

```json
{
  "mobsf": {
    "command": "uvx",
    "args": ["mobsf-mcp"],
    "env": {
      "MOBSF_API_URL": "http://localhost:8000",
      "MOBSF_API_KEY": "your-key-here"
    },
    "disabled": true
  }
}
```

Supercharges: mobile-security (automated APK/IPA analysis, manifest review, hardcoded secrets)

### GitLab Code Search

```json
{
  "gitlab": {
    "command": "npx",
    "args": ["-y", "@modelcontextprotocol/server-gitlab"],
    "env": {
      "GITLAB_PERSONAL_ACCESS_TOKEN": "",
      "GITLAB_API_URL": "https://gitlab.com/api/v4"
    },
    "disabled": true
  }
}
```

Supercharges: source-code-audit, supply-chain-security (cross-repo pattern search, CI/CD config review)

### Cloud Security (AWS)

```json
{
  "aws-security": {
    "command": "uvx",
    "args": ["aws-mcp"],
    "env": {
      "AWS_PROFILE": "security-audit"
    },
    "disabled": true
  }
}
```

Supercharges: cloud-security (S3 bucket enumeration, IAM policy review, security group audit)
