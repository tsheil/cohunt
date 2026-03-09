# JavaScript Analysis for Bug Bounty Recon

Deep-dive into client-side JavaScript analysis — the most underutilized recon technique that consistently yields $5K-$25K+ payouts. Every modern SPA ships its attack surface in the browser.

## Table of Contents

- [Why JavaScript Recon Pays](#why-javascript-recon-pays)
- [Step 1: JavaScript File Discovery](#step-1-javascript-file-discovery)
- [Step 2: Source Map Exploitation](#step-2-source-map-exploitation)
- [Step 3: Secret & Credential Hunting](#step-3-secret--credential-hunting)
- [Step 4: API Endpoint Extraction](#step-4-api-endpoint-extraction)
- [Step 5: Webpack/Build Analysis](#step-5-webpackbuild-analysis)
- [Step 6: Framework-Specific Techniques](#step-6-framework-specific-techniques)
- [Automation Workflow](#automation-workflow)
- [Severity & Reporting](#severity--reporting)

---

## Why JavaScript Recon Pays

| Finding Source | Example Payout | What Was Found |
|---------------|---------------|----------------|
| Hidden API endpoint in obfuscated JS | $25,000 | `/api/v2/internal/users` — complete user database access |
| Unrestricted Firebase database URL | $5,000 | Full read/write access to production database |
| Live Stripe secret key in bundle | $4,000 | Payment processing credentials |
| AWS credentials in config object | $3,000 | Administrative access to cloud infrastructure |
| Internal admin panel route in SPA | $2,000-$8,000 | Undocumented admin functionality |

---

## Step 1: JavaScript File Discovery

### Extraction Methods

| # | Method | Command/Technique | What It Finds |
|---|--------|------------------|---------------|
| 1 | Page source crawl | `curl -s https://target.com \| grep -oP 'src="[^"]*\.js"'` | Directly referenced scripts |
| 2 | Recursive crawl | Spider all pages, collect unique JS URLs | Scripts loaded on specific routes |
| 3 | Source map detection | Check for `.js.map` files (append `.map` to any JS URL) | Original unminified source code |
| 4 | Wayback Machine | `curl "https://web.archive.org/cdx/search/cdx?url=target.com/*.js&output=json"` | Historical JS files (may contain removed secrets) |
| 5 | Certificate transparency | Subdomain enumeration → check each for JS assets | JS from staging/dev environments |
| 6 | Webpack chunk discovery | Look for `webpackChunkXXX` patterns, enumerate chunk IDs | Lazy-loaded modules with hidden functionality |
| 7 | Service worker | Check `navigator.serviceWorker` registration, `/sw.js` | Cached API routes, offline endpoints |
| 8 | Inline scripts | Extract `<script>` tag contents from HTML | Configuration objects, initialization data |

### Tools

| Tool | Purpose | Usage |
|------|---------|-------|
| **LinkFinder** | Extract endpoints from JS files | `python3 linkfinder.py -i https://target.com/app.js -o cli` |
| **SecretFinder** | Find API keys, tokens, credentials in JS | `python3 SecretFinder.py -i https://target.com/app.js -o cli` |
| **GetJS** | Extract all JS URLs from a target | `echo "target.com" \| getjs` |
| **SubJS** | JavaScript file discovery from subdomains | `echo "target.com" \| subjs` |
| **Katana** | Modern web crawler with JS rendering | `katana -u https://target.com -jc` |
| **Gospider** | Fast web spider with JS parsing | `gospider -s https://target.com --js` |
| **De4JS** | JavaScript deobfuscation | Online tool or local: unpack, unminify, deobfuscate |
| **Gau** | Fetch URLs from Wayback/OTX/Common Crawl | `gau target.com --subs \| grep "\.js$"` |

---

## Step 2: Source Map Exploitation

Source maps (`.js.map`) expose the **original, unminified source code** including comments, variable names, and file structure. This is the single highest-value JavaScript recon technique.

### Detection

```
For each JavaScript file:
1. Append .map → curl -sI https://target.com/app.js.map (check for 200)
2. Check last line of JS file for: //# sourceMappingURL=
3. Check response headers for: SourceMap: or X-SourceMap:
4. Try common patterns: /sourcemaps/, /maps/, /.map/
```

### Exploitation

```
When source maps are found:
1. Download the .map file
2. Use source-map-explorer or unwebpack-sourcemap to reconstruct source
3. Search reconstructed code for:
   - API endpoints and internal URLs
   - Authentication logic and token handling
   - Role-based access control implementation
   - Hidden routes and admin functionality
   - Environment configuration (API keys, service URLs)
   - Comments revealing business logic assumptions
```

**Impact framing:** Source maps in production expose the full application architecture. Report as information disclosure (Medium-High) and chain with any findings from the exposed source.

---

## Step 3: Secret & Credential Hunting

### High-Value Patterns

| # | Secret Type | Regex Pattern | Context |
|---|-------------|--------------|---------|
| 1 | AWS Access Key | `AKIA[0-9A-Z]{16}` | AWS IAM credentials |
| 2 | AWS Secret Key | `[A-Za-z0-9/+=]{40}` (near AKIA) | Pair with access key |
| 3 | Google API Key | `AIza[0-9A-Za-z-_]{35}` | GCP services |
| 4 | Firebase URL | `https://[a-z0-9-]+\.firebaseio\.com` | Direct database access |
| 5 | Stripe Secret Key | `sk_live_[0-9a-zA-Z]{24,}` | Payment processing |
| 6 | Stripe Publishable Key | `pk_live_[0-9a-zA-Z]{24,}` | Often leads to finding secret key |
| 7 | GitHub Token | `gh[pousr]_[A-Za-z0-9_]{36,}` | Repository access |
| 8 | Slack Token | `xox[baprs]-[0-9a-zA-Z-]+` | Workspace access |
| 9 | JWT Token | `eyJ[A-Za-z0-9_-]+\.eyJ[A-Za-z0-9_-]+` | Decode for user data, roles |
| 10 | Private Key | `-----BEGIN (RSA\|EC\|OPENSSH) PRIVATE KEY-----` | Cryptographic material |
| 11 | Generic API Key | `[Aa]pi[_-]?[Kk]ey.*['"][A-Za-z0-9_-]{20,}['"]` | Various services |
| 12 | Internal URL | `https?://[a-z0-9.-]*\.(internal\|local\|corp\|dev\|staging)[^'"]*` | Internal infrastructure |
| 13 | Supabase URL | `https://[a-z0-9]+\.supabase\.co` | Database/auth access |
| 14 | Supabase Anon Key | `eyJ[A-Za-z0-9_-]+\.eyJ[A-Za-z0-9_-]+` (near supabase) | Check for service_role key exposure |

### Validation Steps

```
For each discovered credential:
1. Verify it's not a test/placeholder value
2. Check if it's still active (make a benign API call)
3. Determine scope of access (what can it do?)
4. Document in report with redacted values
5. Do NOT access production data — prove access exists, stop there
```

---

## Step 4: API Endpoint Extraction

### What to Extract

| Source | Method | What to Look For |
|--------|--------|-----------------|
| Fetch/XHR calls | Search for `fetch(`, `axios.`, `$.ajax`, `XMLHttpRequest` | API base URLs, endpoint paths |
| Route definitions | Search for `router.`, `Route`, `path:`, `createBrowserRouter` | All application routes including hidden ones |
| GraphQL queries | Search for `query`, `mutation`, `gql` tagged templates | Schema information, available operations |
| WebSocket URLs | Search for `wss://`, `ws://`, `new WebSocket` | Real-time endpoints |
| Configuration objects | Search for `config`, `settings`, `env`, `API_URL`, `BASE_URL` | Environment-specific URLs |
| Error handling | Search for `.catch`, `onError`, `error_url` | Error reporting endpoints, debug URLs |

### SPA Route Extraction

Modern SPAs define all routes client-side. Extract them to find:
- **Admin routes**: `/admin`, `/dashboard`, `/manage`, `/internal`
- **API documentation**: `/docs`, `/swagger`, `/graphql`, `/playground`
- **Debug routes**: `/debug`, `/test`, `/dev`, `/health`
- **Feature flags**: Routes behind feature toggles that may be accessible

```
Common framework patterns:
- React Router: <Route path="/admin" component={AdminPanel} />
- Vue Router: { path: '/admin', component: () => import('./Admin.vue') }
- Angular: { path: 'admin', loadChildren: () => import('./admin/admin.module') }
- Next.js: Check /pages/ or /app/ directory structure in source maps
```

---

## Step 5: Webpack/Build Analysis

### Chunk Enumeration

```
Modern apps use code splitting. Enumerate chunks:
1. Find the webpack runtime (usually main.js or runtime.js)
2. Look for chunk loading patterns: __webpack_require__.e(chunkId)
3. Extract the chunk manifest (mapping of chunk IDs to filenames)
4. Download all chunks — some may contain admin-only code
5. Look for webpackChunkXXX global variable assignments
```

### Environment Leakage

```
Build tools often embed environment variables:
- process.env.REACT_APP_* (Create React App)
- import.meta.env.VITE_* (Vite)
- __NEXT_DATA__ (Next.js — check for server-side props leaking data)
- window.__ENV__ or window.__CONFIG__ (custom injection patterns)
```

**Next.js specific:** Check `/_next/data/` endpoint for server-side data that may not be intended for public access. Also check `__NEXT_DATA__` in page source for leaked props.

---

## Step 6: Framework-Specific Techniques

### React Applications

| # | Technique | What to Look For |
|---|-----------|-----------------|
| 1 | React DevTools detection | `__REACT_DEVTOOLS_GLOBAL_HOOK__` — if present, can inspect component tree |
| 2 | Redux state exposure | `window.__REDUX_DEVTOOLS_EXTENSION__`, `window.__PRELOADED_STATE__` |
| 3 | dangerouslySetInnerHTML | User-controlled content → XSS |
| 4 | Client-side routing | All routes visible, including unlinked admin routes |
| 5 | Apollo Client cache | GraphQL query cache may contain sensitive data |

### Angular Applications

| # | Technique | What to Look For |
|---|-----------|-----------------|
| 1 | ng.probe() | Debug access to component tree and state |
| 2 | Route guards | Client-side auth guards can be bypassed — check API directly |
| 3 | Environment files | `environment.prod.ts` often contains API URLs, feature flags |
| 4 | HTTP interceptors | Auth token handling logic, retry patterns |

### Vue Applications

| # | Technique | What to Look For |
|---|-----------|-----------------|
| 1 | Vue DevTools | `__VUE_DEVTOOLS_GLOBAL_HOOK__` — component inspection |
| 2 | Vuex store | `window.__VUEX_STATE__`, `$store.state` — application state |
| 3 | Route meta fields | `meta: { requiresAuth: true }` — client-side only guards |
| 4 | Pinia stores | Similar to Vuex but in Pinia format |

---

## Automation Workflow

### Quick JS Recon (5 minutes)

```
1. curl the target → extract all <script> tags
2. For each JS file → check for .map file
3. Run SecretFinder on each JS file
4. Run LinkFinder on each JS file
5. Check for __NEXT_DATA__, window.__CONFIG__, etc. in page source
```

### Deep JS Recon (30 minutes)

```
1. Quick recon steps above
2. Download all JS files locally
3. Deobfuscate/beautify with js-beautify or De4JS
4. Search for all regex patterns from Step 3
5. Extract and test all discovered API endpoints
6. Enumerate webpack chunks for hidden modules
7. Check Wayback Machine for historical JS files
8. Compare current vs. historical — secrets may have been removed
```

### Continuous JS Monitoring

```
For long-term program engagement:
1. Hash all current JS files
2. Periodically check for changes (new deployments)
3. Diff changed files — look for new endpoints, removed secrets
4. New JS files may indicate new features (fresh attack surface)
```

---

## Severity & Reporting

| Finding | Typical Severity | Reporting Notes |
|---------|-----------------|-----------------|
| Source maps exposing full source code | Medium | Information disclosure; chain with source code findings |
| Active AWS/GCP credentials | Critical | Prove access scope, report immediately |
| Firebase database with read/write | High-Critical | Depends on data sensitivity |
| Internal API endpoints accessible | Medium-High | Test for auth bypass, IDOR |
| Hardcoded admin credentials | Critical | If active and reachable |
| Stripe secret keys | High-Critical | Payment processing compromise |
| Hidden admin routes (client-side guard only) | Medium-High | Bypass guard, access admin API directly |
| Historical secrets from Wayback Machine | Varies | Check if still active; if rotated, lower severity |
