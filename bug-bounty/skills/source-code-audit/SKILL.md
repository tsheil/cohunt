---
name: source-code-audit
description: Audits source code for security vulnerabilities — traces data flows, finds auth gaps, spots injection sinks, and identifies logic flaws. Works on any codebase you can share or point to. ALWAYS use this skill when user shares code, opens a repository, pastes a code snippet, or asks to review source code for security issues. Trigger with "audit this code", "review this repo for vulns", "find vulnerabilities in this source", "code review for security", "check this code for bugs", "security review", "static analysis", "taint analysis", "dangerous function", "unsafe deserialization", "look at this code", "is this code secure", "what's wrong with this function", "dependency audit", "npm audit", "package vulnerability". Use proactively when user shares code snippets, opens a repository, or mentions reviewing source for a target — even if they don't explicitly say "security". For black-box testing without source, use vuln-patterns. For API-specific code patterns, use api-security. For CI/CD pipeline code review, use supply-chain-security.
---

# Source Code Audit

Find vulnerabilities by reading code — trace data flows from user input to dangerous sinks, spot authentication and authorization gaps, identify injection points, and flag logic flaws. Works on any code you can share or point to: files, repos, snippets, or diffs.

---

## Output Format

```markdown
# Source Code Audit: [Target]

**Generated:** [Date]
**Scope:** [Files/modules reviewed]
**Language:** [Primary language(s)]
**Framework:** [Detected framework(s)]

---

## Executive Summary

[2-3 sentences: overall security posture, most critical finding, recommended immediate action]

---

## Critical Findings

### [CRITICAL-1] [Vulnerability Type] in [location]

**CWE:** CWE-XXX
**File:** [path:line]
**Severity:** Critical
**Exploitability:** [Easy/Moderate/Hard]

**Vulnerable code:**
```[language]
[Relevant code snippet]
```

**Data flow:**
```
[user input source] → [processing] → [dangerous sink]
```

**Exploitation:** [How an attacker would exploit this]

**Fix:**
```[language]
[Corrected code]
```

---

## High Findings

### [HIGH-1] [Title]
[Same structure as critical]

---

## Medium Findings

### [MED-1] [Title]
[Condensed format]

---

## Low / Informational

- [Finding]: [Location] — [Brief description]

---

## Dependency Analysis

| Package | Version | Known CVEs | Severity | Notes |
|---------|---------|-----------|----------|-------|
| [pkg] | [ver] | [CVE-XXXX-XXXXX] | [severity] | [exploit available?] |

---

## Hardening Recommendations

1. [Most impactful improvement]
2. [Second priority]
3. [Third priority]

---

## Sources
- [Reference 1](URL)
```

---

## Execution Flow

### Step 1: Scope the Audit

```
Determine what to review:
- Single file / snippet → focused audit
- Directory / module → component-level audit
- Full repo → prioritize by attack surface
- PR / diff → change-focused review
- Dependency list → supply chain focus
```

**Prioritize by attack surface:**
1. Authentication and session handling
2. Input processing and validation
3. Database and data layer
4. File operations
5. External service integrations
6. Authorization and access control
7. Cryptography usage
8. Error handling and logging

### Step 2: Identify the Stack

```
Detect from code:
1. Language(s) and version indicators
2. Framework and framework version
3. Database (ORM, raw queries, NoSQL)
4. Template engine
5. Authentication library/strategy
6. Dependencies and their versions
```

### Step 3: Map Data Flows

```
For each user input source:
1. Identify entry points (routes, API endpoints, form handlers)
2. Trace input through processing (validation, sanitization, transformation)
3. Identify sinks (SQL queries, command execution, file operations, templates, responses)
4. Check for missing or insufficient sanitization at each step
5. Note any trust boundaries crossed
```

**Common sources:**
- HTTP parameters (query, body, headers, cookies)
- File uploads (name, content, metadata)
- WebSocket messages
- Environment variables from user-controlled sources
- Database values that originated from user input
- Webhook payloads and callback URLs
- Queue/worker job arguments (Sidekiq, Celery, BullMQ)
- Cron job inputs and email-embedded links/tokens
- GraphQL resolver arguments (including nested/batched)
- Object storage metadata and signed URL parameters
- Backup/export request parameters
- Cache keys derived from user input

**Dangerous sinks:**

