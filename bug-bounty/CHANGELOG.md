# Changelog

## v0.48.0

- **2 new skills** — `client-side-security` (DOM XSS, prototype pollution, PostMessage, XS-Leaks, CSP/CORS bypass, WebSocket, service workers, SPA-specific) and `supply-chain-security` (GitHub Actions, GitLab CI, dependency confusion, artifact poisoning, webhooks, container security, runner trust)
- **3 new commands** — `/start-hunt` (routing entrypoint for new users), `/pivot` (mid-hunt recovery when stuck), `/generate-payload` (context-aware, WAF-aware payload generation)
- **auth-testing progressive disclosure** — Extracted OAuth, JWT, session, MFA, password reset, SSO/SAML, rate limiting checklists to `reference/auth-mechanisms.md`, reducing SKILL.md from 446→200 lines
- **Hunt-session stateful checklists** — Agent now outputs markdown progress tracker at top of every response for workflow transparency
- **Session-notes → report pipeline** — Structured finding format in `/session-notes` feeds directly into `/write-report` with zero information loss
- **CLI tool parsing strategies** — New section in `target-recon` with tool-specific output flags to prevent context window bloat
- **vuln-patterns routing index** — Added routing table at top of SKILL.md directing to 11 specialized skills and reference files
- **Connector expansion** — Added "best connector by skill" matrix and 4 new connector recipes (SecurityTrails, MobSF, GitLab, AWS) to CONNECTORS.md
- **README decision matrix** — Replaced changelog with "Where Do I Start?" routing table for new users; changelog moved to CHANGELOG.md
- **Shared reference files** — Common output format schemas and prioritization rubrics extracted for cross-command consistency

## v0.47.0

- **RoguePilot full attack chain** (symlink + $schema exfil → repo takeover) expanded in ide-supply-chain
- **CVE-2026-29783 Copilot CLI** bash parameter expansion details (4 dangerous patterns)
- **Codex Security 14 CVEs** updated metrics (792 critical, 10,561 high)
- **DNS rebinding SSRF bypass** (CVE-2026-27127 Craft CMS TOCTOU) new section in web-vulns
- **Cursor Automations** + **Teramind AI Governance** in tools-landscape
- **11 new market entries** (HackerOne $81M, AI vuln 540% surge, XBOW economics, ArmorCode $81M)
- **3 new hunt-session attack vectors** (RoguePilot, shell expansion bypass, DNS rebinding SSRF)

## v0.46.0

- **web-vulns.md structural split** — Split into `web-vulns.md` (329 lines) + new `infrastructure-vulns.md` (229 lines)
- **MSHTML Mark-of-the-Web bypass chain** — CVE-2026-21513 (CVSS 8.8, APT28 active exploitation) + cluster
- **3 new AI security tools** — Repello AI SkillCheck, Noma MCP Server Security, Crust
- **Krebs on Security OpenClaw exposure** — 42,665 exposed admin interfaces
- **Apple iOS zero-day** — CVE-2026-20700 (CVSS 7.8, Google TAG)

## v0.44.0

- **Advanced recon techniques** — Favicon hash, origin IP discovery, copyright mining, vhost fuzzing
- **DockerDash MCP attack pattern** — Malicious image metadata → MCP Gateway RCE
- **MDM pre-auth RCE** — Ivanti EPMM CVE-2026-1281/1340 (both CVSS 9.8)
- **Burp Suite MCP + Kali Linux MCP connectors**

## v0.43.0

- **WeKnora 5-CVE multi-tenant compromise** — CVE-2026-30855 (CVSS 8.8)
- **RustDesk 7+ CVE cluster**
- **Trail of Bits Claude Code security skills**
- **Cisco Skill Scanner**

## v0.38.0

- **OWASP Agentic Top 10 test procedures** — 19 concrete test procedures (T1-T19)
- **Microsoft "AI as Tradecraft" case study**
- **Codex Security Research Preview stats**

## v0.37.0

- **Edge framework path normalization bypass** — CVE-2026-29045 (Hono)
- **SSRF via webhook/notification/import cluster** — 5-CVE cluster
- **HTTP/3 race conditions (QUICker)**
- **PleaseFix zero-click exfiltration**
- **6 new AI security tools**

## v0.28.0

- **TypeScript runtime type erasure pattern** — CVE-2026-25049 (n8n, CVSS 9.8)
- **11 new IDEsaster CVEs**
- **Claude DXT zero-click RCE** — CVSS 10.0
- **MCP OAuth expansion** — 7 new test procedures
- **VoidLink AI malware**

## v0.27.0

- **source-code-audit progressive disclosure** — Split to `reference/framework-patterns.md`
- **WeKnora CVE patterns** — MCP stdio blacklist bypass, PostgreSQL array SQLi bypass
- **Hacktron AI BeyondTrust RCE** — CVE-2026-1731 (CVSS 9.9)

For earlier versions, see git log.
