---
name: business-logic
description: Tests business logic vulnerabilities — the #1 bounty-paying class (45% of all awards) and the human hunter's strongest edge over autonomous tools. Covers payment flow exploitation, state machine mapping, subscription bypass, multi-step workflow abuse, race conditions with financial impact, referral system gaming, B2B SaaS workflows (SCIM, invites, shared links, connectors, real-time collaboration), and multi-tenant isolation. Activates when testing e-commerce checkout, pricing, subscriptions, approvals, invite flows, shared link abuse, SaaS connector integrations, or any application-specific workflow that autonomous scanners can't reason about. Trigger on "business logic", "payment bypass", "checkout flow", "subscription bypass", "race condition money", "coupon abuse", "price manipulation", "workflow skip", "state machine", "referral abuse", "feature toggle", "paywall bypass", "approval workflow", "trial extension", "shared link", "invite flow", "connector abuse", "webhook replay", "seat limit", "B2B SaaS". For auth/access control bugs (BOLA, BFLA, JWT, OAuth), use auth-testing. For technical vuln classes (XSS, SQLi, SSRF), use vuln-patterns. For race condition protocol details, use http-desync.
---

# Business Logic Vulnerability Testing

The #1 bounty-paying class (45% of all awards, Intigriti 2026) and the area where human hunters have the strongest edge over autonomous tools. Business logic bugs require understanding what the application is *supposed* to do, then finding where implementation diverges from intent.

## Quick Start — First 5 Minutes

1. **Map the money flow** — Find checkout/payment/billing endpoints. Intercept, change `price`, `quantity`, or `currency`.
2. **Skip steps** — Identify a multi-step workflow (checkout, signup, approval). Jump from step 1 to step 3's API.
3. **Race the limit** — Find a single-use action (coupon, invite, vote). Send 20 parallel requests via HTTP/2 single-packet.
4. **Cross-tenant swap** — If B2B SaaS: swap `org_id` or `tenant_id` in any admin/export/webhook endpoint.
5. **Downgrade + retain** — Downgrade from premium to free tier. Check if premium API endpoints still respond.

If you have a hit, run `/reportability-check` before `/write-report`. If blocked or it looks patched, run `/variant-hunt`.

**B2B SaaS targets — test these first** (highest value, lowest competition):

| Surface | Key Test | Why It Pays |
|---------|----------|-------------|
| SCIM provisioning | Cross-tenant user creation via SCIM token scope bypass | Critical — admin access in wrong tenant |
| SSO/JIT | JIT into wrong tenant via email domain collision | Critical — cross-tenant account creation |
| Invitations | Role escalation on invite acceptance | High — join as admin instead of viewer |
| Support impersonation | Impersonate admin from support agent context | Critical — full ATO at scale |
| Data exports | IDOR on export endpoint (`org_id` swap) | High — bulk data exfiltration |
| Approval workflows | Self-approval or bypass via direct API call | High — separation of duties bypassed |

---

## Why Business Logic Pays More

Autonomous tools (XBOW, Shannon, Codex Security) scan for technical patterns — they can't understand business intent:

- **Requires domain understanding** (e-commerce, SaaS, fintech)
- **No signature to match** — each app's logic is unique
- **Multi-step exploitation chains** that scanners can't reason about
- **Context-dependent** — same action may be valid or exploitable depending on state
- **Low duplicate risk** — each finding is application-specific

**Severity guidance:** Business logic bugs often score Medium-High on CVSS but pay High-Critical bounties because programs weight financial and business impact beyond the CVSS formula.

---

## State Machine Mapping

Most business logic bugs come from incomplete state machines — the application allows transitions that shouldn't be possible.

### How to Map a State Machine

1. **Identify the workflow** — e.g., order placement: Browse → Cart → Checkout → Payment → Confirmation → Fulfilled
2. **Document each state** — What data exists at each stage? What API calls happen?
3. **Map valid transitions** — Which states can lead to which other states?
4. **Test invalid transitions** — Can you skip states? Go backwards? Jump to the end?
5. **Test parallel transitions** — Can you be in two states simultaneously?