| Sink Type | Examples | Vulnerability |
|-----------|----------|--------------|
| SQL query | Raw SQL, string concatenation in queries | SQL injection |
| Command execution | `exec()`, `system()`, backticks, `child_process` | Command injection |
| File operations | `open()`, `readFile()`, path construction | Path traversal, LFI |
| Template rendering | `render()` with user data, `v-html`, `dangerouslySetInnerHTML` | XSS, SSTI |
| Deserialization | `pickle.loads()`, `unserialize()`, `ObjectInputStream`, YAML parsers, `serialize-javascript` | RCE, object injection |
| URL construction | Redirects, SSRF-prone fetches, `href` attributes | Open redirect, SSRF |
| Crypto operations | Key generation, encryption, hashing, token creation | Weak crypto, token prediction |

### Step 4: Check Authentication & Authorization

```
Review:
1. Login flow — brute force protection, timing attacks, credential storage
2. Session management — token generation, storage, expiration, invalidation
3. Password handling — hashing algorithm, salt, reset flow
4. MFA implementation — bypass potential, backup codes
5. Authorization checks — per-endpoint, per-object, role validation
6. API authentication — key validation, JWT handling, OAuth flows
```

**Common auth anti-patterns:**

| Anti-Pattern | What to look for |
|-------------|-----------------|
| Client-side auth | Authorization decisions in JavaScript, not server |
| Missing object-level auth | Fetching by ID without ownership check |
| Role in JWT | Trusting role claim without server-side verification |
| Timing-safe comparison | Using `==` instead of constant-time compare for tokens |
| Password in logs | Logging request bodies that contain passwords |
| Session fixation | No session regeneration after login |
| Missing `await` on async auth | Un-awaited `bcrypt.compare()` returns Promise (truthy) → any password accepted (CVE-2026-28514 Rocket.Chat, CVSS 9.3; discovered by GitHub Taskflow Agent) |

### Step 4b: Map Business Invariants

Source/sink analysis catches injection. It misses business logic bugs — the #1 paying class. For each feature, define what *should never happen*, then search for code paths that violate it:

```
For each business feature:
1. Identify actors (unauth, low-priv, alt-tenant, guest, invited, suspended, support-impersonation)
2. Identify objects (orders, credits, invites, exports, billing records, API keys)
3. Define state transitions (pending→approved→fulfilled) and which actors can trigger each
4. Identify server-owned fields (price, role, tenant_id, plan_tier) — these must NEVER come from client
5. Check: does ANY code path allow an actor to skip a state, cross a tenant boundary, or mutate a server-owned field?
```

**What to grep for:**
- `tenant_id`, `org_id`, `team_id` set from request params instead of session/JWT
- State transitions without checking current state (`order.status = 'fulfilled'` without verifying `status == 'approved'`)
- Price/quantity/discount fields accepted from client without server-side recalculation
- Invite/share/export endpoints missing ownership validation on the target resource
- Approval workflows where the approver check uses a client-supplied role

### Step 5: Check for Framework-Specific Issues

Look for language/framework-specific anti-patterns. Common high-risk patterns across all stacks:

| Category | Examples | Risk |
|----------|----------|------|
| **Dynamic execution** | `eval()`, `exec()`, `Function()`, `system()` with user input | RCE |
| **Unsafe deserialization** | `pickle.loads()`, `unserialize()`, `ObjectInputStream.readObject()` | RCE |
| **Template injection** | `render_template_string()`, `render inline:`, `v-html` with user data | SSTI/XSS |
| **SQL via string concat** | `f"SELECT ... {user_input}"`, `$_GET` in raw queries | SQLi |
| **Path manipulation** | File paths from user input without validation or symlink resolution | Path traversal |
| **Prototype/mass assignment** | `Object.assign()`, `_.merge()`, `params.permit!` | Various |

> **Full framework-specific patterns (10 languages/frameworks + AI/LLM + MCP patterns):** See [reference/framework-patterns.md](reference/framework-patterns.md)

### Step 6: Review Dependencies

```
1. Parse dependency file (package.json, requirements.txt, Gemfile, pom.xml, go.mod)
2. Check for known CVEs in pinned versions
3. Flag outdated packages with security implications
4. Identify abandoned/unmaintained dependencies
5. Check for typosquat risk (similar names to popular packages)
```

### Step 7: Check Secrets and Configuration

```
Look for:
1. Hardcoded API keys, tokens, passwords
2. Default credentials in config files
3. Debug/development settings in production code
4. Sensitive data in error messages or logs
5. Secrets in version control (check .gitignore coverage)
6. Environment variable fallbacks with insecure defaults
```

