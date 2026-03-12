---
name: infrastructure-security
description: Tests infrastructure targets — network appliances, management consoles, enterprise platforms, remote access tools, MDM, firewalls, webmail, backup appliances, workflow automation, and Windows trust mechanisms. Use when the target is infrastructure (not a web app), when testing network management interfaces, appliance admin panels, or enterprise IT platforms. Trigger on "network appliance", "firewall", "VPN", "SD-WAN", "MDM", "remote desktop", "management console", "admin panel", "backup appliance", "webmail", "enterprise platform", "SolarWinds", "Cisco", "Fortinet", "Ivanti", "Palo Alto", "VMware", "Dell", "Juniper", "RustDesk", "MeshCentral", "Guacamole", "n8n", "workflow automation", "infrastructure CVE", "appliance hardcoded credentials", "deserialization RCE", "MotW bypass", "Mark of the Web", "ingress-nginx", "SSO bypass", "RRAS", "FMC". Also activates when hunt-session classifies target archetype as "Infrastructure", when recon reveals management ports (8443, 8305, 9090, 21115-21119), or when Shodan/Censys results show appliance fingerprints. For Kubernetes-specific patterns (ingress-nginx, CoreDNS), see reference/infrastructure-vulns.md. For cloud infrastructure (AWS/GCP/Azure), use cloud-security.
---

# Infrastructure Security Testing

Infrastructure targets — appliances, management consoles, enterprise platforms — are where the highest-value CVEs live. CVSS 9.8-10.0 is common because these systems control entire networks, device fleets, or security boundaries. They also have the lowest automation pressure: AI tools focus on web apps, not proprietary admin interfaces with custom protocols.

## Quick Start — First 5 Minutes

1. **Fingerprint** — Identify product + version from headers, login pages, error messages, favicon hashes. `curl -sI https://target:8443 | grep -i server` + Shodan/Censys lookup.
2. **CVE check** — Search NVD for the exact product+version. Sort by CVSS descending. Check CISA KEV for active exploitation. If a CVE exists with public PoC → test immediately.
3. **Default creds** — Try default/hardcoded credentials. Check `tomcat-users.xml`, `application.yml`, `.env`, admin:admin, manufacturer defaults. Use vendor-specific default cred lists.
4. **Unauth endpoints** — Probe `/api/`, `/admin/`, `/backup`, `/export`, `/download`, `/config`, `/manager/`. Management interfaces often expose endpoints without auth.
5. **Management ports** — Scan for non-standard ports: 8443 (admin HTTPS), 8305 (FMC), 9090 (management), 21115-21119 (RustDesk), 5678 (n8n).

If you find something, run `/scope-check` first (infrastructure may be vendor-managed), then `/reportability-check` before `/write-report`.

---

## Scope Gate (Run First)

Before investing time, verify the infrastructure target is in scope:

1. **Ownership** — Is this the target org's asset or vendor-managed cloud infrastructure? Cloud-managed appliances (e.g., CVE-2026-21536 in Microsoft Devices Pricing Program) may already be patched server-side with no customer action needed.
2. **Program scope** — Does the bug bounty program include infrastructure/network targets? Many programs exclude network devices, VPN appliances, and management consoles.
3. **Vendor vs customer** — Some management consoles are SaaS-hosted by the vendor. If the vendor owns the infrastructure, report to the vendor's program, not the customer's.

If scope is unclear, run `/scope-check` before proceeding.

### Common Report Killers

| Mistake | Why It Gets N/A'd |
|---------|-------------------|
| Vendor-managed infrastructure | "This is our cloud service, already patched" — no customer impact |
| Known CVE on unpatched version | Already reported by automated scanners; check for duplicates first |
| Default creds on test/staging | Dev/staging instances may be intentionally left open; verify production |
| Shodan finding without exploitation | "Port is open" is not a vulnerability — demonstrate impact |
| Fingerprint without impact proof | Version detection alone is informational; show what an attacker can do |

---

## Execution Flow

```
1. Fingerprint product + version (headers, login page, error messages)
2. CVE/CISA KEV lookup — known vulns for this exact version?
3. Default credentials check — hardcoded, default, leaked creds
4. Unauthenticated endpoint enumeration — API, admin, backup, export
5. Authentication mechanism analysis — test bypass patterns
6. Post-auth testing — if you get in, test for RCE, privesc, lateral movement
7. Protocol-specific testing — NETCONF, SNMP, custom protocols
8. Variant hunting — if a related product was patched, test this one
9. Validation loop — confirm severity, verify boundary crossed
```

