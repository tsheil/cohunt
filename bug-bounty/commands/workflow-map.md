---
name: workflow-map
argument-hint: "[feature or workflow to map]"
description: Map a feature's business logic — actors, states, invariants, and abuse cases. Business logic = 45% of bounty awards and the strongest human edge over AI scanners
arguments:
  - name: feature
    description: "The feature or workflow to map (e.g., 'Shopify checkout flow', 'Stripe Connect onboarding', 'team invite system')"
    required: true
---

# /workflow-map

Map a feature's business logic into actors, states, invariants, and abuse cases. This is where the money is — business logic flaws pay 45% of all bounty awards (Intigriti 2026) and autonomous tools can't find them because each app's logic is unique.

## Usage

```
/workflow-map <feature or workflow>
```

Mapping workflow: $ARGUMENTS

---

## Step 1: Identify Actors

List every entity that interacts with this feature. Don't just think "user" — think roles, permissions, and trust levels.

```
| Actor | Trust Level | What They Can Do | What They Shouldn't Be Able To Do |
|-------|------------|------------------|-----------------------------------|
| Unauthenticated visitor | None | View public pages | Access any user data |
| Free user | Low | Use basic features | Access premium features |
| Paid user | Medium | Full feature access | Modify other users' data |
| Team admin | High | Manage team settings | Access other orgs |
| API key (service) | Varies | Depends on scope | Exceed granted scope |
| Webhook callback | External | Trigger state changes | Forge signatures |
| Background worker | System | Process queued tasks | Skip validation |
```

**For every pair of actors, ask:** "Can Actor A perform Actor B's actions?"

---

## Step 2: Map the State Machine

Trace the feature through its lifecycle. Every workflow is a state machine — find the states and transitions.

```
[State A] ---(action/trigger)---> [State B] ---(action/trigger)---> [State C]
```

### Output Format

```
## State Machine: [Feature Name]

States:
1. [Initial] - Description, what data exists
2. [Pending] - Description, what changed
3. [Active] - Description, what's unlocked
4. [Completed] - Description, what's finalized

Transitions:
- Initial → Pending: [who can trigger, what's validated]
- Pending → Active: [who can trigger, what's validated]
- Active → Completed: [who can trigger, what's validated]

Side Effects:
- At Pending: [email sent, webhook fired, balance reserved]
- At Active: [access granted, timer started]
- At Completed: [payment captured, record finalized]
```

---

## Step 3: Define Invariants

Invariants are rules that must ALWAYS hold true. Every broken invariant is a potential vulnerability.

```
## Invariants

| # | Invariant | Why It Matters |
|---|-----------|---------------|
| 1 | Only the account owner can change their email | Email change = account takeover |
| 2 | Order total must equal sum of (item prices × quantities) | Price manipulation |
| 3 | A user cannot approve their own request | Authorization bypass |
| 4 | Deleted resources cannot be accessed | Data persistence after deletion |
| 5 | Refund amount cannot exceed original payment | Financial loss |
| 6 | Rate limits apply equally regardless of request method | Limit bypass |
| 7 | Tenant A's data is never visible to Tenant B | Cross-tenant leakage |
```

**Key question for each invariant:** "What happens if I can violate this?"

---

## Step 4: Generate Abuse Cases

For each state transition, generate specific abuse cases. These become your test plan.

### Transition Abuse

| Transition | Abuse Case | Test Method |
|-----------|-----------|-------------|
| Skip state | Jump from Cart → Confirmation (skip Payment) | Call confirmation endpoint directly |
| Reverse state | Revert from Shipped → Cart, modify items | Replay earlier-state API call |
| Parallel state | Be in Checkout in two sessions, complete both | Concurrent requests |
| Stale state | Start flow, wait for timeout, then complete | Delayed request replay |
| Actor swap | User B completes User A's pending action | Swap session tokens mid-flow |
| Parameter tamper | Modify locked fields during transition | Edit request body |

### Cross-Actor Abuse

| Attacker Action | Expected Result | Vulnerable If |
|----------------|----------------|---------------|
| Free user calls premium endpoint | 403 Forbidden | Returns 200 with data |
| User A accesses User B's resource | 404 or 403 | Returns User B's data |
| Expired token used after logout | 401 Unauthorized | Token still works |
| Webhook with forged signature | Rejected | State change occurs |
| API key used outside granted scope | 403 | Action succeeds |

### Invariant Violation Tests

For each invariant from Step 3, write a specific test:

```
Invariant: "Refund amount cannot exceed original payment"
Test: POST /api/refunds {"order_id": "X", "amount": 999999}
Expected: Rejected (amount > original)
Vulnerable if: Refund created for arbitrary amount
Impact: Direct financial loss of $[amount - original]
```

---

## Step 5: Proof Checklist

Before reporting any finding, verify you have evidence for each:

```
## Proof Requirements

□ Cross-boundary impact demonstrated (affects someone other than attacker)
□ Two-account proof captured (attacker account + victim account)
□ Full chain completed (not just "I sent a request" but "here's what happened")
□ Server-side confirmation (not just client-side response manipulation)
□ Financial or security impact quantified with real numbers
□ Reproducible in clean environment (not dependent on cached state)
□ Screenshots/recordings at each step of the chain
```

---

## Output Format

Produce a complete workflow map in this structure:

```markdown
# Workflow Map: [Feature Name]

**Target:** [domain/program]
**Feature:** [what was mapped]
**Date:** [timestamp]

## Actors
[Actor table from Step 1]

## State Machine
[State diagram and transitions from Step 2]

## Invariants
[Numbered invariant list from Step 3]

## Abuse Cases (Prioritized)

### High Priority (Test First)
[Abuse cases most likely to yield findings, with exact test steps]

### Medium Priority
[Abuse cases worth testing if time allows]

### Exploratory
[Long-shot abuse cases for comprehensive coverage]

## Test Plan
[Concrete curl commands / test steps for top 3 abuse cases]

## Chain Opportunities
[How findings from this workflow could chain with other features]
```

---

## When to Use This Command

| Situation | Use /workflow-map |
|-----------|------------------|
| Target has payment/checkout flow | Map the full purchase lifecycle |
| Target has invite/onboarding system | Map actor creation and role assignment |
| Target has approval workflows | Map request → review → execute chain |
| Target has subscription/billing | Map plan changes, upgrades, refunds |
| Target has multi-tenant architecture | Map isolation boundaries |
| Target has API with state-changing operations | Map the API's state machine |

---

## Tips

1. **Start from the money** — If the feature handles payments, subscriptions, or credits, that's your highest-value target
2. **Two accounts minimum** — Always test with at least two accounts at different permission levels
3. **Map before testing** — Resist the urge to start fuzzing. 20 minutes mapping saves hours of unfocused testing
4. **Check the happy path first** — Understand normal behavior before trying to break it
5. **Race conditions at boundaries** — Every state transition is a potential race condition target
6. **Chain with auth findings** — Business logic bugs combined with IDOR/BFLA are almost always Critical severity
