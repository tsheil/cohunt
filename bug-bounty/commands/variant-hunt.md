---
name: variant-hunt
argument-hint: "[CVE, disclosed report, patch, or feature]"
description: Turn a known vulnerability into new findings — hunt for sibling endpoints, incomplete fixes, and pattern variants. Low duplicate risk, high hit rate
arguments:
  - name: source
    description: "The starting point — a CVE ID, disclosed report URL, patch diff, or feature name (e.g., 'CVE-2026-29000', 'Shopify IDOR in /admin/api', 'they just patched their GraphQL endpoint')"
    required: true
---

# /variant-hunt

Turn known vulnerabilities into new findings. When a bug is disclosed or patched, there are almost always siblings — same pattern on different endpoints, incomplete fixes, or the same class of bug in related features. Variant hunting has one of the highest hit rates in bug bounty because you're working from a confirmed vulnerability pattern.

## Usage

```
/variant-hunt <CVE, report, patch, or feature>
```

Starting variant hunt from: $ARGUMENTS

---

## Variant Hunting Strategies

### Strategy 1: Sibling Endpoints

A bug on one endpoint usually means the same class of bug exists on related endpoints. The developer who wrote the vulnerable code likely wrote similar code elsewhere.

```
Given: IDOR on GET /api/v2/orders/{id}
Hunt for:
□ GET /api/v2/invoices/{id}     — same pattern, different resource
□ GET /api/v2/shipments/{id}    — related resource in same domain
□ GET /api/v1/orders/{id}       — older API version (often less hardened)
□ POST /api/v2/orders/{id}      — write variant of read bug
□ DELETE /api/v2/orders/{id}    — delete variant (higher severity)
□ GET /api/v2/orders/{id}/items — child resources
□ GET /api/internal/orders/{id} — internal API (if discoverable)
□ GraphQL query { order(id: X) } — same data via different interface
```

**Methodology:**
1. Take the vulnerable endpoint pattern
2. List all endpoints that share the same: resource type, API version, authentication method, or developer team
3. Test each with the same attack that worked on the original
4. Check both the same HTTP method AND other methods (GET→POST→PUT→DELETE)

### Strategy 2: Incomplete Fix

Patches often fix the specific reported case but miss edge cases. This is one of the most reliable variant patterns.

```
Given: XSS via <script> tag was patched
Hunt for:
□ <img onerror=...>          — different tag, same sink
□ <svg onload=...>           — SVG-based bypass
□ javascript: protocol       — protocol handler bypass
□ Unicode normalization       — encoding bypass
□ Double encoding            — filter applied once, decoded twice
□ Context switching          — XSS in attribute vs element context
□ Different input field      — patch applied to field A but not field B
```

```
Given: SSRF via url= parameter was patched (blocks internal IPs)
Hunt for:
□ DNS rebinding              — resolves to external then internal
□ IPv6 mapping               — [::1] or [0:0:0:0:0:ffff:127.0.0.1]
□ Decimal IP                 — 2130706433 instead of 127.0.0.1
□ Redirect chain             — external URL redirects to internal
□ Different parameter        — url= patched but redirect_uri= not
□ Different protocol         — http:// blocked but file:// or gopher:// not
□ Cloud metadata             — 169.254.169.254 vs metadata.google.internal
```

**Methodology:**
1. Read the patch/fix details (changelog, commit, advisory)
2. Identify exactly what was blocked or fixed
3. List what was NOT addressed by the fix
4. Test each gap systematically

### Strategy 3: Cross-Feature Pattern

If a developer made a mistake in one feature, the same thought pattern likely appears in other features they built.

```
Given: Missing authorization check on Team Settings
Hunt for:
□ Billing settings           — same developer, same oversight pattern
□ Integrations page          — admin-only features with weak authz
□ Audit log access           — often forgotten in authz checks
□ Export/download features   — data access without proper checks
□ Webhook configuration      — write access to sensitive settings
□ API key management         — critical feature, same authz pattern
```

**Methodology:**
1. Identify the root cause class (missing authz, input validation gap, logic error)
2. Find all features that require the same type of check
3. Test whether each feature implements the check correctly
4. Pay special attention to newer features (added after the original, may not have gotten the fix)

### Strategy 4: Version/API Regression

When a fix is applied to the current version, older API versions often remain vulnerable.

```
Given: Bug fixed in /api/v3/users
Hunt for:
□ /api/v2/users              — older version, likely unpatched
□ /api/v1/users              — even older, definitely unpatched
□ /api/beta/users            — beta endpoints often lack security
□ /api/internal/users        — internal APIs often unpatched
□ /mobile/api/users          — mobile-specific API
□ /graphql                   — different interface to same data
□ /legacy/users              — legacy endpoints kept for backwards compat
```

### Strategy 5: From CVE to Target

When a CVE is published, check if your target uses the affected component.