---

## Target Categories

Infrastructure targets cluster into categories with distinct attack patterns. Identify your target's category, then follow the category-specific methodology.

### 1. Network Management & SD-WAN

**What:** SD-WAN controllers, network management consoles, router admin interfaces. Control entire network fabrics.

**Why high-value:** Compromising one controller = fabric-wide access. Consistently CVSS 9.8-10.0 with root/SYSTEM.

**Primary attack patterns:**
- **Peering/federation auth bypass** — Controllers that peer with each other may trust peer connections without full auth (CVE-2026-20127 Cisco SD-WAN CVSS 10.0)
- **Web management interface auth bypass** — Unauthenticated auth bypass in web-based management UI that, in some cases, enables admin password reset (CVE-2026-23813 HPE Aruba AOS-CX CVSS 9.8, reported via HPE bug bounty, published March 11 2026)
- **Post-auth credential exposure** — Management API endpoints that leak credential files to authenticated users (CVE-2026-20128, requires vManage credentials; CVE-2026-20122 CVSS 5.4, requires read-only API credentials)
- **Software downgrade chains** — Auth bypass → downgrade to vulnerable version → exploit old privesc → restore (UAT-8616 technique, active since 2023)
- **EoL device auth bypass** — Devices past end-of-life with improper access control (CVE-2026-0625 D-Link DNS config, no fix ever)

**Test procedure:**

| # | Test | What to do | Signal |
|---|------|-----------|--------|
| 1 | Peering endpoint auth | Probe peering/clustering APIs without credentials | Admin access without auth |
| 2 | Web mgmt reset workflow | Probe password reset/recovery routes without prior auth — check for token issuance or missing auth on reset flow, do NOT execute reset on live targets (CVE-2026-23813 pattern) | Reset endpoint reachable without auth |
| 3 | Credential file access | Request `/api/config`, `/backup`, credential management endpoints | Credential files returned |
| 4 | NETCONF config push | After any auth bypass, test NETCONF for fabric-wide config changes | Routing modification capability |
| 5 | Version management | Check for downgrade capability via version management endpoints | Ability to install older, vulnerable firmware |
| 6 | EoL version detection | Fingerprint version; check if past vendor EoL date | No patch available = permanent risk |

**Decision logic:** If target is a network controller managing 10+ devices, invest heavily — the reward-to-effort ratio is extreme. If it's a single standalone device, probe quickly (15 min max) then move on.

### 2. Firewall & Security Appliance Management

**What:** Firewall management consoles (Cisco FMC/FTD, Palo Alto Panorama, FortiManager, Check Point SmartConsole).

**Why high-value:** Dual CVSS 10.0 is not unusual (CVE-2026-20131 + CVE-2026-20079). Unauth RCE as root on the device that controls your entire security posture.

**Primary attack patterns:**
- **Java deserialization** — Management APIs accepting serialized Java objects (Commons Collections, Spring, ROME gadget chains)
- **Boot-time auth bypass** — Services during startup/migration with reduced auth
- **SSO trust model abuse** — Cross-tenant access via SSO federation (CVE-2026-24858 FortiCloud CVSS 9.4)

**Test procedure:**

| # | Test | What to do | Signal |
|---|------|-----------|--------|
| 1 | Java deser probe | Send crafted serialized objects to management endpoints; test Commons Collections, Spring, ROME chains | DNS/HTTP callback from target |
| 2 | Boot/migration auth | Test management APIs during boot or upgrade — auth may be weakened | Admin access without normal credentials |
| 3 | Cross-tenant SSO | Authenticate to cloud management portal; enumerate device IDs from other tenants | Device access across tenant boundaries |
| 4 | Version fingerprint | Identify exact firmware version; match against CVE database | Known vulnerable version in production |

**Decision logic:** If Shodan shows the management port is internet-exposed, this is highest priority. Firewall management consoles should never be internet-facing — if they are, the org likely has poor security hygiene and multiple issues.

### 3. MDM, Endpoint & Enterprise Management

**What:** Mobile Device Management (Ivanti EPMM, ManageEngine, VMware Workspace ONE, Microsoft Intune), endpoint management (Ivanti EPM, SCCM), and IT management platforms (SolarWinds WHD, VMware Aria, ServiceNow).

**Why high-value:** MDM servers control thousands of devices — push configs, install apps, wipe devices. Endpoint managers control desktop/server fleets with SYSTEM-level access. Compromising either = full fleet control. Enterprise management platforms often store credentials for their entire managed environment.

