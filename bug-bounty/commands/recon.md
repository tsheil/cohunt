---
name: recon
argument-hint: "[target-domain]"
description: Run reconnaissance on a target — external fingerprinting AND authenticated attack surface mapping. Produces fixtures, role-endpoint matrix, and first test cases
arguments:
  - name: target
    description: "Target domain to recon (e.g., 'example.com')"
    required: true
---

# /recon

Run full reconnaissance — external AND authenticated — on a web target. This command triggers the `target-recon` skill and produces a structured recon report with state fixtures and test cases ready for hunting.

**The money is behind login.** External recon finds the front door. Authenticated recon finds the bugs that pay. This command runs both.

## Usage

```
/recon <target domain>
```

Recon target: $ARGUMENTS

---

## What This Does

### Phase 1: External Recon (Quick — 5 min)

1. **HTTP fingerprinting** — headers, cookies, server software, error pages
2. **Technology detection** — framework, CMS, libraries, language
3. **Subdomain enumeration** — crt.sh, DNS lookups, status checks
4. **Security headers audit** — CSP, CORS, HSTS, X-Frame-Options
5. **Common paths** — robots.txt, sitemap.xml, .well-known/security.txt, .mcp.json, CLAUDE.md
6. **WAF/CDN detection** — provider identification, bypass potential

### Phase 2: Authenticated Recon (Where bugs pay — invest most time here)

7. **Account setup** — Identify signup flow, create 2+ accounts at different privilege levels
8. **Role inventory** — Enumerate every user role the app supports (free, paid, admin, support, API-only)
9. **Role-endpoint matrix** — Map which roles can access which endpoints. Every ✗ cell is a test target
10. **Tenant boundary mapping** — If multi-tenant: map org isolation, cross-tenant endpoints
11. **State fixtures** — Set up invite flows, pending approvals, downgraded plans, webhook receivers
12. **First 10 test cases** — Generated from the role-endpoint matrix, sorted by payout likelihood

**If you cannot create accounts:** Return a **BLOCKER** instead of silently stopping. The hunter needs to know that authenticated recon was not completed and most payable bugs are behind login. Suggest: free trial, demo account, social engineering an invite, or contact the program for test credentials.

Use the `target-recon` skill to execute this recon. Follow its full execution flow and output format.

---

## Output

Produce the structured recon report defined in the `target-recon` skill, including:
- Target profile (IP, hosting, CDN, WAF)
- Technology stack table
- Security headers assessment
- Subdomain inventory
- Interesting paths
- **State fixtures checklist** (accounts, roles, pending states, OAST receiver)
- **Role-endpoint matrix** (which roles access which endpoints)
- **Tenant boundary map** (if multi-tenant)
- **First 10 test cases** (sorted by payout likelihood)
- Attack surface summary with prioritized next steps

---

## Fixture Checklist

The recon output MUST include this checklist. Unfilled items are blockers for testing.

```
┌─ STATE FIXTURES ───────────────────────────────────────────────┐
│ □ ACCOUNTS: 2+ accounts at different privilege levels          │
│   (e.g., free + paid, user + admin, tenant-A + tenant-B)      │
│ □ ROLES: Every user role the app supports inventoried          │
│ □ OAST COLLECTOR: Blind XSS/SSRF callback receiver ready      │
│ □ EMAIL INBOX: Disposable inbox for password reset, invite     │
│ □ WEBHOOK RECEIVER: Endpoint to capture outgoing webhook data  │
│ □ PENDING INVITE: Outstanding invitation token                 │
│ □ PENDING APPROVAL: Item in approval queue                     │
│ □ DOWNGRADED PLAN: Account recently downgraded from premium    │
│ □ API TOKEN PAIR: Tokens for both accounts                     │
└────────────────────────────────────────────────────────────────┘
```

---

## Tips

- **Root domains cast a wider net** — "example.com" finds more than "www.example.com"
- **Chain with /hunt-plan** — recon first, then `/hunt-plan` to turn findings into action
- **Mention the program** — "for their HackerOne program" helps scope the work
- **Ask for depth** — add "deep" or "quick" to adjust thoroughness
- **Don't stop at external** — If you only have headers and subdomains, you haven't recon'd the app