### Testing Patterns

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Skip validation steps | Jump from Cart directly to Confirmation endpoint (skip Payment) | Order placed without payment |
| 2 | Reverse transitions | After payment, revert to Cart and modify items, then re-confirm | Different items delivered at original price |
| 3 | Replay completed states | Replay the Payment confirmation request multiple times | Duplicate credit, multiple deliveries |
| 4 | Partial completion | Start checkout in two browser sessions, complete one, use the other | Stale session processes with outdated data |
| 5 | State parameter tampering | Modify order status field in API (e.g., `"status": "fulfilled"`) | Status updated without proper authorization |
| 6 | Timeout exploitation | Start a time-sensitive flow, wait for timeout, then complete it | Expired flow completes successfully |
| 7 | Concurrent state mutation | Modify the same resource from two sessions simultaneously | Inconsistent state, race condition |

### What Makes It Reportable

- **Valid:** "By skipping step 3 of the checkout flow, I can place an order without payment verification, resulting in $X of goods shipped without charge."
- **Invalid:** "I can call the API endpoints out of order." (So what? Does it cause harm?)

---

## Payment & Checkout Flows

The highest-value business logic targets. Programs pay Critical for any finding that allows unauthorized financial transactions.

### Price Manipulation

| # | Test | Method | What to look for |
|---|------|--------|-----------------|
| 1 | Client-side price | Intercept checkout request, modify `price` or `amount` field | Order placed at modified price |
| 2 | Currency switching | Change currency mid-transaction (e.g., USD→INR at checkout) | Charged in lower-value currency, items at original price |
| 3 | Discount stacking | Apply multiple discounts: loyalty + coupon + referral + first-time | Discounts exceed 100% or result in negative total |
| 4 | Quantity boundaries | Set quantity to 0, -1, 0.001, MAX_INT, or NaN | Credit/refund generated, or inventory inconsistency |
| 5 | Tax evasion | Modify billing address to tax-free jurisdiction mid-checkout | Tax removed from order total |
| 6 | Shipping fee bypass | Set shipping method to null or modify fee to 0 after selection | Free shipping on paid-shipping orders |
| 7 | Gift card math | Buy $100 gift card, use to buy item, return item, get $100 credit + gift card | Doubled credit |
| 8 | Partial payment abuse | Pay with split methods, cancel one after confirmation | Full order with partial payment |

### Refund/Chargeback Logic

| # | Test | Method | What to look for |
|---|------|--------|-----------------|
| 1 | Double refund | Request refund via API while also requesting via UI simultaneously | Refund processed twice |
| 2 | Refund amount tampering | Modify refund amount in request to exceed original purchase | Refund greater than payment |
| 3 | Refund to different method | Request refund to a different payment method than original | Money moved to attacker-controlled account |
| 4 | Return without return | Initiate return, get refund confirmation, but cancel physical return | Refund issued without goods returned |
| 5 | Credit balance abuse | Accumulate store credits via refund/return loops | Unlimited store credit |

---

## Subscription & Feature Access

### Paywall Bypass Patterns

| # | Test | Method | What to look for |
|---|------|--------|-----------------|
| 1 | Trial extension | Create new trial with same payment method or email variant (`user+1@email.com`) | Unlimited free trials |
| 2 | Plan downgrade + feature retention | Downgrade from Premium to Free, check if premium API endpoints still respond | Premium features on free tier |
| 3 | Direct API access | Call premium-only API endpoints with free-tier auth token | Features accessible without subscription |
| 4 | Feature flag manipulation | Modify client-side feature flags or `plan` field in requests | Premium features unlocked |
| 5 | Grace period abuse | Cancel subscription, continue using during grace period, re-cancel before charge | Permanent free access via cancel loop |
| 6 | Seat count bypass | On team plans, add more users than allowed by seat count | Extra team members without payment |
| 7 | Plan upgrade race | Rapidly switch between plans during billing transition | Charged for lower plan, given higher plan |
| 8 | API rate limit tier confusion | Use free-tier token but send requests at premium rate | Rate limits don't match plan |

### Quota & Limit Exploitation

| # | Test | Method | What to look for |
|---|------|--------|-----------------|
| 1 | Counter reset | Check when usage counters reset (midnight? billing cycle?) and exploit timing | Extra usage in gap period |
| 2 | Parallel requests | Send burst of requests before counter updates | Exceed limit before enforcement |
| 3 | Limit per-resource vs per-account | Create multiple projects/workspaces, each with its own limit | Effective limit multiplied |
| 4 | API version bypass | Use older API version that doesn't enforce new limits | Unlimited access via legacy API |

---

## Multi-Step Workflow Exploitation

### Common Vulnerable Workflows

**User Registration:**
- Skip email verification → access account
- Modify role during registration (`{"role": "admin"}`)
- Register with another user's email during verification delay
- Bypass invite-only by directly calling registration endpoint