**Primary attack patterns:**
- **Bash code injection in RewriteMap handlers** — Specific EPMM URL rewrite-map handlers pass user input through Bash variable/array handling, enabling command substitution via arithmetic evaluation (CVE-2026-1281/1340, on-prem EPMM 12.5.x/12.6.x/12.7.0.0, CVSS 9.8). Targets: `/mifs/c/appstore/fob/` and `/mifs/c/aftstore/fob/` paths — not generic parameter injection
- **Alternate auth path bypass** — Management APIs with inconsistent auth enforcement across endpoints. CVE-2026-1603 (Ivanti EPM CVSS 8.6 vendor / 7.5 NVD, CISA KEV March 9 2026): auth bypass via alternate weak authentication path (ZDI-26-080: `AuthHelper` flaw) in EPM before 2024 SU5 allows unauthenticated leakage of stored credential data. **Generalized test:** probe management API endpoints for inconsistent auth — some paths may enforce auth while others serving the same data do not
- **Deser filter bypass via URI-gated or key-obfuscation techniques** — Vendors patch deserialization by filtering input conditionally (e.g., URI-based gates, key name checks), but alternate routes or obfuscation bypass the filter. CVE-2025-26399 (SolarWinds WHD ≤12.8.7, CVSS 9.8, CISA KEV March 9 2026): 3rd bypass in chain (CVE-2024-28986 → CVE-2024-28988 → CVE-2025-26399) — patch checked `request.uri().contains("ajax")` in `checkSuspeciousPayload` before sanitizing `params`/`fixups` in JSON payloads; subsequent bypass (CVE-2025-40553) used JSON-key obfuscation (`p\x61rams`, `java\x43lass`) to evade the filter. Post-exploitation observed on WHD instances (Microsoft, exact CVE attribution unconfirmed): ManageEngine RMM deployment, reverse SSH/RDP, QEMU VMs, DLL sideloading, DCSync. **Generalized test:** for any patched deser endpoint, look for URI-gated filtering (e.g., `contains("ajax")`), JSON-key obfuscation, alternate URL paths, or encoding tricks to bypass the filter
- **Unauthenticated API endpoints** — Management APIs that skip auth on specific paths
- **Migration/upgrade command injection** — Privileged operations with reduced validation during support-assisted migration (CVE-2026-22719 VMware Aria CVSS 8.1, exploitable while support-assisted product migration is in progress)

**Test procedure:**

| # | Test | What to do | Signal |
|---|------|-----------|--------|
| 1 | Bash code injection | Target specific RewriteMap handler paths (e.g., `/mifs/c/appstore/fob/`, `/mifs/c/aftstore/fob/`) with command substitution in URL parameters | OS command execution via Bash variable/arithmetic evaluation |
| 2 | Pre-auth endpoint scan | Enumerate `/mifs/`, `/api/`, `/admin/` paths without credentials | Admin functions accessible pre-auth |
| 3 | Migration endpoint injection | Test upgrade/migration endpoints for command injection | Command execution during privileged operations |
| 4 | Device enrollment abuse | Use enrollment endpoints to upload arbitrary files | Web shell deployment via enrollment flow |
| 5 | API version mismatch | Test `/v1/` alongside `/v2/` — older APIs may lack auth checks | Older API version accessible without auth |
| 6 | Alternate auth path probe | Test management API endpoints for inconsistent auth enforcement — some paths may bypass auth controls that protect others (CVE-2026-1603 pattern) | Stored credential data returned without authentication |
| 7 | Deser filter bypass | Test patched deser endpoints with URI-gated filter evasion (e.g., alternate paths bypassing `contains("ajax")` checks) or JSON-key obfuscation (`p\x61rams`) | Deserialization processed despite vendor patch (CVE-2025-26399/40553) |

**Decision logic:** CISA KEV additions for MDM/EPM CVEs indicate mass exploitation is already happening. Internet-exposed or unpatched MDM/EPM systems are high risk and may already be compromised — the bug report is still valid and highly rewarded.

### 4. Remote Access & Desktop Tools

**What:** Self-hosted remote desktop (RustDesk, Guacamole, MeshCentral), VPN gateways, remote access tools commonly exposed to the internet.

**Primary attack patterns:**
- **Client login/peer-auth bypass** — Authentication flaws in client login and peer authentication modules (CVE-2026-30789 RustDesk Client ≤1.4.5)
- **Pre-auth SSRF** — Internal network access before password verification
- **Cleartext credential transmission** — Missing TLS or weak crypto in custom protocols (CVE-2026-30796 RustDesk)

