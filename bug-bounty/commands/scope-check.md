---
description: Quick scope check — verify if a target, vulnerability type, or finding is in scope for a program before you invest time testing or reporting
argument-hint: "<target or finding> for <program>"
---

# /scope-check

Quickly verify whether a target, vulnerability type, or specific finding is in scope before you spend time testing or writing a report.

## Usage

```
/scope-check <target or finding> for <program>
```

Check scope for: $ARGUMENTS

---

## How It Works

```
┌──────────────────────────────────────────────────────────────┐
│                       SCOPE CHECK                            │
├──────────────────────────────────────────────────────────────┤
│  ✓ Web search for program scope page                         │
│  ✓ Match target against in-scope assets                      │
│  ✓ Check vulnerability type against program policy           │
│  ✓ Flag out-of-scope exclusions and restrictions             │
│  ✓ Identify gray areas and recommend caution                 │
└──────────────────────────────────────────────────────────────┘
```

---

## What I Check

1. **Asset scope** — Is the domain/app/API listed as in-scope?
2. **Wildcard matching** — Does `*.example.com` cover `sub.example.com`?
3. **Vulnerability exclusions** — Does the program exclude this vuln type (e.g., DoS, social engineering)?
4. **Testing restrictions** — Any rules about automated scanning, rate limiting, or specific testing methods?
5. **Reward eligibility** — Is this asset in a rewarded tier or VDP-only?
6. **Recent changes** — Has the scope been updated recently?

---

## Output

```markdown
## Scope Check: [target/finding] on [program]

### Verdict: [IN SCOPE / OUT OF SCOPE / GRAY AREA]

**Asset:** [target]
**Program:** [program name]
**Platform:** [HackerOne / Bugcrowd / etc.]

### Details

| Check | Result | Notes |
|-------|--------|-------|
| **Asset listed** | [Yes/No/Partial] | [Exact match or wildcard] |
| **Vuln type allowed** | [Yes/No/Unclear] | [Any exclusions that apply] |
| **Reward tier** | [Critical/Standard/VDP-only/None] | [Bounty range if known] |
| **Testing restrictions** | [None/Some] | [What to watch for] |

### Recommendation

[What to do — proceed, skip, or proceed with caution and why.]
```

---

## Execution Flow

### Step 1: Parse Request

```
Identify what to check:
- "staging.example.com for Shopify" → Asset scope check
- "self-XSS for GitHub" → Vulnerability type check
- "api.target.io/v2/admin for Uber" → Specific endpoint check
- "DoS via regex for Cloudflare" → Testing method check
```

### Step 2: Find Program Scope (Always)

```
Run these searches:
1. "[Program] bug bounty scope" → Official scope page
2. "[Program] hackerone OR bugcrowd policy" → Platform page
3. "[Program] vulnerability disclosure policy" → VDP details
4. site:hackerone.com/[program] → Direct platform link
```

**Extract:**
- In-scope assets (domains, wildcards, apps, APIs)
- Out-of-scope assets and exclusions
- Vulnerability type exclusions
- Testing restrictions and rules of engagement
- Reward tiers per asset

### Step 3: Match Target Against Scope

```
For asset checks:
1. Exact match: Is the domain listed explicitly?
2. Wildcard match: Does *.example.com cover this subdomain?
3. Subdomain depth: Does the wildcard cover sub.sub.example.com?
4. IP vs domain: Is the IP in scope even if the domain is?
5. Third-party: Is this a third-party service (e.g., Cloudflare, AWS)?

For vulnerability type checks:
1. Explicit exclusion: Is this vuln type listed as out of scope?
2. Implicit exclusion: "No DoS" covers regex DoS too
3. Platform defaults: Standard exclusions (phishing, social engineering)
4. Self-only impact: Is self-XSS / self-CSRF excluded?
```

### Step 4: Deliver Verdict

```
Classify as:
- IN SCOPE: Clear match, go test/report
- OUT OF SCOPE: Explicit exclusion, don't waste time
- GRAY AREA: Ambiguous — recommend checking with program

Include actionable recommendation.
```

---

## Common Scope Traps

| Trap | Why it catches hunters |
|------|----------------------|
| **Wildcard doesn't cover subdomains of subdomains** | `*.example.com` might not include `a.b.example.com` |
| **Acquisitions not yet in scope** | Company bought target.io but scope still says example.com only |
| **Third-party infrastructure** | S3 bucket is AWS-owned, not the program's |
| **VDP vs bounty** | Asset is in scope for disclosure but has $0 reward |
| **Testing restrictions** | Domain is in scope but "no automated scanning" applies |
| **Self-impact only** | Self-XSS, self-CSRF typically excluded even if not stated |
| **Staging/dev environments** | Sometimes explicitly in scope, sometimes forbidden |
| **Mobile app but not API** | Mobile app in scope, but its backend API might not be |

---

## Examples

```
/scope-check staging.example.com for Shopify
/scope-check self-XSS for GitHub
/scope-check api.target.io/v2/admin for Uber
/scope-check DoS via regex for Cloudflare
/scope-check IDOR on mobile API for Tesla
```

---

## Tips

1. **Check before testing** — saves time on out-of-scope targets
2. **Check before reporting** — avoids N/A for scope violations
3. **When in doubt, ask** — if I say "gray area," consider reaching out to the program before testing
4. **Chain with hunt-plan** — scope-check your top targets before building a full plan
5. **Re-check periodically** — programs update scope; what was out of scope last month might be in scope now
