---
name: asset-inventory
description: Track and manage discovered assets during a hunting session — subdomains, endpoints, technologies, and findings in one place
arguments:
  - name: action
    description: "Action to perform: add, view, export, or clear"
    required: true
  - name: details
    description: "Asset details — type and value (e.g., 'subdomain api.example.com', 'endpoint POST /api/v2/users')"
    required: false
---

# Asset Inventory

Track discovered assets during a hunting session. Keeps a running inventory of subdomains, endpoints, technologies, credentials, and findings — organized and exportable.

## Actions

### `add` — Log an asset

Add a discovered asset to the inventory. Categorize automatically based on what's provided.

**Asset types:**
- **subdomain** — `add subdomain api.example.com` — discovered subdomains
- **endpoint** — `add endpoint POST /api/v2/users` — API routes and methods
- **technology** — `add technology nginx 1.21.4` — detected tech and versions
- **credential** — `add credential API key in JS bundle at /static/app.js` — leaked secrets (store description only, never the actual secret)
- **finding** — `add finding IDOR on /api/users/{id} — can read other users' profiles` — potential vulnerabilities
- **note** — `add note Rate limiting only on /login, not on /api/auth` — observations

When a user adds an asset, store it in a structured format:

```markdown
## Asset Inventory — {target}

### Subdomains
| # | Subdomain | Source | Status | Notes |
|---|-----------|--------|--------|-------|
| 1 | api.example.com | crt.sh | Active (200) | REST API |

### Endpoints
| # | Method | Path | Auth | Notes |
|---|--------|------|------|-------|
| 1 | POST | /api/v2/users | Bearer token | User creation |

### Technologies
| # | Technology | Version | Location | Notes |
|---|-----------|---------|----------|-------|
| 1 | nginx | 1.21.4 | api.example.com | Reverse proxy |

### Credentials & Secrets
| # | Type | Location | Notes |
|---|------|----------|-------|
| 1 | API key | /static/app.js | Google Maps key, unrestricted |

### Findings
| # | Severity | Type | Location | Status | Notes |
|---|----------|------|----------|--------|-------|
| 1 | High | IDOR | /api/users/{id} | Confirmed | Can read other users' profiles |

### Notes
| # | Note | Context |
|---|------|---------|
| 1 | Rate limiting only on /login | Auth endpoints |
```

### `view` — Show current inventory

Display the full asset inventory for the current session. Show counts by category and highlight high-priority items (confirmed findings, critical technologies, leaked credentials).

### `export` — Export inventory

Export the inventory as clean markdown, suitable for:
- Attaching to a hunt plan
- Sharing in session notes
- Feeding into `/write-report` or `/triage-findings`

### `clear` — Reset inventory

Clear all assets. Ask for confirmation before clearing.

## Workflow Integration

The asset inventory connects to other commands:

- **After `/recon`** — Automatically suggest adding discovered subdomains, technologies, and endpoints to inventory
- **Before `/hunt-plan`** — Feed inventory data into hunt plan for more targeted prioritization
- **During `/session-notes`** — Cross-reference session notes with tracked assets
- **Before `/triage-findings`** — Use inventory findings as input for triage
- **Before `/write-report`** — Pull confirmed findings from inventory as report candidates

## Tips

- Add assets as you discover them — don't wait until the end
- Use notes liberally — context you forget today costs time tomorrow
- Export before ending a session — inventory doesn't persist between conversations
- Flag severity on findings early — helps prioritize what to investigate next