**Test procedure:**

| # | Test | What to do | Signal |
|---|------|-----------|--------|
| 1 | Session replay | Capture and replay authentication session tokens | Access granted with replayed credentials |
| 2 | Pre-auth SSRF | Send requests to internal IPs before authenticating | Internal port scan results pre-auth |
| 3 | API message manipulation | Intercept and modify API messages between client and server | Unauthorized operations via modified messages |
| 4 | Custom protocol analysis | Capture traffic on non-standard ports (21115-21119 for RustDesk) | Cleartext credentials or weak crypto |

**Decision logic:** Shodan shows thousands of exposed instances. Scan for `rustdesk` on ports 21115-21119, `guacamole` on 8080/8443, `meshcentral` on 443. If self-hosted and internet-facing, invest — these are systematically under-tested.

### 5. Backup, DR & Storage Appliances

**What:** Enterprise backup and disaster recovery (Dell RecoverPoint, Veeam, Commvault, Veritas, Cohesity, Rubrik), management UIs with backup/export.

**Why high-value:** Backup appliances access all protected data, lack EDR monitoring, and APT groups use them for long-term persistence. Hardcoded credentials are endemic.

**Primary attack patterns:**
- **Hardcoded credentials** — Plaintext creds in config files (`tomcat-users.xml`, `application.yml`)
- **Unauthenticated backup endpoints** — `/api/backup`, `/export`, `/download` accessible without auth, encryption keys leaked in response headers (CVE-2026-27944 Nginx UI CVSS 9.8: `/api/backup` returns full server backup + AES key in `X-Backup-Security` header → creds, SSL keys, configs). Affects all Nginx UI < 2.3.3
- **Ephemeral lateral movement** — "Ghost NICs" (ephemeral virtual interfaces) for stealthy network pivoting (CVE-2026-22769 Dell RecoverPoint CVSS 10.0)

**Test procedure:**

| # | Test | What to do | Signal |
|---|------|-----------|--------|
| 1 | Default/hardcoded creds | Check `tomcat-users.xml`, `application.yml`, `.env` | Admin access without brute-forcing |
| 2 | Unauth backup download | Request `/api/backup`, `/backup`, `/export`, `/download` without auth | Backup file returned |
| 3 | Encryption key leakage | Check response headers (`X-Backup-Security`, `X-*`) on backup endpoints | Encryption keys/IVs leaked in headers |
| 4 | Tomcat Manager access | Test WAR file deployment via `/manager/text/deploy` | Web shell deployment capability |

**Decision logic:** Backup appliances are the highest-value infrastructure target per unit of effort. They're rarely tested, almost never have EDR, and consistently have CVSS 10.0 vulnerabilities. Prioritize over web apps when in scope.

### 6. Workflow Automation Platforms

**What:** Self-hosted workflow automation (n8n, Make, Huginn, Windmill, Activepieces, Apache Airflow, Temporal).

**Why high-value:** These platforms execute arbitrary code by design — any auth bypass = immediate RCE. 100K+ self-hosted n8n instances globally.

**Primary attack patterns:**
- **Content-type bypass** — Manipulating content-type to override file handling (CVE-2026-21858 n8n CVSS 10.0)
- **Expression injection** — Crafting workflow expressions that execute system commands (CVE-2026-25049 n8n CVSS 9.9)
- **`vm` sandbox escape** — Host objects (Playwright, Puppeteer, `child_process`) exposed to sandboxed user scripts (CVE-2026-30957 OneUptime CVSS 9.9)

**Test procedure:**

| # | Test | What to do | Signal |
|---|------|-----------|--------|
| 1 | Content-type bypass | Send requests without `multipart/form-data` to file handling endpoints | `req.body.files` override enabling path manipulation |
| 2 | Expression injection | Craft workflow expressions with `$()` or `require()` payloads | Command execution via workflow runtime |
| 3 | Sandbox escape | In custom script/code blocks, probe for exposed host objects | Access to `child_process`, Playwright, or DB clients from sandbox |
| 4 | Credential extraction | After initial foothold, access config/database files | Admin credentials, encryption keys |

**Decision logic:** If Shodan shows n8n on port 5678 or any workflow platform internet-facing, this is a high-probability target. The "code execution by design" property means any auth weakness is immediately critical.

### 7. Webmail & Email Infrastructure

