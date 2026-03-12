---
name: mobile-security
description: Mobile application security testing patterns for iOS and Android. Covers API key extraction, certificate pinning bypass, deep link abuse, local storage analysis, and mobile-specific authentication flaws. Trigger with "mobile app security", "test this iOS app", "Android APK analysis", "mobile API testing", "certificate pinning", "deep link vulnerability", "mobile app pentest", or when program scope includes mobile applications. For mobile API backend bugs (IDOR, auth bypass), use auth-testing or api-security. For web views and browser-based issues, use client-side-security.
---

# Mobile Security

Security testing patterns for mobile applications — iOS and Android. Covers the mobile-specific attack surface that web-only testing misses: client-side secrets, transport security, deep link handlers, local data storage, and mobile API abuse patterns.

## Execution Flow

```
1. Identify platform (iOS, Android, or both) and app type
2. Static analysis — extract secrets, endpoints, and configuration
3. Transport security — test TLS, certificate pinning, traffic interception
4. API security — test mobile-specific API patterns
5. Local storage — check for sensitive data at rest
6. Authentication — test mobile auth flows and biometric implementation
7. Deep links and IPC — test URL handlers and inter-app communication
8. WebView — test JavaScript bridge and content loading security
9. Synthesize findings with mobile-specific impact assessment
```

---

## Static Analysis

### Android (APK)

| # | Pattern | Method |
|---|---------|--------|
| 1 | Decompile APK | `apktool d app.apk` or `jadx -d output app.apk` |
| 2 | Extract strings | `strings app.apk \| grep -i 'api\|key\|secret\|token\|password'` |
| 3 | Find hardcoded URLs | Search decompiled source for `https://`, `http://`, API base URLs |
| 4 | Check AndroidManifest.xml | Exported activities, services, receivers, permissions, debuggable flag |
| 5 | Find API keys | Search for patterns: `AIza`, `AKIA`, `sk-`, `pk_live_`, `ghp_` |
| 6 | Check for debug builds | `android:debuggable="true"` in manifest |
| 7 | Network security config | Check `res/xml/network_security_config.xml` for cleartext traffic or custom CAs |
| 8 | Firebase config | Check `google-services.json` for project IDs, API keys |
| 9 | Third-party SDK secrets | Search for Stripe, Twilio, SendGrid, AWS keys in resources |
| 10 | ProGuard/R8 check | Are class/method names obfuscated? Unobfuscated code is easier to audit |

**Key files to inspect:**
- `AndroidManifest.xml` — permissions, exported components, intent filters
- `res/values/strings.xml` — hardcoded strings
- `res/xml/network_security_config.xml` — transport security policy
- `assets/` — configuration files, embedded databases
- `lib/` — native libraries (may contain secrets)
- `google-services.json` — Firebase/GCP configuration

### iOS (IPA)

| # | Pattern | Method |
|---|---------|--------|
| 1 | Extract IPA | Rename `.ipa` to `.zip`, extract |
| 2 | Inspect binary | `strings AppBinary \| grep -i 'api\|key\|secret\|token'` |
| 3 | Check Info.plist | URL schemes, App Transport Security exceptions, permissions |
| 4 | Entitlements | Check `embedded.mobileprovision` for capabilities |
| 5 | ATS exceptions | Look for `NSAllowsArbitraryLoads`, `NSExceptionDomains` |
| 6 | Keychain groups | Check entitlements for keychain access groups |
| 7 | Custom URL schemes | `CFBundleURLTypes` in Info.plist |
| 8 | Universal links | `apple-app-site-association` file on associated domains |
| 9 | Framework analysis | Check `Frameworks/` for known vulnerable versions |
| 10 | Swift metadata | Class names in binary reveal app architecture |

**Key files to inspect:**
- `Info.plist` — ATS config, URL schemes, permissions
- `embedded.mobileprovision` — entitlements and capabilities
- `Frameworks/` — third-party frameworks and versions
- Binary itself — strings, symbols, class dumps

---

## Transport Security

### Certificate Pinning Bypass

| # | Scenario | Test approach |
|---|----------|---------------|
| 1 | No pinning | Set up proxy (Burp/mitmproxy) with custom CA → traffic visible |
| 2 | System CA pinning | Install custom CA on device → may be enough |
| 3 | Custom pinning (Android) | Check network_security_config.xml or OkHttp CertificatePinner |
| 4 | Custom pinning (iOS) | Check for URLSession delegate, TrustKit, or Alamofire pinning |
| 5 | Pinning in native code | Check native libraries for SSL callback implementations |

**What to do when pinning is in place:**
- Android: Modify `network_security_config.xml` in decompiled APK, rebuild
- iOS: Use Frida/Objection to hook SSL validation functions
- Both: Check if pinning is only on certain endpoints (some APIs may not be pinned)
- Check if pinning implementation has fallback to unpinned for certain conditions

