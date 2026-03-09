---
description: Track findings, observations, and progress during a bug bounty hunting session — keep running notes so nothing gets lost
argument-hint: "[add/view/export] [note content]"
---

# /session-notes

Track your hunting session in real time. Add findings, observations, interesting endpoints, and ideas as you go — then export a clean summary when you're done.

## Usage

```
/session-notes add Found IDOR on /api/users/{id} — needs more testing
/session-notes add Interesting endpoint: /api/internal/debug — returns stack traces
/session-notes add XSS attempt on /search blocked by CSP — try CSP bypass next
/session-notes view
/session-notes export
```

Session notes: $ARGUMENTS

---

## How It Works

```
┌──────────────────────────────────────────────────────────────┐
│                     SESSION NOTES                             │
├──────────────────────────────────────────────────────────────┤
│  TRACK (during your hunt)                                    │
│  ✓ Log findings as you discover them                         │
│  ✓ Note interesting endpoints and behaviors                  │
│  ✓ Record what you've tested and what's left                 │
│  ✓ Capture ideas for follow-up testing                       │
│  ✓ Track time spent per target area                          │
├──────────────────────────────────────────────────────────────┤
│  REVIEW (when you need context)                              │
│  ✓ View all notes from the current session                   │
│  ✓ See findings organized by category                        │
│  ✓ Review coverage — what you've tested vs. what's left      │
├──────────────────────────────────────────────────────────────┤
│  EXPORT (when you're done)                                   │
│  ✓ Generate a clean session summary                          │
│  ✓ Findings ready for triage or report writing               │
│  ✓ Coverage map showing tested areas                         │
│  ✓ Follow-up list for next session                           │
└──────────────────────────────────────────────────────────────┘
```

---

## Actions

### `add` — Log a Note

Add observations, findings, or ideas to the running session log.

**Input formats:**

```
/session-notes add [any text describing what you found or observed]
/session-notes add finding: IDOR on /api/users/{id}
/session-notes add tested: auth bypass on /login — rate limiting in place
/session-notes add idea: check if /api/v1/ has weaker auth than /api/v2/
/session-notes add endpoint: /api/internal/health returns server info
```

**Structured finding format (use when logging confirmed findings):**

When a note is a confirmed or likely finding, prompt for and use this structured format:

```markdown
### Finding: [Short Name]
- **Vulnerability:** [Type — e.g., IDOR, XSS, SSRF]
- **URL:** [Exact endpoint]
- **Parameter:** [Affected parameter or field]
- **Payload:** [Exact payload used]
- **Reproduction:** [Numbered steps from zero state]
- **Impact:** [What an attacker achieves]
- **Severity Estimate:** [Critical/High/Medium/Low]
- **Evidence:** [Response snippet, status code, or screenshot reference]
- **Status:** [Ready to report / Needs more work / Needs chain]
```

This structured format feeds directly into `/write-report` and `/triage-findings` — no information is lost in the handoff.

**Categorization:**

When adding notes, automatically categorize them:

| Category | Trigger keywords | Icon |
|----------|-----------------|------|
| **Finding** | found, IDOR, XSS, SSRF, vulnerability, bug, bypass | :red_circle: |
| **Tested** | tested, checked, tried, confirmed, no vuln, not vulnerable | :white_check_mark: |
| **Interesting** | interesting, unusual, worth, investigate, check | :mag: |
| **Endpoint** | endpoint, path, URL, api, route | :link: |
| **Idea** | idea, try, next, follow-up, maybe, could | :bulb: |
| **Blocked** | blocked, WAF, CSP, rate limit, filtered, 403 | :no_entry: |

### `view` — Review Session Notes

Display all notes from the current session, organized by category.

**Output format:**

```markdown
## Session Notes: [Target]

**Session started:** [Time]
**Notes logged:** [count]

---

### Findings (X)

1. [Time] IDOR on /api/users/{id} — can read other users' email
2. [Time] Open redirect on /login?next= parameter

### Tested — No Issues (X)

1. [Time] Auth bypass on /login — rate limiting blocks brute force
2. [Time] CSRF on /settings — token validation present

### Interesting — Investigate More (X)

1. [Time] /api/internal/debug returns stack traces with internal IPs
2. [Time] Different behavior on /api/v1/ vs /api/v2/ endpoints

### Endpoints Discovered (X)

1. [Time] /api/internal/health — server info disclosure
2. [Time] /api/admin/users — 403 but confirms admin API exists

### Ideas for Later (X)

1. [Time] Try CSP bypass with base-uri injection
2. [Time] Check if /api/v1/ has weaker auth than /api/v2/

### Blocked / Filtered (X)

1. [Time] XSS on /search blocked by CSP — unsafe-inline not allowed
2. [Time] SSRF attempt blocked by WAF on /preview endpoint

---

### Coverage Summary

**Tested areas:** [list]
**Untested areas:** [list if hunt plan available]
**Time spent:** [estimate]
```

### `export` — Generate Session Summary

Produce a clean, structured summary of the hunting session that's ready for follow-up.

**Output format:**

```markdown
# Hunt Session Summary: [Target]

**Date:** [Date]
**Duration:** [Estimated]
**Target:** [Domain/program]

---

## Key Findings

[Findings from notes, formatted for triage]

| # | Finding | Severity Estimate | Status |
|---|---------|-------------------|--------|
| 1 | [Finding] | [Estimated] | [Ready to report / Needs more work] |

## What Was Tested

[Coverage summary — what areas were tested and what wasn't]

## What to Do Next

### Report These
1. [Finding ready to report] → Use `/write-report` or `/triage-findings`

### Investigate Further
1. [Interesting observation that needs more testing]

### Next Session
1. [Areas not yet covered]
2. [Ideas to try next time]

---

## Raw Notes

[All session notes in chronological order]
```

---

## How Notes Are Stored

Notes are maintained in the conversation context during the session. When you `export`, a persistent summary is generated. This means:

- Notes persist as long as the conversation is active
- Use `export` before ending your session to save your work
- The export can be saved to a file for future reference

---

## Tips

1. **Log early, log often** — Quick notes during testing save you from forgetting what you tried
2. **Use `add finding:` prefix** — Makes findings easy to find later
3. **Note what DIDN'T work** — "Tested X, not vulnerable" prevents retesting the same thing
4. **Export before closing** — Don't lose session context
5. **Chain with triage** — After export, use `/triage-findings` on your findings list
6. **Start each session fresh** — Begin with `/session-notes add Starting hunt on [target]`

---

## Related Commands & Skills

- `/triage-findings` — Prioritize your findings after reviewing session notes
- `/write-report` — Turn a finding from your notes into a submission-ready report
- `/hunt-plan` — Create the testing plan that session notes track progress against
- `/methodology` — Get the testing checklist that session notes complement