**What:** Self-hosted webmail (Roundcube, Zimbra, Horde), email gateways, Exchange.

**Primary attack patterns:**
- **Rapid weaponization** — Critical RCE/XSS weaponized within 48 hours of disclosure (CVE-2025-49113 Roundcube CVSS 9.9)
- **Stored XSS via email** — Malicious HTML/JS in email body or headers executing in victim's session
- **MIME parsing confusion** — Malformed MIME structures causing parser-level code execution

**Test procedure:**

| # | Test | What to do | Signal |
|---|------|-----------|--------|
| 1 | Version fingerprint | Check server headers, `/program/lib/Roundcube/rcube.php` | Unpatched version (< 1.6.11 / < 1.5.10 for Roundcube) |
| 2 | Stored XSS via email | Send crafted email with malicious HTML/JS in body/headers | Script execution in victim's webmail session |
| 3 | MIME parsing abuse | Send email with malformed MIME structure | Parser confusion leading to code execution |

**Decision logic:** 84,000+ internet-facing Roundcube instances. If version is < 1.6.11, it's almost certainly vulnerable. 48-hour weaponization means unpatched instances are likely already compromised — but the report is still valid.

---

## Windows Trust Mechanism Bypasses

MotW (Mark-of-the-Web) bypasses are a recurring high-value pattern. Three separate MotW bypasses in February 2026 (CVE-2026-21510, 21513, 21514) show this is a hot vulnerability class.

**Why it matters:** MotW is Windows' primary trust mechanism for downloaded files. Bypassing it defeats SmartScreen, Protected View, and macro-blocking simultaneously. APT28 actively exploits these.

**Where to look:**
- Applications processing URLs through MSHTML/IE components (`ieframe.dll`, `mshtml.dll`)
- Electron and CEF-based apps that may invoke IE components for URL handling
- File format handlers that embed URLs (LNK, DOCX, RTF, PDF)

**Test patterns:**

| # | Test | What to do | Signal |
|---|------|-----------|--------|
| 1 | URL validation in IE components | Craft URLs that pass validation but invoke dangerous handlers | MotW removed; SmartScreen bypassed |
| 2 | LNK file MotW bypass | Create LNK files with crafted target URLs | File executes without Protected View warning |
| 3 | DOCX embedded URL bypass | Embed URLs that invoke legacy IE rendering paths | Macros/scripts execute without MotW protection |
| 4 | Cross-zone navigation | Craft URLs that transition from Internet zone to Local Machine zone | Elevated privileges for web content |

---

## Stack-Matched Variant Hunting

Infrastructure vendors ship the same bug patterns across product families. Every published CVE has potential siblings in related products, adjacent endpoints, or incomplete patches. This is the highest-ROI activity in infrastructure testing — you start with a known vulnerability and systematically discover new findings.

### The Variant Hunting Table

Before probing, build a stack-matched table for your target. Fill in each column from recon + CVE research:

| Component | Version Evidence | Advisory/CVE | Why Applicable | First Probe |
|-----------|-----------------|--------------|----------------|-------------|
| AOS-CX 10.13.1100 | `Server: ArubaOS-CX` header | CVE-2026-23813 (CVSS 9.8) | Same web mgmt interface, unpatched version | Password reset endpoint without auth |
| Cisco FMC 7.2 | Port 8305, `Firepower` in title | CVE-2026-20131 (CVSS 10.0) | Java deser on management API | ysoserial Commons Collections payload |
| SolarWinds WHD 12.8.7 | `/helpdesk/` path, version header | CVE-2025-26399 (3rd bypass) | AjaxProxy deser still reachable | Alternative gadget chains on patched endpoint |
| Ivanti EPMM 12.3 | `/mifs/` path, `MobileIron` header | CVE-2026-1281 (CVSS 9.8) | Bash arithmetic expansion in URL params | `$((command))` in GET parameters |

**How to populate:** (1) Fingerprint product + version from recon. (2) Check vendor security advisories and GHSA first (NVD often has minimal enrichment for fresh CVEs). (3) Check CISA KEV for active exploitation. (4) Use NVD for normalization (CVSS, CWE). (5) For each CVE, assess why the same root cause might exist on your specific instance. (6) Design a non-destructive first probe — one request that confirms or rules out the vulnerability without modifying target state.

### Variant Hunting Strategies

**Confirmed patterns** (documented CVE-to-CVE chains):