### TLS Configuration

| # | Test | What to look for |
|---|------|-----------------|
| 1 | Minimum TLS version | App connecting over TLS 1.0 or 1.1 |
| 2 | Cleartext traffic | HTTP connections (Android: `cleartextTrafficPermitted`, iOS: ATS exceptions) |
| 3 | Self-signed cert acceptance | Install self-signed cert → does app connect? |
| 4 | Hostname verification | Connect via IP → does cert hostname check pass? |
| 5 | Proxy detection bypass | App detecting and blocking proxy → can it be bypassed? |

---

## Mobile API Patterns

Mobile apps often have API-specific vulnerabilities because developers assume the mobile client is trusted.

### Client Trust Issues

| # | Pattern | Test | What to look for |
|---|---------|------|-----------------|
| 1 | Client-side validation only | Skip validation by sending direct API requests | Server accepts invalid data |
| 2 | Hidden API endpoints | Extract endpoints from binary/JS bundle | Undocumented admin or debug APIs |
| 3 | API versioning | Try `/v1/`, `/v2/`, older versions | Deprecated endpoints with weaker auth |
| 4 | Hardcoded API keys | Extract from binary/config | Keys with excessive permissions |
| 5 | Device ID as auth | Check if device ID is sole authenticator | Spoofable device identification |
| 6 | Rate limiting client-only | Remove client-side rate limits | No server-side rate limiting |
| 7 | Premium feature flags | Modify API responses or request parameters | Free access to paid features |
| 8 | Offline mode bypass | Manipulate cached data/responses | Bypass payment or auth checks |

### Authentication Patterns

| # | Pattern | Test | What to look for |
|---|---------|------|-----------------|
| 1 | Biometric bypass | Intercept biometric success callback | Auth succeeds without biometric |
| 2 | PIN/passcode in transit | Capture auth requests | PIN sent in cleartext or weak hash |
| 3 | Session token storage | Check where tokens are stored | Tokens in shared preferences (Android) or unprotected files |
| 4 | Token refresh flow | Intercept refresh requests | Refresh token reuse, no rotation |
| 5 | Push notification token | Check if push tokens are tied to auth | Token reuse across accounts |
| 6 | OAuth mobile flow | Check redirect_uri and state handling | PKCE not enforced, redirect hijacking |
| 7 | Magic link interception | Test deep link handling for auth links | Other apps can intercept auth URLs |
| 8 | Background screenshot | Check if app prevents screenshots on sensitive screens | Sensitive data in app switcher |

---

## Deep Links & URL Schemes

### Android

| # | Pattern | Test | What to look for |
|---|---------|------|-----------------|
| 1 | Exported activities | Check `AndroidManifest.xml` for `android:exported="true"` | Direct access to internal screens |
| 2 | Intent injection | Send crafted intents to exported components | Unexpected behavior, data exposure |
| 3 | Deep link validation | Send crafted deep links: `scheme://path?param=evil` | No input validation on deep link params |
| 4 | WebView via deep link | Deep link that opens attacker URL in WebView | XSS in WebView context |
| 5 | Pending intent hijack | Check for implicit PendingIntents | Intercept and redirect intents |
| 6 | Content provider exposure | Query exported content providers | Database access without authentication |
| 7 | Broadcast receivers | Send crafted broadcasts to exported receivers | Trigger unintended actions |
| 8 | Activity hijacking | Register for same intent filters | Intercept sensitive data in transit |

### iOS

| # | Pattern | Test | What to look for |
|---|---------|------|-----------------|
| 1 | Custom URL scheme hijack | Register same scheme in attacker app | Intercept sensitive URL callbacks |
| 2 | Universal link bypass | Fall back to URL scheme when universal links fail | Less secure URL scheme handling |
| 3 | URL scheme input injection | Send `scheme://action?param=evil` | No validation on parameters |
| 4 | Deep link to WebView | Craft link that loads attacker content in WebView | XSS in app context |
| 5 | OAuth callback hijack | Register URL scheme matching OAuth redirect | Token interception |
| 6 | Clipboard exposure | Check if app reads clipboard (UIPasteboard) | Sensitive data from clipboard |
| 7 | App extension data | Check shared containers between app and extensions | Data leakage between contexts |
| 8 | Handoff abuse | Check Handoff/Continuity for sensitive data | Cross-device data exposure |

---

## Local Data Storage

### Android

