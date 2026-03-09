# Shared Output Formats

Standardized output schemas used across commands and skills. When producing output, use these formats to enable pipelining between commands.

---

## Asset Table

Used by: `/recon`, `/asset-inventory`, `target-recon` skill

```markdown
| Asset | Type | Tech Stack | Status | Notes |
|-------|------|-----------|--------|-------|
| api.example.com | API | Node/Express | Live | Auth required |
| admin.example.com | Web App | React + Django | Live (403) | Admin panel |
| cdn.example.com | CDN | Cloudflare | Proxied | Out of scope |
```

## Endpoint Table

Used by: `/recon`, `/asset-inventory`, `target-recon` skill → consumed by `/hunt-plan`, `/methodology`

```markdown
| URL | Method | Auth | Parameters | Tech | Notes |
|-----|--------|------|-----------|------|-------|
| /api/v2/users/{id} | GET | Bearer | id (int) | REST | IDOR candidate |
| /api/v2/files/upload | POST | Bearer | file (multipart) | REST | File upload |
| /graphql | POST | Cookie | query (string) | GraphQL | Introspection enabled |
```

## Finding Record

Used by: `/session-notes` (structured format) → consumed by `/write-report`, `/triage-findings`, `/chain`

```markdown
### Finding: [Short Name]
- **Vulnerability:** [Type]
- **URL:** [Exact endpoint]
- **Parameter:** [Affected parameter]
- **Payload:** [Exact payload]
- **Reproduction:** [Numbered steps]
- **Impact:** [Attacker capability]
- **Severity Estimate:** [Critical/High/Medium/Low]
- **Evidence:** [Response snippet or screenshot ref]
- **Status:** [Ready to report / Needs more work / Needs chain]
```

## Hunt Target Row

Used by: `/hunt-plan`, `hunt-session` agent → consumed by `/session-notes`, `/pivot`

```markdown
| Priority | Target | Vuln Type | Reward | Likelihood | Competition | Rationale |
|----------|--------|-----------|--------|------------|-------------|-----------|
| 1 | /api/users/{id} | IDOR | $2k-$5k | High | Low | Sequential IDs, no authz check on GET |
```

## Scope Boundary

Used by: `/scope-check`, `hunt-session` agent → consumed by `/hunt-plan`

```markdown
**In Scope:**
- [Asset] — [type] — [why interesting]

**Out of Scope:**
- [Asset or vuln type]

**Gray Areas:**
- [Ambiguous item] — [risk assessment]
```
