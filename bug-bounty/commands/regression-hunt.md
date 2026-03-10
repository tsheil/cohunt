---
name: regression-hunt
argument-hint: "[target or scope change]"
description: Hunt for vulnerabilities in recent changes — diff JS bundles, API specs, mobile releases, and new scope additions
arguments:
  - name: context
    description: "Target and what changed (e.g., 'Shopify added a new GraphQL API', 'Stripe released v3 of their SDK')"
    required: false
---

# /regression-hunt

Hunt where the code just changed. New features, API versions, scope additions, and recent patches are the highest-signal attack surface — they haven't been picked over by other hunters yet.

## Usage

```
/regression-hunt <target or recent change>
```

**Examples:**
- `/regression-hunt target.com just added OAuth login`
- `/regression-hunt Shopify — new GraphQL mutations appeared last week`
- `/regression-hunt program X added *.staging.example.com to scope`

## What I Do

1. **Identify what changed** — scope additions, new endpoints, API version bumps, mobile releases, JS bundle changes
2. **Diff the attack surface** — compare before/after to find new parameters, endpoints, auth changes, removed security controls
3. **Prioritize by exploitability** — new code has more bugs than mature code; focus on changes that touch auth, payments, data access
4. **Generate targeted test cases** — not generic checklists, but tests specific to what changed

## Change Detection Playbook

### JavaScript Bundle Diffing

```
1. Fetch current JS bundles — check /static/js/*.js, webpack chunk names, source maps
2. Compare against cached/archived versions (Wayback Machine, local snapshots)
3. Look for:
   □ New API endpoints — strings matching /api/, fetch(), axios calls
   □ New feature flags — isEnabled(), featureFlags, A/B test conditions
   □ Removed auth checks — authorization logic that was simplified or removed
   □ New parameters — query params, POST body fields not in previous versions
   □ Debug/staging code — console.log, debugMode, staging URLs leaked to production
   □ Hardcoded secrets — API keys, tokens, internal URLs that weren't there before
```

### API Specification Diffing

```
1. Fetch current OpenAPI/Swagger spec — /swagger.json, /api-docs, /openapi.yaml
2. Fetch previous version from Wayback Machine or version-controlled docs
3. Diff for:
   □ New endpoints — especially admin, internal, or debug paths
   □ New parameters — added fields that accept user input
   □ Changed auth requirements — endpoints that dropped auth or changed scopes
   □ New response fields — data leakage through expanded responses
   □ Deprecated-but-still-active endpoints — marked deprecated but still respond
```

### GraphQL Schema Diffing

```
1. Run introspection query — { __schema { types { name fields { name args { name } } } } }
2. Compare against previous schema snapshot
3. Look for:
   □ New mutations — especially those touching user data, payments, or permissions
   □ New types — expanded data models often have missing authorization
   □ Changed arguments — new optional args that might bypass validation
   □ Removed deprecation — re-enabled features may have stale security
```

### Mobile App Version Diffing

```
1. Download latest APK/IPA from store or third-party archive
2. Decompile and diff against previous version
3. Check for:
   □ New API endpoints in strings/resources
   □ Changed certificate pinning — pins removed or weakened
   □ New permissions — camera, location, contacts access
   □ New deep link handlers — potential for intent injection
   □ Hardcoded keys/tokens — especially for new integrations
```

### Scope Addition Hunting

```
1. Monitor program scope changes — new domains, IPs, mobile apps
2. For new scope additions:
   □ Full external recon immediately — subdomains, ports, tech stack
   □ Check for staging/dev environments at predictable paths
   □ Test default credentials on newly exposed services
   □ Look for shared infrastructure with existing scope (same CDN, same API gateway)
   □ Check if the new asset shares auth/sessions with existing targets
```

### Patch Analysis / Regression Testing

```
1. Monitor public advisories, CVE disclosures, changelogs
2. For security patches:
   □ Identify the root cause — was it input validation, auth check, logic error?
   □ Test if the fix is complete — same vuln class, different parameter?
   □ Test sibling endpoints — same pattern applied inconsistently?
   □ Test older API versions — /api/v1/ patched but /api/v2/ still vulnerable?
   □ Test mobile clients — web patched but mobile app still sends old request format?
```

## High-Signal Change Indicators

| Signal | Why It Matters | Where to Look |
|--------|---------------|---------------|
| New OAuth/SSO integration | Auth changes = fresh attack surface | Login page, /oauth/, /saml/ |
| Payment flow changes | Money movement = high severity | Checkout, subscription, refund endpoints |
| New file upload feature | Parser bugs, path traversal, SSRF | Upload endpoints, profile pictures, imports |
| API versioning bump | Old auth checks may not carry forward | /api/v2/, /api/v3/ vs /api/v1/ |
| New user role/tier | IDOR between role boundaries | Admin, enterprise, trial user flows |
| Mobile app major release | New features often ship without full security review | App store changelogs, decompiled APKs |
| Infrastructure migration | Cloud misconfigs during transition | DNS changes, new IP ranges, CDN switch |

## Output

For each change detected, I'll provide:

```markdown
### Change: [what changed]

**Risk assessment:** [High/Medium/Low] — [why]
**Test cases:**
1. [Specific test targeting this change]
2. [Specific test targeting this change]
3. [Specific test targeting this change]

**Expected finding class:** [IDOR/XSS/Auth bypass/etc.]
**Estimated severity if vulnerable:** [Critical/High/Medium/Low]
```

## Tips

- **Timing matters**: Hunt within 48 hours of a change for lowest duplicate risk
- **Changelogs are gold**: Read every release note, blog post, and status page update
- **Watch program updates**: Scope additions on HackerOne/Bugcrowd = fresh targets
- **JS source maps**: Some targets accidentally ship `.map` files in production — these make diffing trivial
- **Git blame pattern**: If source is available, `git log --since="2 weeks ago" --all -- "*.py" "*.js"` reveals recent security-relevant changes
