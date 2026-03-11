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
Step 1: State fixtures — Verify accounts, roles, and proof infrastructure are ready (MANDATORY)
Step 2: Gather context — recon data, tech stack, program scope (use existing data or run quick recon)
Step 3: Identify applicable vulnerability classes based on tech stack
Step 4: Prioritize by (reward potential × likelihood) ÷ (automation pressure × competition)
Step 5: Generate concrete test cases for each vulnerability class
Step 6: Organize into a phased testing plan
Step 7: Add tool recommendations for each phase
```

> **Step 0 is mandatory.** Fresh changes have the lowest duplicate risk. Before generating a testing checklist, always check: program changelogs, release notes, blog posts, git activity, Wayback Machine diffs. If something changed recently, prioritize testing the changed surface first.

> **Step 1 is mandatory.** Most payable bugs require two-account proof (IDOR, authz bypass, tenant isolation). Without state fixtures, you'll find bugs you can't prove. If any fixture is missing, the methodology output MUST flag it as a blocker.

## Output Format

```markdown
# Testing Methodology: [Target]

**Generated:** [Date]
**Tech Stack:** [Detected technologies]
**Focus:** [Focus area if specified, otherwise "Full scope"]
**Estimated Time:** [Total time estimate]

---

## Phase 0: State Fixtures & Proof Setup (MANDATORY — before any testing)

### Fixture Checklist
- [ ] **2+ accounts** at different privilege levels (free + paid, user + admin, tenant-A + tenant-B)
- [ ] **Role inventory** — every user role the app supports documented
- [ ] **OAST collector** — blind XSS/SSRF callback receiver (Burp Collaborator, interactsh, or self-hosted)
- [ ] **Disposable email** — for password reset, invite, and notification testing
- [ ] **Webhook receiver** — endpoint to capture outgoing webhook data
- [ ] **Pending invite** — outstanding invitation token ready for testing
- [ ] **Pending approval** — item in approval queue (if target has approval workflows)
- [ ] **Downgraded plan** — account recently downgraded from premium (if target has tiers)
- [ ] **API token pair** — tokens for both test accounts

> **BLOCKER:** If you cannot set up 2+ accounts at different privilege levels, you cannot prove most access control bugs. Flag this as a blocker and resolve before proceeding. Options: free trial, demo account, social engineering an invite, or contact the program for test credentials.

### Proof-Type Matrix
| Bug Class | Required Proof | Fixtures Needed |
|-----------|---------------|-----------------|
| IDOR / BOLA | Two-account: request with A's token returns B's data | 2 accounts |
| BFLA / Privilege escalation | Low-role performs high-role action | 2 accounts (different roles) |
| Tenant isolation | Tenant-A accesses Tenant-B data | 2 tenant accounts |
| Business logic (payments) | Before/after showing state change | 1+ account + payment flow |
| Race condition | Multiple concurrent requests showing double-spend | 1+ account + single-use resource |
| Blind XSS / SSRF | Callback from target to OAST receiver | OAST collector URL |
| Stored XSS | Payload stored by A triggers in B's browser | 2 accounts |

---

## Phase 1: Reconnaissance & Mapping (Time: X hours)

### 1.1 Application Mapping
- [ ] Map all endpoints and functionality
- [ ] Identify API endpoints (REST, GraphQL, WebSocket)
- [ ] Document authentication flows
- [ ] Note file upload points
- [ ] Identify payment/transaction flows
- [ ] Map user roles and permission levels
- [ ] Build role-endpoint matrix (which roles access which endpoints)

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
