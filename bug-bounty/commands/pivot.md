---
name: pivot
description: Stuck or blocked? Get the next 3 highest-value test angles based on your current session
arguments:
  - name: context
    description: "What failed or where you're stuck (e.g., 'XSS blocked by CSP', 'IDOR requires UUID')"
    required: false
---

# /pivot

Mid-hunt pivot advisor. When a test path hits a wall, describe what failed and I'll give you the next 3 highest-value angles to try.

## Usage

```
/pivot <what failed, what's blocking you, or what you've tried>
```

Pivoting from: $ARGUMENTS

---

## How It Works

```
┌──────────────────────────────────────────────────────────────┐
│                         PIVOT                                 │
├──────────────────────────────────────────────────────────────┤
│  1. Read session context — what's been tested, what failed   │
│  2. Analyze blockers — WAF, scope, dead ends, duplicates     │
│  3. Score untested angles by value and feasibility            │
│  4. Output top 3 pivot recommendations with first steps      │
└──────────────────────────────────────────────────────────────┘
```

---

## What I Need From You

**Minimum:** What you tried and what happened (e.g., "XSS payloads all get blocked by Cloudflare").

**Better:** Include the target, what vuln classes you've tested, and what responses you're seeing.

**Best:** Share the endpoint, payload attempts, response codes/bodies, and any error messages.

**Example inputs:**

```
/pivot Tried IDOR on /api/users/{id} but every request returns 403 regardless of auth token
```

```
/pivot Spent 2 hours on XSS in the search bar, CSP blocks everything. WAF is Akamai.
```

```
/pivot Found a low-severity open redirect but can't escalate it. Target is example.com.
```

---

## Output: Top 3 Pivot Angles

For each recommendation I'll provide:

```markdown
### Pivot 1: [Name]

**Target:** [Endpoint or feature to test]
**Vuln class:** [What to test for]
**Why this is promising:** [Reasoning based on what failed and what's likely untested]
**First step:** [Exact action to take — a curl command, a parameter to test, or a feature to explore]
**Time estimate:** [How long this angle should take before deciding to move on]
```

Each pivot is ranked by: `(Payout potential × Success likelihood) ÷ Time investment`

---

## Pivot Triggers

Recognize when it's time to stop and pivot. If any of these are true, you're spending time on diminishing returns:

| Trigger | Signal | Action |
|---------|--------|--------|
| **Repeated WAF blocks** | 3+ payloads blocked with identical responses | Switch vuln class or target different endpoint |
| **Consistent 403/401** | Auth enforcement is solid on this endpoint | Move to a different endpoint or feature |
| **No reflection or injection point** | Input is sanitized, encoded, or ignored | Try a different parameter, header, or input vector |
| **Likely duplicate** | Common vuln on well-known endpoint, program has many disclosed reports | Shift to less-tested features or novel attack chains |
| **Scope gray area** | Unclear if the target or vuln type is in scope | Confirm scope with `/scope-check` before investing more time |
| **Diminishing returns** | 30+ minutes on one endpoint with no progress | Hard pivot to a completely different area of the target |
| **Low-severity dead end** | Confirmed finding but impact too low to report | Try to chain it (use `/chain`) or shelve and move on |

---

## Lateral Movement Ideas

When a direct approach fails, move laterally instead of giving up entirely:

### Same Vulnerability, Different Endpoint
- Test sibling endpoints: `/api/v1/users` blocked? Try `/api/v2/users`, `/api/internal/users`, `/graphql`
- Test non-obvious entry points: file upload, import/export, webhook URLs, PDF generators
- Test mobile API endpoints: often have weaker protections than web counterparts
- Test older API versions: legacy endpoints may lack current security controls

### Different Vulnerability, Same Endpoint
- XSS blocked? Try SSTI, CRLF injection, or parameter pollution on the same input
- SQLi filtered? Try NoSQL injection, LDAP injection, or ORM-specific bypasses
- SSRF blocked on URL? Try redirect chains, DNS rebinding, or IPv6 representations
- Auth solid on read? Test write operations (PUT/PATCH/DELETE) which may have different controls

### Adjacent Feature Testing
- If the user profile is locked down, test: settings, notifications, integrations, export
- If the main app is hardened, test: admin panels, staging environments, documentation sites
- If the API is protected, test: WebSocket endpoints, RSS feeds, public embed endpoints
- If the web app is solid, test: mobile app, browser extension, desktop client

### Escalation From What You Have
- Low-severity open redirect? Chain with OAuth flow for token theft
- Self-XSS only? Chain with CSRF to deliver it to other users
- Information disclosure? Use leaked data to target specific users or endpoints
- Rate limiting bypass? Combine with brute force on auth endpoints

---

## Session Context Analysis

When you describe your situation, I analyze:

1. **Coverage gaps** — What vuln classes haven't been tested yet on this target?
2. **Attack surface gaps** — What endpoints, features, or input vectors are untouched?
3. **Skill alignment** — What angles match your strengths and available tools?
4. **Program intelligence** — What vuln types does this program typically reward? What gets N/A'd?
5. **Time efficiency** — Given your remaining time, what has the best ROI?

---

## Anti-Patterns to Avoid

- **Tunnel vision** — Testing the same vuln class for hours on a hardened endpoint
- **Spray and pray** — Running automated scans without understanding what you're targeting
- **Scope creep** — Drifting into out-of-scope assets when frustrated
- **Sunk cost** — "I've spent too long on this to stop" is the worst reason to continue
- **Copy-paste payloads** — Generic payloads without adapting to the target's specific stack and filters

---

## Tips

1. **Time-box every angle** — Give each pivot 20-30 minutes maximum before evaluating progress
2. **Log what you tried** — Use `/session-notes` so pivots build on prior work instead of repeating it
3. **Check your assumptions** — A "blocked" endpoint might just need a different content type or HTTP method
4. **Fresh eyes help** — If stuck for over an hour, take a break or switch to a completely different target
5. **Low findings can chain** — Before abandoning a low-severity finding, check if `/chain` can escalate it
