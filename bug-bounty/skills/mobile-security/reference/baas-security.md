# BaaS Security & Mobile Platform Threats

Backend-as-a-Service misconfigurations and mobile-specific threats. Reference file for the mobile-security skill.

> **Related files:** [SKILL.md](../SKILL.md) for core mobile testing patterns | [cloud-security](../../cloud-security/SKILL.md) for broader cloud misconfigs | [api-security](../../api-security/SKILL.md) for API patterns

## Table of Contents

- [Firebase Misconfigurations](#firebase-misconfigurations)
- [Supabase Misconfigurations](#supabase-misconfigurations)
- [In-App Purchase Manipulation](#in-app-purchase-manipulation)
- [BaaS Severity Guidelines](#baas-severity-guidelines)
- [Active Mobile Threats (March 2026)](#active-mobile-threats-march-2026)

---

## Firebase Misconfigurations

BaaS misconfigs are a **rapidly growing mobile attack surface** — 20+ data breaches traced to client-side Firebase/Supabase misconfigs since 2025 (Barrack.ai tracker). Mobile apps are the primary vector because API keys and project URLs are embedded in the client.

| # | Pattern | Test | Impact |
|---|---------|------|--------|
| 1 | Open Realtime Database | `curl https://{project-id}.firebaseio.com/.json` — check for data without auth | Full database read (PII, credentials, business data) |
| 2 | Open Firestore | Query collections without auth token via REST API | Document-level data exposure |
| 3 | Storage bucket open | `curl https://firebasestorage.googleapis.com/v0/b/{bucket}/o` | File listing and download |
| 4 | Weak security rules | Rules like `".read": true` or `allow read: if true` | Any user can read all data |
| 5 | Auth-only rules (no owner check) | Rules check `auth != null` but not `auth.uid == resource.data.userId` | Any authenticated user reads all data |
| 6 | Cloud Functions unauthenticated | Call function endpoints without auth headers | Direct function invocation |
| 7 | Service account key in client | Search binary for `"type": "service_account"` | Full project admin access |
| 8 | Firestore rules bypass via REST | Use REST API directly instead of SDK (different rule evaluation) | SDK-enforced rules not applied |

**Finding Firebase project ID:**
- `google-services.json` (Android) or `GoogleService-Info.plist` (iOS)
- JavaScript bundle: search for `firebaseConfig` or `firebaseio.com`
- Network traffic: watch for `firestore.googleapis.com` or `firebaseio.com` calls

### Firebase Testing Workflow

```
1. Extract project ID from client code or network traffic
2. Test Realtime Database: curl https://{project-id}.firebaseio.com/.json
   → 200 with data = Critical (open database)
   → 200 with {} = empty but accessible (still a finding)
   → 401 = rules require auth — try with any auth token
3. Test Firestore: query collections via REST API
4. Test Storage: list bucket contents via googleapis.com
5. Check Cloud Functions: call endpoints from extracted URLs without auth
6. For auth-gated data: create free account, test if rules check ownership
```

### Firebase Severity Escalation

| Access Level | What You Can Do | Severity |
|---|---|---|
| Unauthenticated read | Read all database records without any credentials | Critical |
| Authenticated read (no ownership) | Any logged-in user reads all users' data | High |
| Storage listing | Enumerate and download all files in bucket | High-Critical |
| Service account key | Full admin access to entire Firebase project | Critical |
| Cloud Function invocation | Execute server-side functions without auth | High |

---

## Supabase Misconfigurations

| # | Pattern | Test | Impact |
|---|---------|------|--------|
| 1 | Exposed anon key | Extract `supabaseUrl` and `supabaseAnonKey` from client | Direct API access with anonymous role |
| 2 | Service role key in client | Search for `service_role` key (should never be in client) | Full database admin access |
| 3 | Missing RLS policies | Query tables via `supabase.from('table').select('*')` with anon key | Read all rows without auth |
| 4 | RLS bypass via direct SQL | Use `rpc()` to call functions that bypass RLS | Circumvent row-level security |
| 5 | Storage bucket public | List and download from public storage buckets | File exposure |
| 6 | Auth bypass | Test password reset, magic link, and OAuth flows for bypasses | Account takeover |

**Finding Supabase URLs:**
- Network traffic: `*.supabase.co` requests
- JavaScript/binary: search for `supabase.co`, `SUPABASE_URL`, `SUPABASE_ANON_KEY`

### Supabase-Specific Testing

```
1. Extract supabaseUrl and anon key from client code
2. Test RLS: query each table with anon key
   → Data returned without auth = Critical (missing RLS)
3. Test service_role key exposure: search binary for "service_role"
   → Found = Critical (full admin access)
4. Test RPC functions: call stored procedures that may bypass RLS
5. Test storage: list public buckets and download files
6. Test auth: password reset, magic link, OAuth flows
```

---

## In-App Purchase Manipulation

| # | Pattern | Test | Impact |
|---|---------|------|--------|
| 1 | Receipt validation client-side only | Intercept purchase verification — modify receipt or skip validation | Free premium features |
| 2 | Server-side receipt validation bypass | Replay valid receipts, use sandbox receipts in production | Unauthorized purchases |
| 3 | Subscription status check bypass | Modify `isSubscribed` response from API | Free premium access |
| 4 | Consumable item duplication | Race condition on consuming in-app items | Unlimited consumables |
| 5 | Price tier manipulation | Change product ID to cheaper tier at purchase time | Discounted premium features |

### IAP Testing Workflow

```
1. Intercept purchase flow — identify where validation happens
2. Client-side only: modify purchase success callback or stored receipt
3. Server-validated: replay valid receipts, test sandbox-in-production
4. Subscription checks: intercept and modify API response for subscription status
5. Race condition: send multiple consume requests simultaneously
6. Cross-platform: does Android receipt work on iOS API or vice versa?
```

### Platform-Specific IAP Notes

**Apple StoreKit 2:**
- JWS-signed transactions — harder to forge but test server-side validation
- App Store Server API v2 — verify the app validates against Apple's API, not just locally
- Sandbox vs production environment separation — test if sandbox receipts accepted in prod

**Google Play Billing v6+:**
- Signed purchase tokens — test if signature verification is implemented
- Voided purchases API — test if app checks for refunded/voided purchases
- Promo codes — test for abuse in promo code redemption flow

---

## BaaS Severity Guidelines

| Finding | Typical Severity | Notes |
|---------|-----------------|-------|
| Open Firebase database with PII | Critical | Report immediately — active data exposure |
| Supabase service_role key in client | Critical | Full admin access to database |
| Open storage bucket with user files | High-Critical | Depends on file sensitivity |
| Missing RLS with authenticated access | High | Any user reads all data |
| In-app purchase bypass (premium features) | Medium-High | Financial impact to vendor |
| Client-side subscription check bypass | Medium | Limited to the device |

---

## Active Mobile Threats (March 2026)

### Qualcomm Android Zero-Day (CVE-2026-21385, CVSS 7.8)

- Integer overflow/wraparound in graphics/display component during memory allocation alignment
- Affects **230+ Qualcomm chipset models** — enormous attack surface
- Limited, targeted exploitation confirmed by Google (likely spyware/nation-state attribution)
- Added to CISA KEV March 3, 2026; patched in Google's March 2026 Android update (129 total vulnerabilities)
- **Hunting relevance**: test for graphics subsystem memory corruption patterns; chipset-level bugs affect all apps on the device

### Google March 2026 Android Update

- 129 vulnerabilities patched including critical System component RCE (CVE-2026-0006) requiring no privileges or user interaction
- Signal for hunters: large patch cycles often contain less-publicized bugs in the same components worth investigating

### iOS Threat Landscape

- Apple CVE-2026-20700: iOS zero-day exploited in targeted attacks (Feb 2026)
- Regular cadence of WebKit/Safari vulnerabilities — test WebView implementations against latest patches
- App Tracking Transparency (ATT) bypass patterns emerging — fingerprinting via device capabilities

---

## Common BaaS Report Mistakes

| Mistake | Why It Gets Downgraded |
|---------|----------------------|
| Reporting Firebase API key exposure alone | Firebase API keys are designed to be public — only a finding if paired with open security rules |
| Missing RLS claimed without proof | Must show actual data returned, not just the absence of RLS configuration |
| Supabase anon key as "leaked credential" | Anon keys are meant to be in client code — finding is the missing RLS policy |
| IAP bypass on free app | No financial impact if app is free — only reportable for paid apps/subscriptions |
| Client-side receipt validation "vulnerability" | Many programs accept this as Low/Info — escalate by showing financial impact |
