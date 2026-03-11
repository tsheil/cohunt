---
name: quick-test
argument-hint: "[target-url] [vuln-class]"
description: Rapid single-vulnerability check — give a target and vuln class, get a focused test plan with exact requests to send
arguments:
  - name: target
    description: "The target URL or endpoint (e.g., 'api.example.com/users/{id}')"
    required: true
  - name: vuln
    description: "Vulnerability class to test (e.g., 'IDOR', 'SSRF', 'XSS', 'SQLi', 'auth bypass')"
    required: true
---

# Quick Test

Rapid, focused vulnerability check. Give a target and vulnerability class — get back exact requests to send, what to look for, and how to confirm the bug.

## How It Works

When invoked, produce a **focused test plan** with:

### 1. Minimum Proof Setup
What accounts, tokens, or proof infrastructure you need before testing. **Match the proof to the bug class** — without the right setup, you'll find bugs you can't prove.

| Bug Class | Minimum Setup |
|-----------|--------------|
| IDOR / BOLA | 2 accounts (Account A's token + Account B's ID) |
| BFLA / Privilege escalation | 2 accounts at different roles (user + admin) |
| Blind XSS / SSRF | OAST collector URL (Burp Collaborator, interactsh) |
| Stored XSS | 2 accounts (inject with A, trigger in B's context) |
| Race condition | Single-use resource + parallel request tool |
| Business logic | Account with relevant state (pending invite, downgraded plan, etc.) |

**If you lack the setup, resolve it first.** A test without proof infrastructure is a waste of time.

### 2. Test Requests
Exact HTTP requests (curl commands) ready to copy-paste. Include:
- The baseline request (normal, authorized behavior)
- The attack request (the actual test)
- The verification request (confirming the bug)

Format each as a numbered step with a curl command and expected response.

### 3. What Confirms the Bug
Specific response indicators that prove the vulnerability:
- Status codes (200 when expecting 403)
- Response body differences (other user's data present)
- Timing differences (blind attacks)
- Error messages that leak information

### 4. False Positive Checks
Common reasons this might look like a bug but isn't:
- Cached responses showing stale data
- Public data that's supposed to be accessible
- Rate limiting vs actual authorization failure
- Different response codes that still deny access

### 5. If Confirmed — Next Steps
- How to escalate impact (read → write → delete)
- What to test next (related endpoints, similar patterns)
- Key details to capture for the report

## Example Output Format

For `/quick-test api.example.com/users/{id} IDOR`:

```
## Quick Test: IDOR on api.example.com/users/{id}

### Setup
- Account A: your regular account (token: $TOKEN_A)
- Account B: second test account (token: $TOKEN_B)
- Note Account B's user ID: $ID_B

### Tests

**Step 1: Baseline — access your own profile**
curl -s -H "Authorization: Bearer $TOKEN_A" \
  https://api.example.com/users/$ID_A | jq .

Expected: 200 with your profile data.

**Step 2: Attack — access Account B's profile with Account A's token**
curl -s -H "Authorization: Bearer $TOKEN_A" \
  https://api.example.com/users/$ID_B | jq .

Expected if vulnerable: 200 with Account B's data.
Expected if secure: 403 or 404.

**Step 3: Verify — confirm it's not cached/public data**
curl -s https://api.example.com/users/$ID_B | jq .

If this returns data without any auth header, the data is public (not a bug).

### Confirms the Bug If
- Step 2 returns 200 with Account B's private data
- Step 3 returns 401/403 (confirming auth is required, but authz is broken)

### False Positives
- Data returned is identical to what's publicly visible (check unauthenticated)
- Response is generic/empty but returns 200 (soft 200, no actual data)

### If Confirmed
- Test write: PUT/PATCH to /users/$ID_B with $TOKEN_A
- Test delete: DELETE /users/$ID_B with $TOKEN_A
- Test related: /users/$ID_B/settings, /users/$ID_B/billing
- Capture: screenshots of both responses side by side
```

## Supported Vulnerability Classes

Generate test plans for any of these (and variations):

| Class | Key Test |
|-------|----------|
| IDOR / BOLA | Swap object IDs between accounts |
| BFLA | Call admin endpoints with user tokens |
| Auth bypass | Access protected endpoints without valid auth |
| SSRF | Inject internal URLs in user-controlled parameters |
| XSS (reflected) | Inject payloads in reflected parameters |
| XSS (stored) | Store payload, trigger in another user's context |
| SQL injection | Inject SQL syntax, observe errors or data |
| Command injection | Inject OS commands in parameters |
| Path traversal | Read files outside intended directory |
| Open redirect | Redirect to external domain via parameter |
| CSRF | Perform state-changing action without valid token |
| Race condition | Send concurrent requests to exploit timing |
| Mass assignment | Send extra fields to modify protected attributes |
| JWT issues | Manipulate token claims or algorithm |
| GraphQL abuse | Introspection, batching, nested query DoS |
| Auth alternate path (CWE-288) | Send magic values, alternate endpoints, or undocumented params to bypass auth |
| Deserialization patch bypass | Test patched deser endpoints with alternate gadget chains (ysoserial) |
| Credential stuffing / reuse | Test credential-reuse protections and account lockout behavior |
| Indirect prompt injection | Inject instructions in content AI processes (docs, emails, comments) |
| Tool poisoning / MCP abuse | Inject malicious tool descriptions or hidden schema params |
| DXT/MCP confused deputy | Chain low-privilege data source (calendar, email) to code execution |
| Project config poisoning | Plant .claude/, .mcp.json, .cursorrules, copilot-instructions.md in repos |
| Pre-trust config exfil | Redirect API keys via env vars/settings before user grants trust |
| Memory poisoning | Inject persistent instructions that survive across sessions |
| LLM output injection | Prompt LLM to include XSS/SQLi/command injection in rendered output |
| System prompt extraction | Extract internal APIs, credentials, or config from system prompt |

For any class not listed, generate a reasonable test plan following the same structure.

## Stateful Workflow Mode

For vulnerabilities beyond simple IDOR (invite flows, approvals, checkouts, OAuth, async workers, webhooks), use the multi-actor stateful test pattern:

### Actor Setup

```
Define actors for the test scenario:
□ Attacker — the threat actor exploiting the vulnerability
□ Victim — the user whose data/account is affected
□ Admin — privileged user (if relevant to escalation)
□ Webhook receiver — external endpoint capturing callbacks
□ Background worker — async process that acts on queued data
```

### State Transition Testing

```
For each workflow (e.g., invite flow, payment, approval):

1. Map the state machine:
   [Initial] → [Pending] → [Approved] → [Completed]

2. Test state manipulation:
   □ Can you skip states? (jump from Initial → Completed)
   □ Can you reverse states? (Completed → Pending)
   □ Can you act on another user's state transition?
   □ Does the transition enforce the correct actor? (only approver can approve)
   □ Can you modify data after a state is "locked"?

3. Test race conditions at state boundaries:
   □ Send approve + deny simultaneously
   □ Send two withdrawals against the same balance
   □ Modify cart after checkout initiated but before payment
```

### Multi-Step Workflow Tests

```
Invite Flow:
□ Can attacker accept invite meant for victim? (IDOR on invite token)
□ Can expired invite be replayed?
□ Can invite be used to escalate role beyond what was granted?

Payment/Checkout:
□ Can item price be modified between cart and payment?
□ Can attacker use victim's saved payment method?
□ Can refund be triggered by non-purchaser?
□ Is the payment amount validated server-side after client sends total?

Approval Workflow:
□ Can requester approve their own request?
□ Can a lower-role user approve a higher-role action?
□ Can approval be bypassed by directly calling the post-approval endpoint?
```

## Design Principles

- **Copy-paste ready** — Every test should be a curl command you can run immediately
- **Two-account pattern** — Always use two accounts to prove cross-account access
- **Stateful pattern** — For workflows, map the state machine and test transitions
- **Baseline first** — Establish normal behavior before testing the attack
- **Confirmation step** — Every test includes how to verify it's not a false positive
- **Minimal scope** — Test one thing at a time, don't combine vulnerability classes