**Common patterns:**
- `password = "admin123"` or similar hardcoded creds
- `SECRET_KEY = "changeme"` or low-entropy secrets
- `API_KEY = "sk-..."` committed to repo
- `.env` file not in `.gitignore`
- Sensitive values in `docker-compose.yml` or CI configs

### Step 7b: Mine Variant Surfaces

Don't just review the code in front of you — actively mine for hidden endpoints and partially-fixed siblings:

```
1. Git history mining — search for "fix", "security", "patch", "vuln", "authz", "IDOR" in commit messages;
   read the diffs to understand what was fixed, then grep sibling handlers for the same pattern unfixed
2. Sibling handler analysis — if POST /api/users/:id has an authz check, do PUT, DELETE, PATCH on the same
   route? Do /api/users/:id/export, /api/users/:id/avatar, /api/users/:id/settings?
3. Test/fixture extraction — test files reveal expected behavior and valid object states;
   fixtures contain valid IDs, tokens, and role configurations usable for dynamic validation
4. Frontend/API schema extraction — search JS bundles, OpenAPI/Swagger specs, and GraphQL introspection
   for endpoints not covered by auth middleware; diff middleware route coverage vs full route list
5. Middleware coverage diff — list all registered routes; list all routes protected by auth middleware;
   the difference is your attack surface
```

### Step 7c: Dynamic Validation (Code→Exploit Loop)

**Critical step.** A code finding is not a vulnerability until you prove it's exploitable at runtime. This is the methodology XBOW uses to merge static and dynamic testing — and it's how they found 1,060 vulnerabilities in 90 days.

For each finding from steps 3-7b:

```
1. IDENTIFY — Pinpoint the vulnerable code path (file:line, function, route)
2. CONSTRUCT — Build the exact HTTP request/WebSocket message/GraphQL query that reaches the vulnerable sink
   Include: method, path, headers, auth token, body with the specific parameter that triggers the bug
3. BASELINE — Send a normal (non-malicious) version of the request; capture the expected response
4. EXPLOIT — Send the crafted payload; compare response to baseline
   If blocked: read the code to understand WHY (WAF? validation? middleware?), then bypass
5. PROVE IMPACT — Demonstrate actual harm: data from another user returned, state changed without authz,
   command output in response, file contents leaked
6. GATE — Only keep findings that end in one of:
   ✓ Validated exploit with impact proof → report
   ✓ Exploitable but needs precondition you can document → report with precondition
   ✗ Theoretical only (can't reach sink at runtime, dead code, test-only path) → discard
```

**Identity matrix** — Test every validated finding against all identity levels:

| Identity | How to test |
|----------|------------|
| Unauthenticated | Strip all cookies/tokens |
| Low-privilege user | Regular user account token |
| Alt-tenant user | User from different org/tenant |
| Guest/invited user | Limited-access invitation token |
| Suspended/deactivated | Account marked inactive |
| Support-impersonation | Support role acting as user |

**Why this matters:** Many endpoints block cross-user access but allow anonymous access entirely (XBOW found this in Spree Commerce CVE-2026-22588/22589). Test every identity level — don't assume one failure means the endpoint is secure.

### Step 8: Synthesize and Gate Findings

```
1. Apply reportability gate — discard findings without runtime proof (Step 7c)
2. Rank validated findings by severity and exploitability
3. Map findings to CWE identifiers
4. Include the exact request/response pair that proves exploitation (from Step 7c)
5. Write fix recommendations with code examples
6. Identify systemic patterns — if one handler is missing authz, check ALL handlers in the same module
7. Recommend hardening priorities
```

**Reportability check:** Before including any finding, verify: "Could I submit this to a bug bounty program with the evidence I have right now?" If not, either go back to Step 7c or discard.

---

## Validation Gate — Is This Submittable?

Before including any finding in your report, pass it through these checks. Source-code-only findings without runtime proof are the #1 cause of N/A verdicts in code-audit-originated reports.

### Common False Positives