```
Given: CVE-2026-XXXXX in [library/framework]
Hunt for:
1. Check target's tech stack (via recon) for affected component
2. Check version (headers, JS bundles, package.json, /api/version)
3. If vulnerable version detected:
   □ Reproduce the CVE PoC against the target
   □ Check if WAF/custom code blocks the specific PoC
   □ Try bypass variants if blocked
4. If patched version detected:
   □ Check if the patch was applied correctly
   □ Test for incomplete patches (common with complex CVEs)
   □ Check for custom code that reintroduces the pattern
```

### Strategy 6: AI/MCP Component Variants

When a vulnerability is found in one AI/MCP component, the same pattern almost always exists in sibling components — different MCP servers, different AI frameworks, or different transports.

```
Given: Command injection in MCP server wrapping kubectl (CVE-2026-XXXXX)
Hunt for:
□ Other CLI-wrapping MCP servers (git, nmap, ffuf, biome, sqlmap)
□ Same server, different tool endpoints (server has 10 tools, only 1 tested)
□ Same parameter, different transport (HTTP vs stdio vs SSE)
□ Same pattern in competitor products (Cursor → Windsurf → Claude Code)
□ Gateway layer (MCP-to-OpenAPI gateway passes params unsanitized)
```

```
Given: Prompt injection in AI assistant feature
Hunt for:
□ Same injection via different input channels (chat, file upload, API, email)
□ Indirect injection surfaces (shared docs, issue titles, webhook payloads)
□ Cross-transport: injection blocked in HTTP but works via WebSocket/SSE
□ Schema drift: tool definitions changed since last audit (hidden params, altered defaults)
□ Different agent modes: injection works in one model/temperature but not another
```

**AI/MCP Variant Checklist:**

| Source Pattern | Variant Targets |
|---|---|
| MCP tool poisoning | Other tools on same server, other servers in same deployment, tool shadowing via similar names |
| OAuth scope confusion | SCIM/JIT provisioning drift, API key scope vs OAuth scope, service account residual perms |
| CLI-wrapper injection | Every `child_process.exec()` or `subprocess(shell=True)` in the server — search source for all instances |
| Prompt injection in feature A | Same injection in features B, C, D — especially features added after A was hardened |
| Transport-parity gap | Security control on HTTP but missing on WebSocket subscriptions (CVE-2026-30241 pattern) |
| Config/rules file poisoning | `.cursorrules` → `.github/copilot-instructions.md` → `.claude/settings.json` → `.windsurf/rules` |

**Methodology:**
1. Identify the root cause class (injection sink, missing validation, transport gap)
2. Enumerate all components that share the same pattern (same SDK, same developer, same architecture)
3. Test each systematically — AI/MCP ecosystems are young and fixes are often applied to one component but not siblings
4. Each confirmed variant on a different component is a separate report

---

## Output Format

```markdown
# Variant Hunt: [Source Vulnerability]

**Source:** [CVE/report/patch reference]
**Root Cause:** [The underlying pattern — e.g., "missing server-side authorization on resource access"]
**Target:** [domain/program]

## Variant Hypothesis

Based on [source], these variants are likely to exist:

### High Probability Variants
| # | Variant | Endpoint/Feature | Test Method | Why Likely |
|---|---------|-----------------|-------------|-----------|
| 1 | [description] | [where to test] | [how to test] | [reasoning] |

### Medium Probability Variants
| # | Variant | Endpoint/Feature | Test Method | Why Likely |
|---|---------|-----------------|-------------|-----------|

### Speculative Variants
| # | Variant | Endpoint/Feature | Test Method | Why Likely |
|---|---------|-----------------|-------------|-----------|

## Test Plan

### Test 1: [Highest probability variant]
**Hypothesis:** [What you expect to find]
**Steps:**
1. [Exact test step with curl/request]
2. [What to observe]
3. [How to confirm]

**If confirmed → also test:** [next related variant]
**If not vulnerable → try:** [alternative approach]

### Test 2: [Second variant]
...

## Reporting Notes
- Each confirmed variant is a separate report (unless they share the exact same root cause fix)
- Reference the original CVE/report to help the triager understand the pattern
- Emphasize that this is a *new* finding, not a duplicate of the original
```

---

## Tips

1. **Variants are NOT duplicates** — A bug on endpoint A and the same class of bug on endpoint B are separate findings, each worth a bounty
2. **Read the patch** — Understanding exactly what was fixed tells you exactly what wasn't
3. **Check all API versions** — v1 and v2 often coexist, and only the latest gets patched
4. **Time it right** — Hunt variants immediately after a disclosure, before other hunters catch on
5. **Reference the original** — Mentioning "this is a variant of [CVE/report]" helps triagers understand and accept faster
6. **Don't over-claim** — If it's truly the same root cause fix, it's a duplicate. Variants must require a separate fix
