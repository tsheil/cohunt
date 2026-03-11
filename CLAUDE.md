# Cohunt — Bug Bounty Hunting Plugin for Claude Code

## Project Structure

This is a **file-based Claude Code plugin** with no code or infrastructure. All content is markdown and JSON.

```
bug-bounty/
├── .claude-plugin/plugin.json    # Plugin metadata and version
├── skills/                        # 16 domain knowledge modules (SKILL.md files)
│   ├── ai-hunting/               # AI-assisted hunting + LLM vuln patterns (progressive disclosure)
│   ├── vuln-patterns/            # Testing patterns for 50+ vuln classes
│   ├── infrastructure-security/  # Network appliances, firewalls, MDM, backup, workflow automation
│   ├── business-logic/           # Payment flows, state machines, subscription bypass (#1 bounty class)
│   ├── program-research/         # Bug bounty program intelligence
│   ├── target-recon/             # Web target reconnaissance
│   ├── source-code-audit/        # Security code review (progressive disclosure)
│   ├── api-security/             # API-specific testing (REST, GraphQL, gRPC) (progressive disclosure)
│   ├── auth-testing/             # Auth & authorization testing
│   ├── cloud-security/           # Cloud misconfigurations (AWS, GCP, Azure, AI/ML services) (progressive disclosure)
│   ├── mobile-security/          # Mobile app testing (iOS, Android)
│   ├── http-desync/              # HTTP smuggling, cache poisoning, race conditions
│   ├── client-side-security/     # Browser/frontend security (DOM XSS, CSP, XS-Leaks)
│   ├── supply-chain-security/    # CI/CD, dependency confusion, GitHub Actions
│   ├── file-processing/          # File upload, import, archive extraction, parser abuse
│   └── report-writing/           # Bug bounty report structure
├── agents/                        # 2 autonomous workflow agents (flat .md files)
│   ├── hunt-session.md           # End-to-end hunting orchestrator
│   └── report-review.md          # Pre-submission quality assurance
└── commands/                      # 18 slash commands
```

## Key Conventions

- **Versioning**: Semantic, committed as `v0.X.0: Summary of changes`
- **Skills**: Each skill has a `SKILL.md` with YAML frontmatter (name, description, trigger phrases)
- **Progressive Disclosure**: Large skills use `reference/` subdirectories to stay under 500 lines in SKILL.md
- **Agents**: Flat `.md` files with frontmatter (name, description, examples, model, color)
- **No code**: Everything is markdown/JSON — no scripts, no build process, no dependencies
- **License**: Apache 2.0

## Development Guidelines

- Make targeted edits to existing files; avoid full rewrites
- When adding new vulnerability patterns, include: CVE reference, CVSS score, affected products, testing procedure, severity guidance
- Agent edge cases should reference specific CVEs or incidents for concreteness
- Market stats should include source attribution (e.g., "Bugcrowd 2026", "HackerOne 9th Annual Report")
- Bump version in `plugin.json` after each update batch
- Commit messages follow: `v0.X.0: Summary of changes` with detailed bullets per skill updated

## Research Sources

Primary: HackerOne reports, Bugcrowd reports, SecurityWeek, The Hacker News, arXiv papers, NVD/CVE databases, vendor security blogs (Check Point Research, Snyk, Adversa AI, Wiz, Tenable)
