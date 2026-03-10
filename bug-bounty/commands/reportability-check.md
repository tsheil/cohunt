---
name: reportability-check
argument-hint: "[describe your finding]"
description: Early go/no-go gate — check if a finding is worth reporting before investing time in a full write-up. Catches common N/A and duplicate patterns
arguments:
  - name: finding
    description: "Describe what you found (e.g., 'IDOR on /api/users/{id} returns other users profile data', 'self-XSS in profile bio field')"
    required: true
---

# /reportability-check

Quick triager-style assessment of whether a finding is worth reporting. Run this BEFORE writing a full report — it catches the patterns that lead to N/A, Informational, or Duplicate verdicts and saves you hours of wasted effort.

## Usage

```
/reportability-check <describe what you found>
```

Checking reportability: $ARGUMENTS

---

## Assessment Framework

Evaluate the finding against these 5 gates. ALL must pass for a Go verdict.

### Gate 1: Trust Boundary Crossing

**Question:** Does the attacker affect something beyond their own account?

| Result | Verdict |
|--------|---------|
| Attacker can access/modify another user's data | PASS |
| Attacker can escalate their own privileges | PASS |
| Attacker can affect server-side state for others | PASS |
| Attacker can only affect their own account | FAIL — self-only impact is almost always N/A |
| Attacker needs victim to perform unusual action | WARN — severity capped, add caveat |

