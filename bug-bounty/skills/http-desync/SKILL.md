---
name: http-desync
description: HTTP request smuggling, web cache poisoning, and race condition testing patterns. High-value bug classes that require understanding protocol-level behavior. Trigger with "request smuggling", "cache poisoning", "race condition testing", "HTTP desync", "web cache deception", "CL.TE", "TE.CL", "how do I test for smuggling", "cache key analysis".
---

# HTTP Desync & Cache Attacks

Protocol-level and timing-based vulnerability classes that consistently pay high bounties. These bugs exploit how infrastructure components — proxies, caches, load balancers — interpret HTTP differently from the origin server.

## How It Works

```
┌──────────────────────────────────────────────────────────────┐
│                    HTTP DESYNC & CACHE                        │
├──────────────────────────────────────────────────────────────┤
│  ALWAYS (works standalone)                                   │
│  ✓ HTTP request smuggling: CL.TE, TE.CL, TE.TE patterns     │
│  ✓ Web cache poisoning: unkeyed inputs, cache key analysis   │
│  ✓ Web cache deception: path confusion, extension tricks     │
│  ✓ Race conditions: TOCTOU, limit overrun, state collision   │
│  ✓ HTTP/2 desync: H2.CL, H2.TE, CRLF in H2 headers         │
│  ✓ Response queue poisoning                                  │
├──────────────────────────────────────────────────────────────┤
│  SUPERCHARGED (when you connect your tools)                  │
│  + Web scanner: automated smuggling/cache detection          │
│  + Asset discovery: identify CDN/proxy layer for targeting   │
└──────────────────────────────────────────────────────────────┘
```

---

## Getting Started

Ask about any of these attack classes:

- "How do I test for request smuggling on this target?"
- "This target uses Cloudflare + nginx — what cache poisoning is possible?"
- "Race condition patterns for a coupon redemption endpoint"
- "Can I do HTTP/2 desync against this target?"
- "Web cache deception test cases for an API behind Akamai"

I'll give you concrete test cases tailored to the target's infrastructure when I know it.

---

## HTTP Request Smuggling

### What It Is

Front-end (proxy/load balancer) and back-end (origin) servers disagree on where one HTTP request ends and the next begins. This lets an attacker "smuggle" a request that the front-end doesn't see but the back-end processes.

### Architecture Prerequisite

```
Client → [Front-end: proxy/LB/CDN] → [Back-end: origin server]
```

Smuggling only works when there's a proxy or load balancer in front of the origin. Single-server setups aren't vulnerable to classic smuggling (but may be vulnerable to other desync attacks).

### Variants

| Variant | How It Works | Detection |
|---------|-------------|-----------|
| **CL.TE** | Front-end uses Content-Length, back-end uses Transfer-Encoding | Send CL that's shorter than body + TE chunked trailer |
| **TE.CL** | Front-end uses Transfer-Encoding, back-end uses Content-Length | Send TE chunked with CL that includes smuggled prefix |
| **TE.TE** | Both use TE but parse obfuscated TE headers differently | Obfuscate `Transfer-Encoding` header to confuse one side |
| **H2.CL** | HTTP/2 front-end, HTTP/1.1 back-end using CL | Inject `Content-Length` header in HTTP/2 request |
| **H2.TE** | HTTP/2 front-end, HTTP/1.1 back-end using TE | Inject `Transfer-Encoding` in HTTP/2 request |
| **CL.0** | Back-end ignores body on certain endpoints | Send request with body to endpoint that doesn't expect one |

### Test Patterns

| # | Test | Method | What to look for |
|---|------|--------|-----------------|
| 1 | CL.TE detection | Send `CL: short` + chunked body ending with `0\r\n\r\nSMUGGLED` | Timeout on follow-up request (smuggled prefix captured) |
| 2 | TE.CL detection | Send chunked body + `CL` that includes extra data | Timeout or error on follow-up |
| 3 | TE obfuscation | Vary `Transfer-Encoding` header: `Transfer-Encoding: xchunked`, `Transfer-Encoding : chunked`, `Transfer-Encoding: chunked\r\nTransfer-encoding: x` | Different behavior per obfuscation |
| 4 | H2 smuggling | Downgrade HTTP/2 → HTTP/1.1 with injected headers | CRLF injection via HTTP/2 pseudo-headers |
| 5 | CL.0 detection | Send POST with body to GET-like endpoint | Body treated as next request |
| 6 | Confirm smuggling | Smuggle a prefix that changes the next request (e.g., prepend `GET /404 HTTP/1.1\r\n`) | Next legitimate request gets 404 |
| 7 | Capture other users' requests | Smuggle a request that stores the next request's headers (via reflected parameter) | See another user's cookies/headers |
| 8 | Bypass front-end controls | Smuggle request to admin endpoint that the proxy blocks | Access to restricted paths |

