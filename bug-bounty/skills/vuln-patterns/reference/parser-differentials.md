# Parser Differentials & Canonicalization Attacks

Cross-cutting attack class exploiting disagreements between components that parse the same input differently. Named #1 exploitation technique in PortSwigger's Top 10 Web Hacking Techniques of 2025. Reference file for the vuln-patterns skill.

> **Related skills:** [http-desync](../../http-desync/SKILL.md) (proxy parser quirks), [auth-testing](../../auth-testing/SKILL.md) (SAML parser differentials), [client-side-security](../../client-side-security/SKILL.md) (DOM parsing)

---

## Table of Contents

- [Core Concept](#core-concept)
- [Unicode Normalization Attacks](#unicode-normalization-attacks)
- [URL Parser Differentials](#url-parser-differentials)
- [Content-Type Confusion](#content-type-confusion)
- [HTTP/2 Protocol Downgrade Differentials](#http2-protocol-downgrade-differentials)
- [Path Normalization Differentials](#path-normalization-differentials)
- [Filename Canonicalization](#filename-canonicalization)
- [SAML / XML Parser Differentials](#saml--xml-parser-differentials)
- [JSON Parser Differentials](#json-parser-differentials)
- [Cache Key vs. Origin Differentials](#cache-key-vs-origin-differentials)
- [Side-Channel Differentials](#side-channel-differentials)
- [Where to Hunt](#where-to-hunt)

---

## Core Concept

A **parser differential** exists whenever two components in a request path interpret the same input differently:

```
Attacker Input → [Component A parses as X] → [Component B parses as Y] → Exploit
```

Any multi-layer architecture (WAF → reverse proxy → application → database) creates parser differential opportunities. The more layers, the more differentials.

**PortSwigger Top 10 (2025):** Parser differentials are "the defining exploitation primitive of 2025" — underpinning cache deception, request smuggling, WAF bypass, auth bypass, and SSRF chains.

---

## Unicode Normalization Attacks

**What it is:** Different systems normalize Unicode codepoints differently. An attacker uses fullwidth, combining, or visually similar characters that one component accepts and another normalizes into dangerous characters.

**Key patterns:**

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | Fullwidth angle bracket XSS | `＜script＞alert(1)＜/script＞` (U+FF1C, U+FF1E) | WAF passes fullwidth; backend normalizes to ASCII `<>` |
| 2 | Fullwidth slash path traversal | `..／..／etc／passwd` (U+FF0F) | WAF allows fullwidth; filesystem normalizes to `/` |
| 3 | Combining character auth bypass | `admin\u0300` vs `admin` | Auth check sees different string; DB normalizes to same user |
| 4 | Homoglyph domain confusion | `аdmin` (Cyrillic а) vs `admin` (Latin a) | SSRF filter checks ASCII; DNS resolves to attacker domain |
| 5 | NFKC normalization injection | `ℌello` (U+210C) → `Hello` after NFKC | Input validation on raw input; normalized form triggers different logic |
| 6 | Zero-width character insertion | `ad​min` (U+200B between d/m) | Length check passes; rendered string matches `admin` |

**Real-world example:** CVE-2026-28289 (FreeScout, CVSS 10.0) — Zero-Width Space (U+200B) in email attachment filename bypasses `.php` extension validation at upload time, stripped at storage → valid PHP webshell. Generalized pattern: filename canonicalization + TOCTOU.

**Where WAFs fail:** Most WAFs operate on raw bytes. Backend frameworks (Python, Java, .NET) apply Unicode normalization forms (NFC, NFKC, NFKD) after the WAF check. Test every input that passes through WAF → application normalization.

**Severity Guidance:** High-Critical when bypass leads to XSS, path traversal, auth bypass, or SSRF. The key finding is the normalization differential itself — document which component normalizes and which doesn't.

---

## URL Parser Differentials

**What it is:** URL parsing libraries (Python's `urllib`, Go's `net/url`, Node's `url`, browser `URL()`, Java `URI`) disagree on edge cases: authority parsing, scheme handling, fragment processing, encoded characters, and userinfo.

**Key patterns:**

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | Authority confusion | `http://evil.com#@trusted.com/api` | SSRF filter sees `trusted.com`; actual request goes to `evil.com` |
| 2 | Backslash-as-slash | `http://evil.com\@trusted.com` | Some parsers treat `\` as `/`; others as part of userinfo |
| 3 | Scheme confusion | `file:///etc/passwd` vs `file:/etc/passwd` | Different parsers accept different scheme formats |
| 4 | Double-encoding | `%252e%252e%252f` (`../../`) | Proxy decodes once; origin decodes twice → path traversal |
| 5 | Encoded @ in URL | `http://trusted.com%40evil.com` | Parser A reads host as `trusted.com`; Parser B as `evil.com` |
| 6 | Tab/newline injection | `http://evil.com\t@trusted.com` | Whitespace handling differs between URL parsers |
| 7 | IPv6 bracket abuse | `http://[::ffff:127.0.0.1]` | Some SSRF filters don't recognize IPv4-mapped IPv6 |

**Where to hunt:** Any SSRF filter, URL allowlist/blocklist, redirect validation, or webhook URL verification. The filter and the HTTP client must use the same URL parser — when they don't, differential exists.

**Severity Guidance:** Critical when SSRF filter bypass leads to cloud metadata or internal API access. The CVE-2026-27127 (Craft CMS) DNS rebinding SSRF is a specific URL parser TOCTOU variant.

---

## Content-Type Confusion

**What it is:** Server-side and client-side disagreement on how to parse request/response bodies based on `Content-Type` headers, file extensions, or magic bytes.

**Key patterns:**

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | JSON/form confusion | Send JSON body with `application/x-www-form-urlencoded` Content-Type | Server parses as form; security middleware expected JSON |
| 2 | XML/JSON polyglot | Craft body valid as both XML and JSON | XXE triggered when XML parser selected over JSON |
| 3 | Multipart boundary confusion | Duplicate or conflicting `boundary` values | Different parsers pick different boundaries → parameter smuggling |
| 4 | MIME sniffing bypass | Upload `.txt` with HTML/JS magic bytes | Browser sniffs content as HTML despite declared type |
| 5 | TypeScript type erasure | `Content-Type: text/typescript` → server strips types → JS | Type-checked input becomes untyped at runtime |

**Real-world example:** CVE-2026-21858 (n8n Ni8mare, CVSS 10.0) — Content-Type confusion in Form Webhook allowed `req.body.files` override → unauthenticated RCE in ~100K servers.

**Severity Guidance:** High-Critical when confusion leads to parameter smuggling, XXE, or auth bypass. The n8n pattern (Content-Type → body property override → RCE) is a reusable hunt target.

---

## HTTP/2 Protocol Downgrade Differentials

**What it is:** HTTP/2 features (binary framing, header compression, multiplexing) create mismatches when proxies downgrade to HTTP/1.1 or when H2-specific behaviors don't map to H1 equivalents.

**Key patterns:**

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | H2.CL smuggling | HTTP/2 request with `Content-Length` header (should be stripped) | Backend processes extra data after H2→H1 downgrade |
| 2 | H2.TE smuggling | HTTP/2 with `Transfer-Encoding: chunked` in headers | Backend processes chunked body after H2 conversion |
| 3 | CONNECT method abuse | HTTP/2 CONNECT to internal service | Proxy allows CONNECT; backend processes as regular request |
| 4 | Header case sensitivity | Mixed-case headers in H2 (`:Method` vs `:method`) | H2 is case-sensitive; H1 proxy may normalize differently |
| 5 | Pseudo-header injection | Inject `:authority` or `:path` overrides in H2 CONTINUATION | Proxy trusts original; backend uses injected values |

**Where to hunt:** Any architecture with H2→H1 downgrade (CDN → origin, load balancer → backend). See [http-desync skill](../../http-desync/SKILL.md) for detailed smuggling patterns.

**Severity Guidance:** High-Critical. H2 smuggling enables request hijacking, cache poisoning, and auth bypass. The CONNECT method exploitation (PortSwigger Top 10 #2) is newly viable against modern H2 deployments.

---

## Path Normalization Differentials

**What it is:** Different components (proxy, web server, application, framework router) normalize URL paths differently — dot segments, encoded slashes, trailing characters, and case sensitivity.

**Key patterns:**

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | Dot segment traversal | `/admin/../public/../admin/secret` | Proxy resolves path; application sees literal path |
| 2 | Encoded slash bypass | `/admin%2fsecret` vs `/admin/secret` | WAF doesn't decode; application does → access control bypass |
| 3 | Trailing dot bypass | `/admin/secret.` | Some frameworks strip trailing dots; path matchers don't |
| 4 | Semicolon parameter | `/admin;jsessionid=x/secret` | Tomcat/Spring strip `;` params; proxy routes full path |
| 5 | Backslash on Windows | `/admin\secret` | IIS treats `\` as `/`; proxy treats as literal |
| 6 | Case sensitivity mismatch | `/Admin/Secret` vs `/admin/secret` | Case-insensitive filesystem; case-sensitive router |
| 7 | Null byte injection | `/admin/secret%00.jpg` | Old: null terminates C string; framework reads full path |

**Proxy-specific behaviors (from http-desync):**

| Proxy | Quirk |
|-------|-------|
| NGINX | Merges `//` to `/`; handles `%2f` differently based on config |
| Apache | Allows path parameters via `;`; different alias behavior |
| HAProxy | Normalizes `..` segments before routing |
| AWS ALB | Strips certain headers; path handling differs from origin |
| Envoy | URL-decodes before routing; config-dependent normalization |
| Traefik | `stripPrefix` middleware creates path differential with backend |
| Caddy | Automatic HTTPS redirect can leak internal paths |

**Severity Guidance:** High when bypass leads to admin panel access, restricted API endpoints, or auth bypass. Path normalization differentials are the most consistently found bug class across reverse proxy setups.

---

## Filename Canonicalization

**What it is:** File upload systems validate filenames at upload time but process them differently at storage/execution time — creating TOCTOU (time-of-check-time-of-use) opportunities.

**Key patterns:**

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | Zero-width space | `shell​.php` (U+200B between `l` and `.`) | Validation sees `shell​.php` (fails extension check); storage strips U+200B → `shell.php` |
| 2 | Right-to-left override | `file\u202Ephp.txt` | Visually appears as `.txt`; actually `.php` |
| 3 | Double extension | `file.php.jpg` | Validation checks last extension; Apache processes first |
| 4 | Null byte (legacy) | `file.php%00.jpg` | Old: validation sees `.jpg`; filesystem truncates at null |
| 5 | Case folding | `file.PhP` | Case-insensitive execution; case-sensitive validation |
| 6 | Trailing spaces/dots | `file.php .` | Windows strips trailing space/dot; Linux preserves |
| 7 | Alternate data stream | `file.php::$DATA` | Windows NTFS ADS; bypasses some extension checks |

**Real-world:** CVE-2026-28289 (FreeScout, CVSS 10.0) — U+200B in attachment filename bypasses validation → stripped at storage → PHP webshell execution. Zero-click via email.

**Severity Guidance:** Critical when upload bypass leads to webshell deployment. Always test Unicode characters in filenames when file upload is in scope.

---

## SAML / XML Parser Differentials

**What it is:** SAML relies on XML parsing for assertions. Different XML libraries handle comments, entity expansion, namespace processing, and canonicalization differently — enabling signature wrapping and auth bypass.

**Key CVEs:**
- **CVE-2025-25291/25292** (ruby-saml): parser differentials in REXML vs Nokogiri enabled full auth bypass
- Pattern recurs across SAML libraries (saml2-js, SimpleSAMLphp, Spring Security SAML)

**Key patterns:**

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | XML comment injection | `<NameID>admin<!---->@evil.com</NameID>` | Signature check includes comment; app strips it |
| 2 | Entity expansion | `&xxe;` entity in assertion values | Entity resolved differently between signature verification and assertion processing |
| 3 | Namespace confusion | Duplicate namespace declarations with different values | Signature validates against one namespace; assertion processed with another |
| 4 | Canonicalization mismatch | Different C14N algorithms between IdP and SP | Signature valid under one canonicalization; assertion altered under another |

**Where to hunt:** Any application using SAML SSO. Test the SP's SAML response processing with manipulated assertions.

**Severity Guidance:** Critical — SAML auth bypass is consistently CVSS 9.0+ and enables full account takeover.

---

## JSON Parser Differentials

**What it is:** JSON parsers handle duplicate keys, large numbers, Unicode escapes, and comments differently.

**Key patterns:**

| # | Test | Payload | What to look for |
|---|------|---------|-----------------|
| 1 | Duplicate key override | `{"role":"user","role":"admin"}` | Auth check reads first key; backend reads last |
| 2 | Number precision | `{"id":9999999999999999999}` | JavaScript truncates to `10000000000000000000`; server uses exact |
| 3 | Unicode escape smuggling | `{"ro\u006ce":"admin"}` | WAF scans raw; parser decodes `\u006c` to `l` → "role" |
| 4 | Trailing comma | `{"role":"admin",}` | Some parsers accept; others reject → error path bypass |
| 5 | Comment injection | `{"role":"user"/*,"role":"admin"*/}` | JSON5-compatible parsers strip comments; strict parsers reject |

**Severity Guidance:** Medium-High. Duplicate key and Unicode escape patterns are the most commonly exploitable. Test API endpoints that pass through multiple JSON parsing layers.

---

## Cache Key vs. Origin Differentials

**What it is:** CDN/cache computes a cache key from a subset of the request; origin processes the full request. Any parameter, header, or path component excluded from the cache key but processed by the origin creates a deception opportunity.

See [http-desync skill](../../http-desync/SKILL.md) for full cache poisoning and web cache deception patterns.

**Quick tests:**

| # | Test | What to look for |
|---|------|-----------------|
| 1 | Unkeyed headers (X-Forwarded-Host, X-Original-URL) | Cache serves poisoned response to other users |
| 2 | Path extension appending (`/account/settings.css`) | Cache treats as static; origin returns dynamic user data |
| 3 | Query parameter pollution | Cache ignores unknown params; origin processes them |
| 4 | Fragment handling (`#fragment`) | Browser strips fragment; CDN may include in cache key |

---

## Side-Channel Differentials

**What it is:** Timing, error message, and response size differences that reveal information about server-side parsing behavior. Named "the defining trend of 2025" in PortSwigger's Top 10.

**Key patterns:**

| # | Test | Technique | What to look for |
|---|------|-----------|-----------------|
| 1 | Timing-based user enum | Valid vs invalid username in login | Measurable timing difference (bcrypt on valid, fast reject on invalid) |
| 2 | Error message differentials | Invalid param types in API requests | Different error messages reveal backend parser, framework, or DB |
| 3 | Response size as oracle | Blind injection with `AND 1=1` vs `AND 1=2` | Byte-level response size difference confirms injection |
| 4 | Cache timing oracle | Request known-cached vs uncached resource | Timing difference reveals whether resource exists |
| 5 | ReDoS-based content detection | Regex backtracking on specific input patterns | Timing difference reveals regex presence and input matching |

**Severity Guidance:** Medium alone but High when chained. Side channels enable blind exploitation of injection vulnerabilities and information disclosure that feeds further attacks.

---

## Where to Hunt

Parser differentials exist wherever multi-component architectures process attacker-controlled input. Highest-value targets:

1. **Reverse proxy → application** — Path normalization, header handling, body parsing
2. **WAF → origin** — Unicode normalization, encoding, content-type
3. **CDN/cache → origin** — Cache key computation vs. full request processing
4. **SAML IdP → SP** — XML canonicalization, comment handling
5. **API gateway → microservice** — URL parsing, header forwarding, body transcoding
6. **File upload → storage → execution** — Filename canonicalization, MIME type
7. **H2 proxy → H1 backend** — Protocol downgrade, header mapping, CONNECT

**Hunting heuristic:** Identify all parsing layers in the request path. For each layer boundary, test: does this input parse the same way on both sides? If not, you have a parser differential — now determine if it's exploitable.