| Strategy | What to Try | Why It Works | Confirmed Example |
|----------|-------------|--------------|-------------------|
| **Patch bypass** | Alternative gadget chains on a patched endpoint | Vendors block the specific exploit but not the root cause | SolarWinds WHD: 3 deser CVEs on the same AjaxProxy endpoint |
| **Adjacent endpoint** | Same root cause on a different API path or feature | Developers repeat patterns across endpoints | Ivanti EPMM: `/mifs/c/appstore/fob/` (CVE-2026-1281) + `/mifs/c/aftstore/fob/` (CVE-2026-1340) |
| **Version regression** | Check for downgrade capability + old vuln (do NOT downgrade live targets without authorization) | Downgrade capability + old vuln = chain | UAT-8616: auth bypass → software downgrade → exploit old privesc → restore |

**Hunting heuristics** (reasonable but unconfirmed — test carefully):

| Strategy | What to Try | Rationale |
|----------|-------------|-----------|
| **Sibling product** | Same CWE on a related product from the same vendor family | Vendors reuse code — but check vendor advisory for actual scope first |
| **Transport parity** | Same flaw via a different protocol (HTTP vs NETCONF vs SNMP) | Fixes may apply to one transport only |
| **Deployment mode** | Test standalone vs clustered vs cloud-managed variants | Patches may not cover all deployment modes |

### Deserialization Variant Hunting (Specific Pattern)

Deserialization patch bypasses are the most reliable variant pattern. Vendors block one gadget chain but leave the deser endpoint accessible — or filter by URL path but miss alternate routes.

1. Find a previously patched deser CVE for the target product
2. Verify the deser endpoint is still accessible post-patch
3. Test alternative gadget chains (Commons Collections, Spring, ROME, BeanUtils)
4. If the fix validates object types, test chain variants using allowed types
5. **Test URI-gated and key-obfuscation filter bypass** — if the patch filters by URI content (e.g., `request.uri().contains("ajax")`), access the same endpoint via an alternate path or use JSON-key obfuscation to evade key name checks (SolarWinds WHD chain: CVE-2025-26399 bypassed URI gate; CVE-2025-40553 used `p\x61rams` key obfuscation)
6. Check if the fix was ported to all deployment modes (standalone, clustered, cloud)

**High-value deser targets:** SolarWinds WHD (3 bypass CVEs), Atlassian Confluence, Fortra GoAnywhere MFT, Apache OFBiz, VMware products, SAP NetWeaver.

### When to Stop Variant Hunting

- All versions in the stack-matched table are patched AND you've tested alternative transports/endpoints
- The target product doesn't match any recent CVE + you've exhausted sibling/adjacent probes (spend 30 min max)
- **Next probe would be destructive** — requires password reset, firmware change, service restart, or cross-device impact → document the finding and request explicit authorization before proceeding
- You find a confirmed variant → stop hunting, start reporting. Run `/reportability-check` then `/write-report`

---

## Prioritization Logic

```
IF target is internet-exposed management console:
    PRIORITY = CRITICAL (these should never be internet-facing)

IF target has known CVE within past 90 days AND version matches:
    PRIORITY = HIGH (test for patch bypass or variant)

IF target is backup/DR appliance:
    PRIORITY = HIGH (highest value per unit effort)

IF target is workflow automation platform:
    PRIORITY = HIGH (auth bypass = immediate RCE)

IF target is EoL product:
    PRIORITY = HIGH (no patch coming = permanent risk)

IF target is single standalone device (not managing others):
    PRIORITY = MEDIUM (probe 15 min, move on unless you find something)
```

---

## Validation Loop

Before reporting any infrastructure finding:

1. **Boundary check** — What security boundary was crossed? (unauth→admin, tenant-A→tenant-B, user→root)
2. **Impact chain** — What can an attacker actually do? (RCE, credential theft, fleet control, data exfil)
3. **Scope verification** — Is this the target's asset or vendor-managed? Cloud-managed infrastructure may be out of scope
4. **Reproduction** — Can you reproduce from scratch? Document exact steps, version, and network conditions
5. **Variant assessment** — Is this a known CVE or a new variant? Variants of known CVEs are still valid findings

Run `/reportability-check` before `/write-report`. Infrastructure findings often need extra context about fleet impact — "this controls N devices" is a severity multiplier.

---

## Reference

For detailed CVE examples, test patterns, and vulnerability details organized by category, see:
- [reference/infrastructure-vulns.md](reference/infrastructure-vulns.md) — 17 vulnerability categories with test tables, CVE references, and severity guidance
