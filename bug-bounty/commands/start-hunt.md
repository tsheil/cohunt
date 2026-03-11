---
name: start-hunt
argument-hint: "[target or situation]"
description: Get started hunting — tell me your target, phase, and time budget, and I'll route you to the right workflow
arguments:
  - name: context
    description: "Your target, current phase, or situation (e.g., 'new to Shopify', 'have SSRF, need to escalate')"
    required: false
---

# /start-hunt

New to the plugin or starting a fresh session? Tell me what you're working on and I'll route you to the right skill, command, or agent.

## Usage

```
/start-hunt <target, situation, or describe what you need>
```

Starting hunt: $ARGUMENTS

---

## If No Arguments Provided

I need four things to route you effectively:

1. **What changed recently?** — New features, patches, scope additions, API versions, acquisitions. Fresh changes = highest-signal attack surface. If nothing changed, that's fine — but always ask first.
2. **What's your target?** — Domain, app, program name, or "I don't have one yet"
3. **What phase are you in?** — Recon, testing, stuck, have findings, writing report
4. **How much time do you have?** — 30 minutes, 2 hours, full day, ongoing

Example: "I want to hunt on Shopify's API. I'm in the testing phase. I have about 3 hours."

---

## Routing Matrix

Based on your situation, I'll send you to the best workflow:

| Situation | Route To | Why |
|-----------|----------|-----|
| **Target recently changed** (new feature, patch, scope addition, API version) | `/regression-hunt` | Fresh changes = highest-signal attack surface |
| **Know a disclosed bug, want more findings** | `/variant-hunt` | Turn one bug into siblings — low duplicate risk |
| **Want to map business logic** | `/workflow-map` | Actors, states, invariants, abuse cases — 45% of payouts |
| **Found something, is it worth reporting?** | `/reportability-check` | Early go/no-go before investing in a full write-up |
| New target, no recon yet | `/recon` then `/hunt-plan` | Map the attack surface before testing |
| New target, want full guided session | `hunt-session` agent | End-to-end orchestrator handles everything |
| Have recon, need a test plan | `/hunt-plan` | Turns recon into prioritized test cases |
| Know the endpoint, want to test one vuln | `/quick-test` | Fastest path — exact requests to send |
| Need a testing checklist for a tech stack | `/methodology` | Structured methodology tailored to the target |
| Have findings, need to prioritize | `/triage-findings` | Rank by severity, reportability, duplicate risk |
| Have multiple findings that chain together | `/chain` | Score and document the chain as one report |
| Ready to write a report | `/write-report` | Submission-ready report for the platform |
| Report written, need quality check | `report-review` agent | Pre-submission QA with AI slop detection |
| Hunting an AI/LLM target | `ai-hunting` skill | LLM prompt injection, MCP vulns, agent attacks |
| Testing a mobile app | `mobile-security` skill | iOS/Android-specific testing patterns |
| Testing APIs (REST, GraphQL, gRPC) | `api-security` skill | API-specific vulnerability patterns |
| Testing auth or session management | `auth-testing` skill | OAuth, JWT, SSO, session fixation patterns |
| Cloud infrastructure target | `cloud-security` skill | AWS/GCP/Azure misconfiguration patterns |
| HTTP smuggling or cache poisoning | `http-desync` skill | Desync, request splitting, race conditions |
| Source code available for review | `source-code-audit` skill | Static analysis and variant hunting |
| Want to compare programs before choosing | `/compare-programs` | Side-by-side program analysis |
| Need to check if something is in scope | `/scope-check` | Scope boundary validation |
| Want to inventory assets for a target | `/asset-inventory` | Structured asset enumeration |
| Stuck or blocked mid-hunt | `/pivot` | Next 3 highest-value test angles |
| Need context-aware payloads | `/generate-payload` | WAF-aware, encoded, ready to paste |

---

## Quick Start Paths

### "I have 30 minutes"
Best bet: `/quick-test <vuln type> <target>` on a specific endpoint you already know. One focused test, one potential finding.

### "I have 2-4 hours"
Start with `/recon <target>` (includes authenticated recon + fixture setup), then `/hunt-plan` to prioritize, then `/quick-test` on the top 2-3 targets. **Set up 2+ accounts before testing** — most payable bugs require two-account proof. Use `/triage-findings` at the end to decide what to report.

### "I have a full day"
Use the `hunt-session` agent for an end-to-end guided session. It handles recon, prioritization, testing, and report preparation across a full hunting session.

### "I found something and need to report it"
Go straight to `/write-report <describe your finding>`. If you have multiple findings, run `/triage-findings` first to decide report order. If findings chain together, use `/chain` before writing.

### "I don't have a target yet"
Run `/compare-programs` to evaluate programs side by side, or ask me about a specific platform (HackerOne, Bugcrowd, Intigriti) and I'll help you pick a program based on your skills and payout goals.

### "I'm reviewing a report before submission"
Use the `report-review` agent. It catches common issues: missing reproduction steps, incorrect CVSS, AI-generated slop, weak impact statements, and platform-specific formatting problems.

---

## Phase Detection

If you describe your situation, I'll detect your phase automatically:

| Keywords in Your Input | Detected Phase | Route |
|------------------------|---------------|-------|
| "changed", "updated", "new feature", "patched", "released", "added scope" | Change-driven | `/regression-hunt` |
| "CVE", "disclosed", "variant", "similar bug", "same pattern" | Variant hunting | `/variant-hunt` |
| "business logic", "workflow", "payment", "checkout", "state machine" | Workflow mapping | `/workflow-map` |
| "is this reportable", "worth reporting", "N/A risk", "should I submit" | Reportability check | `/reportability-check` |
| "new", "first time", "getting started" | Onboarding | `/recon` + `/hunt-plan` |
| "scanning", "enumerating", "fingerprinting" | Recon | `target-recon` skill |
| "testing", "trying", "fuzzing", "injecting" | Active testing | `/quick-test` or `/methodology` |
| "found", "discovered", "confirmed" | Have findings | `/triage-findings` |
| "stuck", "blocked", "failing", "WAF" | Blocked | `/pivot` |
| "chain", "combine", "escalate" | Chaining | `/chain` |
| "report", "write up", "submit" | Reporting | `/write-report` |
| "review", "check", "before I submit" | QA | `report-review` agent |

---

## Flexible Input

You don't need to match the routing table exactly. Just describe your situation naturally:

- "I want to hunt on Shopify" — I'll start with program research and recon
- "I have a mobile app APK to test" — I'll route to mobile-security patterns
- "I found an XSS but CSP blocks it" — I'll help you bypass or chain it
- "I have 3 findings and need to report" — I'll triage and prioritize them
- "I'm stuck on this WAF" — I'll suggest pivot angles and bypass strategies
- "Is this IDOR reportable?" — I'll assess reportability and help write it up
- "I have source code access" — I'll route to source-code-audit for static analysis

---

## Tips

1. **Be specific about your target** — "example.com" routes better than "a website"
2. **Mention your skills** — "I'm good at API bugs" helps me prioritize relevant tests
3. **State your time budget** — forces realistic scoping and prevents rabbit holes
4. **Say what you've already tried** — avoids repeating work you've already done
5. **Name the program** — program intelligence shapes test prioritization and payout estimates
6. **Come back anytime** — run `/start-hunt` again mid-session if your situation changes