**Common N/A patterns:**
- Self-XSS (only fires in your own browser)
- Modifying your own profile data via API (that's just using the API)
- Rate limiting your own account
- CSRF on non-state-changing endpoints (logout CSRF)
- Missing security headers with no demonstrated exploit

### Gate 2: Reproducibility

**Question:** Can you reproduce this reliably in a clean environment?

| Result | Verdict |
|--------|---------|
| Reproduces every time with documented steps | PASS |
| Reproduces intermittently (race condition) | PASS — but capture multiple successful attempts |
| Worked once, can't reproduce | FAIL — do not report until reproducible |
| Requires special timing/conditions | PASS — document the conditions precisely |

**Red flags:**
- "I saw this error once" — not reproducible, don't report
- "The response was different" — could be caching, load balancing
- "It worked in development" — must work in production (if in-scope)

### Gate 3: Real Impact

**Question:** What's the concrete harm to the company or its users?

| Impact | Severity Estimate |
|--------|------------------|
| Read other users' PII (email, phone, address) | Medium-High |
| Read other users' financial data | High-Critical |
| Write/modify other users' data | High |
| Delete other users' data | High-Critical |
| Execute code on server | Critical |
| Bypass payment/subscription | High (with $ quantification) |
| Account takeover chain | Critical |
| Read non-sensitive metadata only | Low-Informational |

**Impact must be concrete, not theoretical:**
- BAD: "An attacker could potentially..." (speculative)
- GOOD: "I accessed user ID 12345's email address and phone number using my low-privilege account token"

### Gate 4: Scope & Duplicate Check

**Question:** Is this in scope and likely not already reported?

| Check | Action |
|-------|--------|
| Endpoint is in program scope? | Verify against scope definition |
| Vulnerability type is accepted? | Check exclusions (often: self-XSS, logout CSRF, rate limiting, missing headers) |
| Similar finding in program's hacktivity? | Search disclosed reports for same endpoint/pattern |
| Common automated finding? | If basic XSS/SQLi/SSRF on a popular program → high duplicate risk |
| Known limitation or by-design? | Check program policy and FAQ |

**High duplicate risk indicators:**
- Simple reflected XSS on a top-100 HackerOne program
- IDOR on an obviously enumerable endpoint
- Missing rate limiting on login
- Information disclosure in error messages
- Open redirect without demonstrated chain

### Gate 5: Evidence Quality

**Question:** Do you have proof that would convince a skeptical triager?

| Evidence | Required? |
|----------|-----------|
| HTTP request/response showing the vulnerability | Yes |
| Two-account proof (attacker + victim) | Yes for access control bugs |
| Before/after showing state change | Yes for write/delete bugs |
| Full chain completion (not just first step) | Yes for chain-dependent findings |
| Video or step-by-step screenshots | Strongly recommended |
| Impact quantification with real numbers | Recommended for business logic |

---

## Verdict Output

After assessment, output one of:

### GO — Report This

```
## Reportability: GO

**Finding:** [one-line summary]
**Estimated Severity:** [Critical/High/Medium/Low]
**Duplicate Risk:** [Low/Medium/High]
**Confidence:** [High/Medium]

**Why it's reportable:**
- [Gate 1 pass reason]
- [Gate 3 impact statement]

**Before submitting, verify:**
- [ ] [Any remaining evidence to capture]
- [ ] [Any edge case to confirm]

**Recommended next steps:**
→ Run `/write-report` to draft the submission
```

### MAYBE — Needs More Work

```
## Reportability: MAYBE — Needs Strengthening

**Finding:** [one-line summary]
**Blocker:** [What's missing]

**To upgrade to GO:**
1. [Specific action — e.g., "Demonstrate cross-account impact with second test account"]
2. [Specific action — e.g., "Complete the chain by showing data is actually sensitive"]
3. [Specific action — e.g., "Check if endpoint is in scope per program policy"]

**If you can't resolve the blocker:**
→ Move on — time is better spent finding a clean finding
```

### NO-GO — Don't Report

```
## Reportability: NO-GO

**Finding:** [one-line summary]
**Reason:** [Which gate failed and why]

**This will likely be marked:** [N/A / Informational / Duplicate / Out of Scope]

**Common N/A patterns this matches:**
- [Pattern name — e.g., "Self-XSS", "Missing header without exploit", "CSRF on logout"]

**Salvage options:**
- [Can this chain with another finding? → try `/chain`]
- [Is there a harder variant? → try `/variant-hunt`]
- [Is the real bug elsewhere? → try `/pivot`]
```

---

## Quick N/A Reference

These findings are almost always N/A on major platforms. Don't report unless you can demonstrate exceptional impact:

| Finding | Why N/A | Unless... |
|---------|---------|-----------|
| Self-XSS | Only affects attacker | Chains with CSRF to become stored XSS |
| CSRF on logout | Minimal impact | Chains with session fixation |
| Missing X-Frame-Options | No demonstrated clickjacking | You demo a working clickjacking PoC with real impact |
| Missing rate limiting on login | Often by-design | You demonstrate account lockout affecting other users |
| SPF/DKIM/DMARC misconfiguration | Usually out of scope | Program explicitly includes email security |
| Open redirect (standalone) | Low impact alone | Chains with OAuth to steal tokens |
| Information disclosure (version numbers) | Low impact | Version has known critical CVE |
| Verbose error messages | Informational | Leaks credentials, internal IPs, or code |
| Cookie without Secure/HttpOnly flag | Best practice, not a vuln | You demonstrate actual session theft via this gap |
| Password policy issues | Usually informational | You demonstrate credential stuffing at scale |

---

## AI/MCP-Specific Reportability

AI and MCP findings have unique acceptance criteria. Many triagers are still learning this space — help them by hitting these evidence bars.

### AI/MCP Reportability Rubric

| Gate | What to Prove | Evidence Required |
|------|--------------|-------------------|
| **Unauthorized tool action** | Agent performed an action the user didn't request or approve | Full request/response chain showing: (1) user's original prompt, (2) injected instruction, (3) agent's unauthorized action. Video strongly recommended |
| **Cross-user / cross-tenant impact** | Finding affects users other than the attacker | Two-account proof: attacker's injection → victim's data exposed or modified. Show tenant isolation failure if multi-tenant |
| **Session persistence** | Injected instructions survive across conversations | Demonstrate: (1) inject in session A, (2) close session, (3) open session B, (4) injection still active. Timestamp evidence |
| **Cost amplification** | Finding causes disproportionate resource consumption | Quantify: "$X in API credits consumed per attack request" or "142x token amplification" — real dollar amounts |
| **Data exfiltration path** | Sensitive data leaves the system via AI channel | Show complete exfil chain: injection → data access → data transmitted to attacker-controlled endpoint. HTTP logs as proof |

### Common AI/MCP N/A Patterns

| Finding | Why Often N/A | Upgrade Path |
|---------|-------------|-------------|
| System prompt extraction (no secrets) | Low impact — most programs treat this as informational | Show the system prompt contains API keys, internal URLs, or credentials |
| Basic chatbot jailbreak | Safety filter bypass alone is usually informational unless program explicitly scopes it | Chain with data access, unauthorized action, or output injection (XSS/SQLi) |
| Prompt injection on your own session | Self-only impact, same as self-XSS | Demonstrate indirect injection (attacker plants payload, victim triggers it) |
| MCP tool description disclosure | Tool descriptions are often meant to be visible | Show descriptions contain credentials, internal endpoints, or injection payloads |
| Theoretical "could potentially" agent abuse | Speculative impact = N/A | Demonstrate actual unauthorized action with concrete evidence |
| DoS via large prompt | Resource exhaustion without amplification = Low | Quantify amplification ratio (e.g., 1KB input → 100MB processing) |

### AI/MCP Severity Calibration

```
Critical: Cross-user data exfiltration via AI, RCE through agent tool abuse,
          persistent memory poisoning affecting all users
High:     Single-user data exfil with indirect injection, unauthorized tool
          actions with real impact, credential exposure via AI channel
Medium:   Session-scoped injection requiring user interaction, cost
          amplification with measurable but bounded impact
Low:      Self-only system prompt leak, safety bypass without chain,
          theoretical agent abuse without demonstrated impact
```

---

## Tips

1. **Run this check early** — 10 minutes of assessment saves 2 hours writing a doomed report
2. **Be your own triager** — Read the finding as if you're paid to reject it. What would you challenge?
3. **When in doubt, chain it** — A Medium finding + another Medium finding = a High report
4. **Check hacktivity BEFORE reporting** — 5 minutes of search prevents a duplicate
5. **Quantify everything** — "$X impact affecting Y users" beats "attacker could potentially..."