| # | Location | Test | Risk |
|---|----------|------|------|
| 1 | SharedPreferences | Check `/data/data/{app}/shared_prefs/` | Tokens, user data in plaintext XML |
| 2 | SQLite databases | Check `/data/data/{app}/databases/` | Unencrypted sensitive data |
| 3 | Internal storage files | Check `/data/data/{app}/files/` | Config files, cached responses |
| 4 | External storage | Check `/sdcard/Android/data/{app}/` | World-readable on older Android |
| 5 | Cache directory | Check `/data/data/{app}/cache/` | Cached API responses with sensitive data |
| 6 | Backup extraction | `adb backup -f backup.ab {package}` | Sensitive data in backups if `allowBackup="true"` |
| 7 | Logcat | `adb logcat \| grep {package}` | Sensitive data logged to system log |
| 8 | Clipboard data | Monitor clipboard programmatically | Passwords or tokens copied to clipboard |

### iOS

| # | Location | Test | Risk |
|---|----------|------|------|
| 1 | Keychain | Dump keychain items for app | Tokens with wrong accessibility flags |
| 2 | NSUserDefaults | Check plist files in app container | Sensitive data in unprotected plist |
| 3 | CoreData / SQLite | Check app container for `.sqlite` files | Unencrypted database |
| 4 | Cache.db | Check `Library/Caches/` | Cached HTTPS responses |
| 5 | Cookies.binarycookies | Check for persistent cookies | Session data persisted to disk |
| 6 | App snapshots | Check `Library/SplashBoard/` | Screenshots of sensitive screens |
| 7 | Pasteboard | Check if app writes to general pasteboard | Other apps can read sensitive data |
| 8 | Backup extraction | iTunes/Finder backup (unencrypted) | Sensitive data in unencrypted backups |

---

## WebView Security

### Android WebView

| # | Pattern | Test | What to look for |
|---|---------|------|-----------------|
| 1 | JavaScript enabled | Check `setJavaScriptEnabled(true)` | XSS in WebView context |
| 2 | JavaScript interface | Check `addJavascriptInterface()` | Bridge to Java methods from JS |
| 3 | File access | Check `setAllowFileAccess(true)` | Read local files via `file://` |
| 4 | Universal file access | Check `setAllowUniversalAccessFromFileURLs()` | Cross-origin file access |
| 5 | Mixed content | Check if HTTPS WebView loads HTTP resources | Content injection |
| 6 | URL loading override | Check `shouldOverrideUrlLoading()` | Missing validation on loaded URLs |
| 7 | Content from deep links | WebView loading URL from intent data | Attacker-controlled content in app context |
| 8 | Cookie theft | JS bridge + cookie access | Exfiltrate session cookies |

### iOS WKWebView

| # | Pattern | Test | What to look for |
|---|---------|------|-----------------|
| 1 | Message handlers | Check `userContentController` | JS-to-native bridge methods |
| 2 | Custom schemes | Check `setURLSchemeHandler` | Custom protocol handling vulnerabilities |
| 3 | Navigation delegate | Check `decidePolicyFor navigationAction` | Missing URL validation |
| 4 | JavaScript evaluation | Check `evaluateJavaScript()` with user input | Code injection |
| 5 | Universal links in WebView | Click on universal link inside WebView | Different handling than Safari |
| 6 | Content from deep links | WebView loading URL from URL scheme | Attacker-controlled content |

---

## Push Notification Security

| # | Pattern | Test | What to look for |
|---|---------|------|-----------------|
| 1 | Sensitive data in notifications | Trigger notifications with sensitive content | PII visible on lock screen |
| 2 | Notification token enumeration | Check if push tokens are predictable | Send notifications to other users |
| 3 | Notification actions | Check notification action handlers | Trigger actions without unlocking |
| 4 | Silent push abuse | Check if app processes silent pushes | Trigger background operations |
| 5 | Rich notification content | Check notification service extension | Content manipulation |

---

## Mobile-Specific Impact Assessment

When reporting mobile vulnerabilities, frame impact in terms of:

1. **Device compromise scope** — Does the bug affect just the app or the device?
2. **Data persistence** — Is the exposed data recoverable even after the app is uninstalled?
3. **Physical vs. remote** — Can the attack be performed remotely or does it need device access?
4. **Scale** — How many devices/users are affected?
5. **Platform specifics** — iOS sandboxing vs. Android's more open model

### Severity Guidelines

| Finding | Typical Severity | Notes |
|---------|-----------------|-------|
| Hardcoded API key with sensitive permissions | High–Critical | Depends on what the key accesses |
| Certificate pinning bypass + sensitive API | Medium–High | Enables MitM, but requires network position |
| Deep link account takeover | High–Critical | Remote exploitation possible |
| Sensitive data in local storage (no root required) | Medium–High | Accessible via backup extraction |
| Missing screenshot prevention | Low | Information disclosure via app switcher |
| Debug build in production | Medium | Enables debugging and data extraction |
| WebView JavaScript bridge exploitation | High–Critical | Depends on exposed native methods |
| Biometric bypass | Medium–High | Requires physical access or malware |

---

## BaaS & Platform Services