### Safe Testing Tips

- **Use time-based detection first** — smuggle a request that triggers a timeout, not one that affects other users
- **Test on staging if available** — smuggling can disrupt other users on production
- **Start with detection, not exploitation** — confirm the desync exists before attempting impactful smuggling
- **Document the proxy stack** — knowing the exact proxy and origin software helps choose the right variant

---

## Web Cache Poisoning

### What It Is

Injecting malicious content into cached responses by manipulating inputs that the cache ignores (unkeyed inputs) but the origin processes. Subsequent users who hit the same cache entry get the poisoned response.

### Architecture Prerequisite

```
Client → [Cache: CDN/proxy cache] → [Origin server]
```

The cache decides what to store based on a "cache key" (typically: method, host, path, some query params). Anything NOT in the cache key is an "unkeyed input" — and a potential poisoning vector.

### Finding Unkeyed Inputs

| Input | Where | Detection method |
|-------|-------|-----------------|
| **Headers** | `X-Forwarded-Host`, `X-Original-URL`, `X-Rewrite-URL` | Send custom header → check if reflected in cached response |
| **Cookies** | Session-independent cookies affecting response | Send cookie → check if response changes for cached resource |
| **Query params** | Params stripped by cache but processed by origin | `?utm_content=<payload>` → check if reflected |
| **Port** | Port in Host header | `Host: example.com:1234` → reflected in redirects/links |
| **Scheme** | `X-Forwarded-Scheme`, `X-Forwarded-Proto` | Force HTTP scheme → check for redirect poisoning |

### Test Patterns

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Header reflection | Add `X-Forwarded-Host: evil.com` to request | `evil.com` in response (scripts, links, redirects) |
| 2 | Cache key analysis | Use `Pragma: x-get-cache-key` or similar headers | Cache key structure revealed |
| 3 | Parameter poisoning | Add unkeyed param: `?cachebuster=1&utm_content=<svg/onload=alert(1)>` | Payload cached and served to others |
| 4 | Fat GET | Send GET with body that affects response | Body not in cache key but alters response |
| 5 | Normalized path | `/path/../target` vs `/target` — different to cache, same to origin? | Different cache entries for same resource |
| 6 | Port poisoning | `Host: example.com:evil` | Incorrect port in cached redirects |
| 7 | Method override | `X-HTTP-Method-Override: POST` on GET request | POST behavior served from GET cache |
| 8 | Vary header abuse | Find responses without proper Vary headers | Same cache entry serves different content |

### CDN-Specific Notes

| CDN | Behavior to check |
|-----|-------------------|
| **Cloudflare** | Cache-Status header, Transform Rules stripping headers, cf-cache-status |
| **Akamai** | Pragma: akamai-x-check-cacheable, X-Cache headers |
| **Fastly** | X-Cache, Surrogate-Control, vary handling |
| **AWS CloudFront** | X-Cache, default TTL, query string forwarding config |
| **Varnish** | X-Varnish, Age header, VCL configuration clues |

---

## Web Cache Deception

### What It Is