| Looks Like a Bug | Why It Usually Isn't |
|---|---|
| `eval()` / `exec()` with user input | Check if input is actually user-controlled at runtime — may be admin-only config, build-time constant, or test helper |
| SQL query with string concatenation | Verify the concatenated value reaches production — may be an ORM-generated query, parameterized at a higher layer, or test-only code |
| Hardcoded secret in source | Verify it's a real production secret, not a placeholder, test fixture, or revoked key — check git history for rotation |
| Missing CSRF protection on endpoint | Many SPAs use token-based auth (Bearer/JWT) which is inherently CSRF-proof — only report if session cookies are used |
| Missing rate limiting | Informational in most programs unless you can demonstrate account takeover, financial loss, or DoS |
| Unsafe deserialization call | Confirm untrusted input reaches the deserializer — internal-only RPC, admin tools, and test fixtures are not exploitable |
| Path traversal in file operation | Verify the path component comes from user input, not from a database lookup, config file, or hardcoded constant |
| Debug endpoint / admin panel | Only a finding if reachable in production — check middleware, deployment configs, and environment guards |
| Deprecated crypto (MD5/SHA1) | Only reportable if used for security-sensitive purposes (password hashing, token generation) — not checksums or cache keys |

### Proof Checklist (Every Finding Must Pass)

```
□ Runtime proof — You have a request/response pair showing exploitation, not just a code snippet
□ User-controlled input — The source of the vulnerability is actually attacker-controllable
□ Production-reachable — The code path runs in production, not just tests/dev/admin-only
□ Cross-boundary harm — Someone other than yourself is harmed (data leak, privilege escalation, state corruption)
□ Not already mitigated — No WAF, middleware, or upstream validation blocks the attack
□ Reproducible — Steps work on a clean setup, not dependent on stale state or race conditions
```

If any check fails, either go back to Step 7c to gather more evidence, or discard the finding. **A plausible code path is not a vulnerability — a proven exploit is.**

---

## Audit Variations

### Quick Review
Focus on: Dangerous sinks, obvious injection points, hardcoded secrets. Best for: snippets, single files, time-constrained reviews.

### Auth-Focused
Focus on: Authentication flows, session management, authorization checks, access control. Best for: login/signup code, middleware, API auth.

### Dependency Audit
Focus on: Package versions, known CVEs, supply chain risks. Best for: `package.json`, `requirements.txt`, lock files.

### PR / Diff Review
Focus on: Security implications of changes, newly introduced sinks, removed protections. Best for: code review in development workflow.

### Full Audit
Focus on: Everything — data flows, auth, dependencies, configuration, logic. Best for: new codebases, pre-launch reviews, bounty targets with source access.

### AI/LLM Integration Audit
Focus on: How the application integrates with LLM APIs and AI services. Best for: apps using OpenAI, Anthropic, Google AI APIs, or any LLM-backed features. Check for: prompt construction injection, output trust (LLM output → eval/exec/SQL/HTML), API key handling, multi-tenant isolation, tool call validation, RAG/memory poisoning.

