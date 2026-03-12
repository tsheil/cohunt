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
| 8 | Firestore REST as alternate client surface | Test REST API directly (same Security Rules apply, but different client behavior may reveal misconfigs) | Rules still enforced; IAM-auth only bypasses with service account |

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

### Firestore REST API Testing (often missed)

Many hunters only test Realtime Database. Firestore is a separate product with its own REST API.

```bash
# 1. List documents in a collection (requires knowing collection name)
# Common collection names: users, profiles, orders, messages, payments, settings
curl "https://firestore.googleapis.com/v1/projects/{PROJECT_ID}/databases/(default)/documents/users"

# 2. If auth required, get an ID token first:
# Anonymous sign-up (only works if anonymous auth is enabled in project):
curl "https://identitytoolkit.googleapis.com/v1/accounts:signUp?key={API_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"returnSecureToken": true}'
# If anonymous auth is disabled, create a test account via email/password sign-up instead.
# Extract idToken from response

# 3. Query Firestore with auth token:
curl "https://firestore.googleapis.com/v1/projects/{PROJECT_ID}/databases/(default)/documents/users" \
  -H "Authorization: Bearer {ID_TOKEN}"

# 4. Test structured query (bypass simple rules):
curl "https://firestore.googleapis.com/v1/projects/{PROJECT_ID}/databases/(default)/documents:runQuery" \
  -H "Authorization: Bearer {ID_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{"structuredQuery": {"from": [{"collectionId": "users"}], "limit": 10}}'
```

### Firebase Storage Enumeration

```bash
# 1. List all files in default bucket
# New projects (after Oct 2024) use: {PROJECT_ID}.firebasestorage.app
# Older projects use: {PROJECT_ID}.appspot.com
# Try both:
curl "https://firebasestorage.googleapis.com/v0/b/{PROJECT_ID}.firebasestorage.app/o"
curl "https://firebasestorage.googleapis.com/v0/b/{PROJECT_ID}.appspot.com/o"

# 2. Download a specific file
curl "https://firebasestorage.googleapis.com/v0/b/{BUCKET}/o/{FILE_PATH}?alt=media"

# 3. Common high-value paths to check:
# - /uploads/ (user uploads — may contain ID documents, photos)
# - /backups/ (database backups)
# - /exports/ (data exports)
# - /profile_pictures/ (PII indicator)
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
| 4 | RPC functions as RLS bypass | Call `rpc()` functions defined as `SECURITY DEFINER` under a `bypassrls` owner — these skip RLS | Circumvent row-level security (only if function is misconfigured) |
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

### Supabase RLS Deep Testing

RLS (Row Level Security) is Supabase's primary access control. Many developers enable it but write weak policies.

```bash
# 1. Discover tables — there is no built-in RPC for this.
# Instead, extract table names from client code or network traffic (see discovery section below).

# 2. Test each table for missing RLS
curl "https://{PROJECT}.supabase.co/rest/v1/{TABLE}?select=*&limit=5" \
  -H "apikey: {ANON_KEY}" \
  -H "Authorization: Bearer {ANON_KEY}"
# Common tables: profiles, users, orders, messages, payments, subscriptions

# 3. Test with authenticated user — check for ownership checks
# Sign up first, then query other users' data:
curl "https://{PROJECT}.supabase.co/rest/v1/profiles?select=*&id=neq.{YOUR_ID}" \
  -H "apikey: {ANON_KEY}" \
  -H "Authorization: Bearer {USER_JWT}"
# If you get other users' profiles → RLS policy checks auth but not ownership

# 4. Test RPC functions (bypass RLS only if defined as SECURITY DEFINER with bypassrls owner):
# Most functions run as SECURITY INVOKER and respect RLS. Check for misconfigured ones:
curl "https://{PROJECT}.supabase.co/rest/v1/rpc/{FUNCTION_NAME}" \
  -H "apikey: {ANON_KEY}" \
  -H "Content-Type: application/json" \
  -d '{"param": "value"}'

# 5. Test storage buckets
curl "https://{PROJECT}.supabase.co/storage/v1/object/list/{BUCKET}" \
  -H "apikey: {ANON_KEY}" \
  -H "Authorization: Bearer {ANON_KEY}"
```

### Supabase Table Discovery Techniques

When you don't know table names:
1. **Client code analysis** — search decompiled code for `.from('` or `supabase.from(`
2. **Network traffic** — watch for `/rest/v1/{table}` requests
3. **Common names** — try: users, profiles, accounts, orders, products, messages, notifications, payments, subscriptions, settings, teams, organizations
4. **Error messages** — invalid table names sometimes reveal valid ones in error responses

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

**Google Play Billing (v7+/v8):**
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

## Worked Example: Firebase Open Database → Critical PII Exposure

**Scenario:** Fitness tracking app with Firebase backend. Discovered via mobile APK analysis.

**Step 1 — Extract Firebase config from APK:**
```bash
jadx -d output fitness.apk
grep -rn "firebaseio.com\|firebase.*project" output/
# Found in google-services.json:
# "project_id": "fitness-tracker-prod",
# "firebase_url": "https://fitness-tracker-prod.firebaseio.com"
```

**Step 2 — Test Realtime Database access:**
```bash
curl "https://fitness-tracker-prod.firebaseio.com/.json"
# Response: 200 OK
# Returns: {"users": {"uid1": {"email": "user@example.com", "weight": 180,
#   "health_data": {...}, "location_history": [...]}}, ...}
# → Full database dump — emails, health data, GPS locations for ALL users
```

**Step 3 — Assess scope:**
```bash
# Count records (estimate from response size)
curl -s "https://fitness-tracker-prod.firebaseio.com/users.json?shallow=true" | python3 -c "import json,sys; print(len(json.load(sys.stdin)))"
# Output: 47832 (users affected)
```

**Finding:** Firebase Realtime Database with `.read: true` security rules exposes 47,832 user records including emails, health data, and GPS location history. No authentication required. **Severity: Critical.** Health data may create regulatory exposure depending on jurisdiction and business model (GDPR in EU; HIPAA in US only if operator is a covered entity/business associate).

**Report note:** Include the `curl` command and a redacted sample of 2-3 records as proof. Do NOT download the full database — demonstrate access, estimate scope, and report immediately.

**Context:** A similar Firebase misconfiguration in an AI chat app exposed 300M messages from 25M users in January 2026 (Hackread). Researchers found 103 of 200 tested iOS apps had the same class of misconfiguration.

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