**Password Reset:**
- Use reset token for different account
- Token not invalidated after use → replay
- Token predictable (sequential, timestamp-based)
- Host header poisoning → reset link points to attacker domain
- Race condition: request reset for two accounts, use token interchangeably

**Invitation Systems:**
- Accept invite, modify invite to change permission level
- Accept same invite from multiple sessions
- Use expired invite link (not properly invalidated)
- Modify invite payload to target different organization

**Approval Workflows:**
- Submit request, modify it after approval but before execution
- Self-approve by manipulating approver ID
- Bypass approval by calling execution endpoint directly
- Race condition: submit + approve simultaneously

### Testing Methodology

```
For each multi-step workflow:
1. Map every API call in the flow (Burp Suite proxy)
2. For each call, test:
   a. Can I call it out of sequence?
   b. Can I modify parameters between steps?
   c. Can I replay it after completion?
   d. Can I perform the same step from two sessions?
   e. Can I skip this step entirely?
3. Document the state at each step
4. Test each transition for authorization checks
```

---

## Race Conditions in Business Context

Race conditions are business logic bugs when the timing exploit has financial or authorization impact.

### High-Value Race Targets

| Target | Attack | Impact | Technique |
|--------|--------|--------|-----------|
| Coupon redemption | Redeem same coupon in parallel requests | Multiple discounts from single-use coupon | HTTP/2 single-packet |
| Balance transfer | Transfer full balance to two recipients simultaneously | Balance doubled | Turbo Intruder |
| Invitation acceptance | Accept limited invitation from two accounts | Both accounts get access to limited-seat resource | Parallel sessions |
| Vote/like systems | Submit multiple votes in single-packet attack | Vote manipulation, ranking abuse | Last-byte sync |
| Account creation with bonus | Create account + claim bonus in parallel | Multiple sign-up bonuses | Race window in signup flow |

### Timing-Based Testing Approach

```
1. Identify the check-then-act pattern:
   - Read balance → Check >= amount → Deduct → Transfer
2. Find the race window:
   - Time between check and act (often 10-500ms)
3. Exploit the window:
   - HTTP/2 single-packet attack for sub-millisecond precision
   - Turbo Intruder for high-volume parallel requests
   - Last-byte synchronization for exact timing
4. Verify with evidence:
   - Show the duplicated transaction/action in the response
   - Calculate the financial impact
```

---

## Referral & Reward Systems

| # | Test | Method | What to look for |
|---|------|--------|-----------------|
| 1 | Self-referral | Use email aliases (`user+ref1@email.com`) to refer yourself | Unlimited referral bonuses |
| 2 | Referral code brute-force | Enumerate referral codes to credit to your account | Unauthorized bonus accumulation |
| 3 | Referral after conversion | Apply referral code after account creation (if flow allows) | Retroactive bonus application |
| 4 | Referral chain exploitation | A refers B, B refers C, C refers A — circular chain | Infinite referral loop |
| 5 | Reward timing abuse | Complete reward condition, claim reward, undo condition | Keep reward after reversal |
| 6 | Points/loyalty manipulation | Modify point balance in request, or exploit conversion ratios | Inflated loyalty balance |
| 7 | Gamification bypass | Modify achievement/milestone parameters in API | Unlock rewards without completion |

---

## Multi-Tenant Isolation

These bugs are high-severity because they cross organizational boundaries.

| # | Test | Method | What to look for |
|---|------|--------|-----------------|
| 1 | Tenant ID swap | Change `org_id`, `tenant_id`, or `workspace_id` in API requests | Access to another tenant's data |
| 2 | Webhook cross-tenant | Configure webhook that receives events from other tenants | Data leakage via event system |
| 3 | Shared resource pollution | Upload file, check if accessible from different tenant context | Cross-tenant file access |
| 4 | Admin panel isolation | Access admin features from one tenant using another tenant's admin token | Cross-tenant admin actions |
| 5 | API key scope confusion | Use API key from tenant A to access tenant B's resources | Key not properly scoped to tenant |
| 6 | Subdomain isolation | Access `tenant-a.app.com` resources from `tenant-b.app.com` session | Session/cookie not tenant-scoped |
| 7 | Shared database inference | Enumerate IDs from tenant A to discover tenant B's record counts | Information disclosure via sequential IDs |
| 8 | Event/notification bleed | Receive notifications or events intended for another tenant | Real-time data exposure |

---