> **Full testing procedures:** See [reference/baas-security.md](reference/baas-security.md) for Firebase, Supabase, in-app purchase, and mobile threat intelligence.

| Target Surface | Key Test | Top Finding |
|---|---|---|
| **Firebase** | `curl https://{project-id}.firebaseio.com/.json` | Open Realtime Database = Critical (full PII exposure) |
| **Supabase** | Query tables with extracted anon key | Missing RLS policies = High-Critical |
| **In-App Purchases** | Intercept receipt validation flow | Client-side only validation = free premium access |
| **Storage Buckets** | List contents via REST API | Open bucket with user files = High-Critical |

**Quick start:** Extract project ID from `google-services.json` (Android) or `GoogleService-Info.plist` (iOS), then test each BaaS surface. See reference file for full workflows and severity guidelines.

---

## Tools Reference

### Android

| Tool | Purpose |
|------|---------|
| apktool | APK decompilation and recompilation |
| jadx | Java decompilation from DEX |
| Frida | Dynamic instrumentation and hooking |
| Objection | Runtime mobile exploration (built on Frida) |
| adb | Android Debug Bridge — device interaction |
| drozer | Android security assessment framework |
| MobSF | Automated mobile security analysis |

### iOS

| Tool | Purpose |
|------|---------|
| class-dump | Extract class information from binary |
| Frida | Dynamic instrumentation and hooking |
| Objection | Runtime mobile exploration |
| ipatool | IPA download and extraction |
| MobSF | Automated mobile security analysis |
| Hopper/Ghidra | Binary disassembly and analysis |

### Both Platforms

| Tool | Purpose |
|------|---------|
| Burp Suite / mitmproxy | Traffic interception and analysis |
| Frida | Universal hooking framework |
| Objection | Mobile exploration toolkit |
| MobSF | Static and dynamic analysis |

---

## Validation Gate — Is This Submittable?

Before writing the report, check these mobile-specific false positives:

| Looks Like a Bug | Why It Usually Isn't |
|---|---|
| Certificate pinning not implemented | Most programs accept this as Low/Informational at best — only High if you demonstrate MITM → credential theft on a real network |
| Root/jailbreak detection bypass | Bypass alone is Informational — chain with data extraction or privilege escalation |
| Data stored unencrypted on device | Only a finding with physical access + sensitive data; most programs require remote exploitation |
| Hardcoded API key in APK/IPA | Only a finding if the key grants access to sensitive data or paid services — many are public/limited keys |
| Debug logging enabled in release build | Informational unless logs contain tokens, PII, or credentials |
| Missing `android:exported="false"` on component | Only a finding if you can demonstrate exploitation (data exfiltration, auth bypass, or action trigger) |
| WebView with JavaScript enabled | Standard behavior — only a finding if the WebView loads attacker-controlled content leading to token theft or RCE |

**Common mistakes in mobile reports:**
- Reporting insecure storage findings that require physical device access (most programs exclude this)
- Claiming certificate pinning bypass as High without demonstrating the full MITM attack chain
- Submitting hardcoded keys without verifying they grant meaningful access
- Reporting root detection bypass without a follow-up exploit
- Listing multiple minor issues as separate reports instead of chaining them

**Severity calibration:** Cert pinning bypass alone = Low/Informational. Deep link → WebView XSS → token theft = High. Exported content provider → PII = High. Intent injection → payment trigger = Critical.

---

## Reference Files

This skill uses progressive disclosure. Detailed reference material is available on demand:

| File | Contents | Lines |
|------|----------|-------|
| [reference/baas-security.md](reference/baas-security.md) | Firebase misconfigs (8 patterns + testing workflow + severity escalation), Supabase misconfigs (6 patterns + RLS testing), in-app purchase manipulation (5 patterns + StoreKit 2/Play Billing notes), BaaS severity guidelines, active mobile threats (CVE-2026-21385 Qualcomm, March 2026 Android update, iOS zero-days), common BaaS report mistakes | ~160 |

**Quick search** — find specific BaaS/threat patterns:
```
grep -n "Firebase\|Firestore\|firebaseio" ${CLAUDE_SKILL_DIR}/reference/baas-security.md
grep -n "Supabase\|RLS\|service_role" ${CLAUDE_SKILL_DIR}/reference/baas-security.md
grep -n "CVE\|zero-day\|threat\|CISA" ${CLAUDE_SKILL_DIR}/reference/baas-security.md
```

---

## Related Skills

- **target-recon** — Identify mobile API endpoints and infrastructure
- **vuln-patterns** — API-level vulnerability patterns applicable to mobile backends
- **source-code-audit** — Audit mobile app source code (if available)
- **report-writing** — Write up mobile findings with platform-appropriate framing
- **cloud-security** — Mobile apps frequently interact with cloud services
