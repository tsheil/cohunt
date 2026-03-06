# cohunt

A plugin marketplace for security research and bug bounty hunting. Built for [Claude Cowork](https://claude.com/product/cowork), also compatible with [Claude Code](https://claude.com/product/claude-code).

## Plugins

| Plugin | How it helps | Connectors |
|--------|-------------|------------|
| **[bug-bounty](./bug-bounty)** | Full hunting lifecycle — recon targets, research programs, audit source code, test for vulns, write reports, and plan engagements. Works standalone with curl/dig and web search, supercharged with platform and asset discovery connectors. | HackerOne, Bugcrowd, Shodan, Censys, SecurityTrails, GitHub |

Complements [breach-marketplace](https://github.com/tsheil/breach-marketplace) for source code analysis workflows.

## Getting Started

### Cowork

Install plugins from [claude.com/plugins](https://claude.com/plugins/).

### Claude Code

```bash
# Add the marketplace first
claude plugin marketplace add tsheil/cohunt

# Then install a specific plugin
claude plugin install bug-bounty@cohunt
```

Once installed, plugins activate automatically. Skills fire when relevant, and slash commands are available in your session (e.g., `/bug-bounty:hunt-plan`).

## How Plugins Work

Every plugin follows the same structure:

```
plugin-name/
├── .claude-plugin/plugin.json   # Manifest
├── .mcp.json                    # Tool connections
├── agents/                      # Autonomous multi-step workflows
├── commands/                    # Slash commands you invoke explicitly
└── skills/                      # Domain knowledge Claude draws on automatically
```

- **Agents** are autonomous workflows that orchestrate multiple skills and commands for complex, multi-step tasks (e.g., a full hunt session).
- **Skills** encode the domain expertise, workflows, and methodology Claude needs to give you useful help. Claude draws on them automatically when relevant.
- **Commands** are explicit actions you trigger (e.g., `/bug-bounty:hunt-plan`).
- **Connectors** wire Claude to the external tools your workflow depends on — bug bounty platforms, asset discovery, vulnerability databases, and more — via [MCP servers](https://modelcontextprotocol.io/).

Every component is file-based — markdown and JSON, no code, no infrastructure, no build steps.