## Monetary Impact Quantification

Programs care about dollars. Quantifying financial impact elevates severity.

```
Financial Impact = Affected Users × Average Transaction Value × Exploitation Frequency

Where:
- Affected Users: How many users could an attacker target?
- Average Transaction Value: What's at stake per exploitation?
- Exploitation Frequency: How often could this be exploited?
```

### Example Impact Statements

**Weak:** "An attacker could manipulate prices."
**Strong:** "An attacker can reduce any item's price to $0.01 at checkout by modifying the `unit_price` parameter. With an average order value of $85 and approximately 10,000 daily orders, this could result in revenue loss of up to $850,000 per day if exploited at scale."

### Severity Elevation Guide

| Impact Type | Base Severity | With Quantification |
|-------------|--------------|---------------------|
| Price manipulation | Medium | High-Critical (depending on $ amount) |
| Unlimited free tier | Low-Medium | Medium-High (with subscription revenue impact) |
| Double-spend | High | Critical (with transaction volume) |
| Cross-tenant data access | High | Critical (with regulatory context: GDPR, HIPAA) |
| Referral abuse | Low | Medium (with fraud prevention context) |

---

## Validation Gate: Is This Submittable?

Before writing a report, run this checklist:

| Check | Required? | Details |
|-------|-----------|---------|
| **Crosses a trust boundary** | Yes | Attacker affects someone/something other than themselves |
| **Reproducible** | Yes | Can reproduce in clean environment, not just once |
| **In scope** | Yes | Confirmed feature/endpoint is in program scope |
| **Financial or security impact** | Yes | Quantifiable harm — dollars, data, access |
| **Not a known limitation** | Yes | Not documented as "by design" or known issue |

### Common Mistakes in Business Logic Reports

| Mistake | Fix |
|---------|-----|
| "I changed the price to $0" without showing the order completed | Complete the full exploitation chain — show the order confirmation |
| Describing the vulnerability without quantifying impact | Add financial impact calculation with real numbers |
| Not testing if server-side validation exists | Show that the server accepted the modified value, not just that the client sent it |
| Race condition without evidence of successful double-spend | Show both transactions completed with evidence (database state, two confirmation emails) |
| "Any user could do this" without specifying prerequisites | List exact requirements: authentication level, account state, feature access |

---

## B2B SaaS Enterprise Workflows

The highest-value, lowest-competition attack surface. B2B SaaS admin workflows combine business logic with authorization and are systematically undertested by autonomous tools.

> **Detailed playbook:** Read `reference/b2b-saas-playbook.md` for complete test patterns covering SCIM, SSO/JIT, invitations, role sync, seat enforcement, data export/import, support impersonation, audit logs, approval workflows, webhooks, and billing.

**Quick routing — test these first on any B2B SaaS target:**

| Surface | Key Test | Why It Pays |
|---------|----------|-------------|
| SCIM provisioning | Cross-tenant user creation via SCIM token scope bypass | Critical — admin access in wrong tenant |
| SSO/JIT | JIT into wrong tenant via email domain collision | Critical — cross-tenant account creation |
| Invitations | Role escalation on invite acceptance | High — join as admin instead of viewer |
| Seat enforcement | Exceed seat limits via API (UI-only enforcement) | Medium-High — licensing bypass |
| Support impersonation | Impersonate admin user from support agent context | Critical — full ATO at scale |
| Data exports | IDOR on export endpoint (`org_id` swap) | High — bulk data exfiltration |
| Approval workflows | Self-approval or approval bypass via direct API call | High — separation of duties bypassed |
| Webhooks | SSRF via webhook URL (RFC 1918 ranges, cloud metadata) | High — internal network access |
| Billing | Plan confusion — premium features at free-tier price | High — revenue loss |
| Shared links | Scope confusion — admin share link leaks admin-only data to unauthenticated users | High — data exposure at scale |
| SaaS connectors | Cross-tenant connector bleed via unscoped OAuth tokens | Critical — cross-service data exfil |
| Real-time collab | WebSocket auth parity gap — actions blocked on REST succeed via WebSocket | High — auth bypass |

---

## Related Skills

- **vuln-patterns** — Quick reference for all vulnerability classes
- **auth-testing** — BOLA/BFLA/OAuth testing patterns that complement business logic
- **report-writing** — How to frame business logic findings for maximum payout
- **http-desync** — Race conditions and timing attacks in detail
- **workflow-map** (command) — Map B2B SaaS business logic workflows
