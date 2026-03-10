---
name: methodology
argument-hint: "[target] [focus-area]"
description: Generate a testing methodology checklist tailored to a target's tech stack. Combines recon data, vulnerability patterns, and program scope into a prioritized, step-by-step testing plan you can follow during a hunt session.
arguments:
  - name: target
    description: The target domain or application (e.g., example.com, "Shopify admin panel")
    required: true
  - name: focus
    description: Optional focus area (e.g., "API", "auth", "payments", "GraphQL")
    required: false
---

Generate a customized testing methodology for **$ARGUMENTS**.

## What This Command Does

Produces a structured, step-by-step testing checklist tailored to the target's technology stack, attack surface, and the hunter's focus area. Unlike generic OWASP checklists, this methodology is specific — it references actual endpoints, technologies, and vulnerability patterns relevant to the target.

## Execution Flow

```
Step 0: What changed? — Check for recent target changes (new features, patches, scope, API versions)
Step 1: Gather context — recon data, tech stack, program scope (use existing data or run quick recon)
Step 2: Identify applicable vulnerability classes based on tech stack
Step 3: Prioritize by (reward potential × likelihood) ÷ competition
Step 4: Generate concrete test cases for each vulnerability class
Step 5: Organize into a phased testing plan
Step 6: Add tool recommendations for each phase
```

> **Step 0 is mandatory.** Fresh changes have the lowest duplicate risk. Before generating a testing checklist, always check: program changelogs, release notes, blog posts, git activity, Wayback Machine diffs. If something changed recently, prioritize testing the changed surface first.

## Output Format

```markdown
# Testing Methodology: [Target]

**Generated:** [Date]
**Tech Stack:** [Detected technologies]
**Focus:** [Focus area if specified, otherwise "Full scope"]
**Estimated Time:** [Total time estimate]

---

## Phase 1: Reconnaissance & Mapping (Time: X hours)

### 1.1 Application Mapping
- [ ] Map all endpoints and functionality
- [ ] Identify API endpoints (REST, GraphQL, WebSocket)
- [ ] Document authentication flows
- [ ] Note file upload points
- [ ] Identify payment/transaction flows
- [ ] Map user roles and permission levels

### 1.2 Technology Fingerprinting
- [ ] Confirm server technology and version
- [ ] Identify frontend framework
- [ ] Check for CDN/WAF and note bypass techniques
- [ ] Review JavaScript for API keys, endpoints, secrets
- [ ] Check for source maps (.map files)

**Tools:** [Relevant tools for this phase]

---

## Phase 2: Authentication & Session Testing (Time: X hours)

[Concrete test cases based on the auth mechanisms detected]

- [ ] Test: [Specific test with target-relevant details]
- [ ] Test: [...]

**Tools:** [...]

---

## Phase 3: Authorization & Access Control (Time: X hours)

[IDOR, privilege escalation, and access control tests specific to the target's architecture]

- [ ] Test: [...]

**Tools:** [...]

---

## Phase 4: Injection & Input Handling (Time: X hours)

[Injection tests relevant to the detected tech stack — SQLi for SQL backends, SSTI for template engines, etc.]

- [ ] Test: [...]

**Tools:** [...]

---

## Phase 5: Business Logic (Time: X hours)

[Logic tests specific to the target's domain — payments, workflows, rate limits, etc.]

- [ ] Test: [...]

**Tools:** [...]

---

## Phase 6: Advanced / Stack-Specific (Time: X hours)

[Tests specific to the tech stack — cloud misconfigs, serverless issues, GraphQL, etc.]

- [ ] Test: [...]

**Tools:** [...]

---

## Quick Wins to Try First

[3–5 tests most likely to yield results based on the target profile]

1. [Specific quick win with reasoning]
2. [...]
3. [...]

## Things to Avoid

[Known N/A patterns, out-of-scope areas, testing restrictions]

## Tool Setup Checklist

- [ ] [Tool 1] configured for [target-specific setup]
- [ ] [Tool 2] configured for [...]
```

## Customization Rules

**By tech stack:**
- Node.js/Express → Prioritize prototype pollution, SSRF, SSTI (if template engine detected)
- PHP/Laravel → Prioritize deserialization, file upload, SQL injection
- Python/Django → Prioritize SSTI, ORM injection, debug mode exposure
- Java/Spring → Prioritize SSRF, deserialization, SpEL injection, actuator exposure
- Ruby/Rails → Prioritize mass assignment, IDOR, deserialization
- GraphQL → Prioritize introspection, batching, nested query DoS, authorization bypass
- REST API → Prioritize IDOR, BOLA, mass assignment, rate limiting
- SPA (React/Angular/Vue) → Prioritize DOM XSS, API authorization, client-side routing bypass

**By focus area:**
- "API" → Heavy authorization testing, IDOR, rate limiting, API versioning
- "auth" → Session management, OAuth, JWT, 2FA bypass, password reset
- "payments" → Race conditions, price manipulation, currency confusion, coupon abuse
- "file upload" → Extension bypass, content-type manipulation, path traversal, polyglots
- "cloud" → Storage misconfigs, SSRF to metadata, IAM issues (invoke cloud-security skill)
- "mobile" → API key extraction, certificate pinning bypass, deep link abuse

**By time budget:**
- < 2 hours → Quick wins only (Phase 1 light recon + top 5 tests)
- 2–8 hours → Phases 1–3 + quick wins
- 8–20 hours → Phases 1–5
- 20+ hours → Full methodology

## Related Skills & Commands

- `target-recon` — Provides tech stack and attack surface data
- `vuln-patterns` — Detailed test payloads for each vulnerability class
- `api-security` — API architecture-specific testing (REST, GraphQL, WebSocket, gRPC)
- `cloud-security` — Cloud-specific testing methodology
- `http-desync` — Protocol-level test patterns
- `/hunt-plan` — Broader hunt planning including program research and time budgeting
- `/scope-check` — Verify targets are in scope before testing