Tricking the cache into storing a private response (like a user's profile page) by making the URL look like a static resource. The attacker then accesses the cached private data.

### How It Differs From Cache Poisoning

- **Cache poisoning:** Attacker puts bad content INTO the cache
- **Cache deception:** Attacker tricks the cache into storing content it shouldn't (private pages)

### Test Patterns

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Path extension | Access `/account/settings.css` (appending static extension) | Private page content cached as static file |
| 2 | Path confusion | Access `/account/settings/nonexistent.js` | Origin serves `/account/settings`, cache treats as static |
| 3 | Encoded separator | `/account/settings%2F.css`, `/account%3B.js` | Parser differential between cache and origin |
| 4 | Dot segment | `/static/../account/settings` | Cache sees static path, origin resolves to private page |
| 5 | Trailing characters | `/account/settings;.js` | Semicolon path parameter ignored by origin, used by cache |

### Exploitation Flow

```
1. Identify a private page (e.g., /account/profile that shows PII)
2. Craft a deceptive URL: /account/profile/x.css
3. Send this URL to the victim (e.g., in a link)
4. Victim visits → origin serves their private page → cache stores it
5. Attacker requests the same URL → gets victim's cached private page
```

---

## Race Conditions

### What It Is

Exploiting timing windows where concurrent requests can bypass checks, exceed limits, or create inconsistent state because the application isn't properly serializing access to shared resources.

### Where to Look

- Any endpoint with rate limiting or quotas
- Payment and transaction processing
- Coupon/promo code redemption
- Account registration (unique email checks)
- Resource allocation (inventory, seats, etc.)
- Multi-step workflows with state transitions
- Feature flag or permission changes

### Test Patterns

| # | Test | Method | What to look for |
|---|------|--------|-----------------|
| 1 | Limit overrun | Send N parallel requests to rate-limited endpoint | More than allowed actions succeed |
| 2 | Double-spend | Two parallel purchase/redeem requests | Both succeed — resource used twice |
| 3 | TOCTOU | Two parallel requests that check-then-act | Both pass the check before either acts |
| 4 | State collision | Parallel requests that modify same state | Inconsistent or corrupted state |
| 5 | Registration race | Two parallel registrations with same email | Both succeed — duplicate accounts |
| 6 | Coupon race | Apply same single-use coupon in parallel | Applied twice |
| 7 | Permission race | Request permission change + privileged action simultaneously | Action succeeds during transition |
| 8 | Balance race | Two parallel withdrawals exceeding balance | Both succeed — negative balance |

### Execution Technique

**Single-packet attack (HTTP/2):**
```
Send multiple requests in a single TCP packet so they arrive
at the server simultaneously, eliminating network jitter:

1. Prepare N requests in a single HTTP/2 connection
2. Send all request headers, holding back the final bytes
3. Release all final bytes at once (single packet)
4. All requests arrive within microseconds of each other
```

**Last-byte sync (HTTP/1.1):**
```
1. Open N connections to the target
2. Send all but the last byte of each request
3. Release all final bytes simultaneously
4. Requests arrive nearly simultaneously
```

### Session-Based Race Conditions

Some race conditions only work within a single session:

| Pattern | Example | Test |
|---------|---------|------|
| **Session overwrite** | Login as user A + login as user B simultaneously | Check if session mixes user data |
| **Cart race** | Add item + checkout simultaneously | Item added after payment calculated |
| **Email verification** | Request email change + verify old email simultaneously | Verify wrong email |

---

## Response Queue Poisoning

### What It Is

When the back-end sends responses in a different order than the front-end expects, causing responses to be served to the wrong requests (and potentially the wrong users).

### Where to Look

- Targets with connection reuse between proxy and origin
- Endpoints with significantly different response times
- HTTP/1.1 keep-alive connections through proxies

### Test Pattern

```
1. Send a request that triggers an unusual response from the back-end
   (e.g., a smuggled request that gets a redirect)
2. The proxy expects the response to match request A,
   but it's actually for the smuggled request B
3. Subsequent responses shift — user C gets response D, etc.
```

---

## Using This Skill

### With Recon Data
If you've already run `target-recon`, I'll tailor attacks to the detected proxy/cache/CDN stack.

### Without Context
I'll give you the full methodology for the attack class you ask about.

### During Hunting
Ask as you go: "I see Cloudflare + nginx in the response headers — what desync attacks should I try?"

---

## Related Skills

- **target-recon** — Identify the proxy/cache/CDN stack before testing desync attacks
- **vuln-patterns** — Broader vulnerability testing patterns beyond HTTP-level attacks
- **report-writing** — Write up smuggling and cache bugs with proper reproduction steps
- **program-research** — Check if the program accepts infrastructure-level bugs
