---
name: source-code-audit
description: Audit source code for security vulnerabilities — trace data flows, find auth gaps, spot injection sinks, and identify logic flaws. Works on any codebase you can share or point to. Trigger with "audit this code", "review this repo for vulns", "find vulnerabilities in this source", "code review for security", "check this code for bugs", "security review", "static analysis", "taint analysis", "dangerous function", "unsafe deserialization". Use proactively when user shares code snippets, opens a repository, or mentions reviewing source for a target.
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

**Dangerous sinks:**

| Sink Type | Examples | Vulnerability |
|-----------|----------|--------------|
| SQL query | Raw SQL, string concatenation in queries | SQL injection |
| Command execution | `exec()`, `system()`, backticks, `child_process` | Command injection |
| File operations | `open()`, `readFile()`, path construction | Path traversal, LFI |
| Template rendering | `render()` with user data, `v-html`, `dangerouslySetInnerHTML` | XSS, SSTI |
| Deserialization | `pickle.loads()`, `unserialize()`, `JSON.parse()` with reviver | RCE, object injection |
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

### Step 8: Synthesize Findings

```
1. Rank findings by severity and exploitability
2. Map findings to CWE identifiers
3. Provide concrete exploitation paths
4. Write fix recommendations with code examples
5. Identify systemic patterns (recurring anti-patterns)
6. Recommend hardening priorities
```

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

## Reference Files

This skill uses progressive disclosure. Detailed reference material is available on demand:

| File | Contents | Lines |
|------|----------|-------|
| [reference/framework-patterns.md](reference/framework-patterns.md) | Language-specific anti-patterns (10 languages/frameworks), AI/LLM integration audit patterns, MCP server audit patterns | ~246 |

**Quick search** — find language/framework-specific patterns:
```
grep -n "Node\|Express\|JavaScript\|TypeScript" reference/framework-patterns.md
grep -n "Python\|Django\|Flask" reference/framework-patterns.md
grep -n "PHP\|Laravel\|Java\|Spring\|Go\|Rust" reference/framework-patterns.md
grep -n "AI/LLM\|prompt\|injection\|MCP\|tool_call" reference/framework-patterns.md
grep -n "type erasure\|Content-Type\|deserialization" reference/framework-patterns.md
```

---

## Pairing with Other Skills

- **target-recon** — Recon the running instance, then audit the source to confirm findings
- **vuln-patterns** — Get testing patterns for vulnerability classes found in code review
- **report-writing** — Write up code-level findings with source references and fix suggestions
- **program-research** — Check if source code auditing is rewarded by the program
- **ai-hunting** — For AI/LLM-specific vulnerability patterns beyond code review (prompt injection testing, MCP test procedures)