> **Full AI/LLM integration checklist with code patterns:** See [reference/framework-patterns.md](reference/framework-patterns.md#aillm-integration-patterns)

### MCP Server Implementation Audit
Focus on: Security of MCP server code — input validation, authentication, command injection. Best for: reviewing custom MCP servers or auditing MCP server dependencies. Check for: eval()/exec() epidemic (67% of servers), path traversal (82% have file ops), missing auth (38% lack auth), tool poisoning, OAuth endpoint injection, symlink escape, command blacklist bypass (WeKnora `-p` flag pattern), PostgreSQL expression bypass.

> **Full MCP audit checklist with 13 patterns:** See [reference/framework-patterns.md](reference/framework-patterns.md#mcp-server-implementation-patterns)

### Repo-Artifact Security Audit
Focus on: AI tooling config files and CI/CD artifacts that create supply chain RCE or credential exfil vectors. Best for: repos that use AI coding tools, have CI/CD pipelines, or publish npm/PyPI packages.

**Config files to audit (these execute before trust is granted):**

| Artifact | Risk | What to Check |
|----------|------|---------------|
| `.claude/settings.json` | API key redirect (CVE-2026-21852) | `ANTHROPIC_BASE_URL` pointing to external domain |
| `.claude/settings.local.json` | Env var injection | Arbitrary env vars set in project config |
| `CLAUDE.md` | Instruction injection | Hidden instructions in project memory files |
| `.claude/hooks/` | Auto-RCE (CVE-2025-59536) | Shell commands that execute on session events |
| `.mcp.json` / `.mcp/` | Rogue MCP servers | Untrusted MCP servers auto-loaded on project open |
| `.cursor/mcp.json` | Same as above for Cursor | Rogue servers in Cursor-specific config |
| `.cursorrules` / `.cursor/rules/` | Rule file injection | Hidden prompt injection in cursor rules |
| `.github/copilot-instructions.md` | Copilot instruction poisoning | Malicious instructions for GitHub Copilot |
| `package.json` scripts | postinstall RCE | `preinstall`/`postinstall` scripts executing on `npm install` |
| `.github/workflows/` | Script injection | `${{ github.event.* }}` in `run:` blocks |
| `Dockerfile` / `docker-compose.yml` | Privilege escalation | `--privileged`, host mounts, exposed secrets |

**Testing procedure:**
1. Clone/fork target repo → scan for all files above
2. Check if any config file redirects API calls, adds rogue MCP servers, or runs shell commands
3. Test pre-trust exploitation: do any configs execute before the user explicitly grants trust?
4. Check `postinstall` scripts for download-and-execute patterns
5. Verify GitHub Actions workflows for injection via user-controlled event properties

---

## Tips for Better Audits

1. **Start with entry points** — routes and API endpoints are where user input enters the system
2. **Follow the data** — trace every user input from source to every sink it reaches
3. **Check the auth layer first** — the most impactful bugs are usually auth/authz failures
4. **Read the tests** — test files often reveal expected behavior that can be subverted
5. **Check the git history** — `git log` for "fix", "security", "patch", "vuln" reveals past issues
6. **Look at error handling** — error paths are frequently less hardened than happy paths
7. **Read the config** — default configs often have debug mode, weak secrets, or permissive CORS

---

## AI-Augmented Audit: Suggest-and-Audit Pattern

A two-stage methodology proven to find more business logic and auth bugs than single-pass review:

**Stage 1 — Suggest:** Read each component and generate potential vulnerability hypotheses. Cast a wide net — list every possible issue, including speculative ones. Focus on business logic, auth, and data flow.

**Stage 2 — Audit:** With fresh context, validate each hypothesis against the actual source code. Apply strict criteria: only confirm findings with clear evidence (source → sink trace, missing check, or exploitable pattern). Discard speculative findings without code evidence.

**Why it works:** GitHub Security Lab's Taskflow Agent evaluated 80+ vulnerability hypotheses across 40 repositories using this pattern — highest confirmed rate in business logic (25%) and 38 IDOR/access-control issues in the evaluation bucket (19 impactful results kept after manual review). Key AI-discovered CVEs:
- **CVE-2026-28514** (Rocket.Chat, CVSS 9.3): Missing `await` on `bcrypt.compare()` — suggest-stage flagged async auth pattern, audit-stage confirmed any-password bypass
- **CVE-2026-25758** (Spree Commerce): Sequential ID enumeration in address endpoints — unauthenticated address data exposure via ID increment
- **WooCommerce**: Authenticated users could view all guest orders including PII via predictable order identifiers

**When to use:** Any codebase audit where you want to maximize coverage. Especially effective for large repos where single-pass review misses subtle interaction bugs.

---

## Reference Files

This skill uses progressive disclosure. Detailed reference material is available on demand:

| File | Contents | Lines |
|------|----------|-------|
| [reference/framework-patterns.md](reference/framework-patterns.md) | Language-specific anti-patterns (10 languages/frameworks), AI/LLM integration audit patterns, MCP server audit patterns | ~288 |

**Quick search** — find language/framework-specific patterns:
```
grep -n "Node\|Express\|JavaScript\|TypeScript" ${CLAUDE_SKILL_DIR}/reference/framework-patterns.md
grep -n "Python\|Django\|Flask" ${CLAUDE_SKILL_DIR}/reference/framework-patterns.md
grep -n "PHP\|Laravel\|Java\|Spring\|Go\|Rust" ${CLAUDE_SKILL_DIR}/reference/framework-patterns.md
grep -n "AI/LLM\|prompt\|injection\|MCP\|tool_call" ${CLAUDE_SKILL_DIR}/reference/framework-patterns.md
grep -n "type erasure\|Content-Type\|deserialization" ${CLAUDE_SKILL_DIR}/reference/framework-patterns.md
```

---

## Pairing with Other Skills

- **target-recon** — Recon the running instance first, then audit source to explain and confirm findings
- **vuln-patterns** — Get testing patterns for vulnerability classes found in code review
- **auth-testing** — Deep-dive BOLA/BFLA testing after Step 4b identifies missing authz in code
- **business-logic** — State machine testing after code audit reveals workflow skip paths
- **report-writing** — Write up code-level findings with source references and fix suggestions
- **program-research** — Check if source code auditing is rewarded by the program
- **ai-hunting** — For AI/LLM-specific vulnerability patterns beyond code review (prompt injection testing, MCP test procedures)
