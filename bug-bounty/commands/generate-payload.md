---
name: generate-payload
description: Generate context-aware security testing payloads — encoded, WAF-aware, and ready to use
arguments:
  - name: vuln_type
    description: "Vulnerability type and optional constraints (e.g., 'XSS with Cloudflare WAF', 'SQLi MySQL blind')"
    required: true
---

# /generate-payload

Generate security testing payloads tailored to your target's stack, WAF, and constraints. Outputs ready-to-paste payloads with encoding strategies and injection points.

## Usage

```
/generate-payload <vuln type> [WAF, framework, encoding constraints, blocked characters]
```

Generate payloads for: $ARGUMENTS

---

## What I Need From You

**Minimum:** Vulnerability type (e.g., "XSS", "SQLi", "SSRF").

**Better:** Vuln type + context (e.g., "XSS on a React app behind Cloudflare, angle brackets blocked").

**Best:** Vuln type + target details + constraints (e.g., "Blind SQLi on PostgreSQL, behind Akamai, single quotes blocked, 256 char limit, injection in a cookie value").

---

## Output Format

For each payload (3-5 per request):

```markdown
### Payload N: [Name/Strategy]

**Payload:**
```
[ready to copy-paste]
```

**Encoding strategy:** [Why this bypasses the filter]
**Injection point:** [URL param, POST body, header, cookie, JSON value, filename]
**Expected response if successful:** [Alert fires, time delay, DNS callback, error, data in response]
**Notes:** [Edge cases or prerequisites]
```

---

## Supported Vulnerability Classes

| Class | Subtypes | Key Bypass Techniques |
|-------|----------|----------------------|
| **XSS** | Reflected, stored, DOM, blind | Event handlers w/o brackets, mutation XSS, JS URI schemes, template literals |
| **SQLi** | Error-based, union, blind boolean, blind time, OOB | Comment insertion, case variation, CAST/CONVERT, conditional expressions |
| **SSRF** | Cloud metadata, internal service, protocol smuggling | Decimal IP, IPv6, DNS rebinding, redirect chains, gopher:// |
| **SSTI** | Jinja2, Twig, Freemarker, ERB, Pebble | `__class__.__mro__` (Jinja2), `_self.env` (Twig), Runtime exec (Pebble) |
| **Command injection** | Direct, blind, OOB | Backticks, `$()` substitution, `%0a` newline, `${IFS}` for spaces |
| **Path traversal** | Relative, absolute, null byte | `....//`, URL encoding, overlong UTF-8, null byte truncation |
| **JWT** | Algorithm confusion, claim tampering | `alg:none`, HS256/RS256 confusion, `kid` injection, JKU spoofing |
| **CRLF** | Header injection, response splitting | `%0d%0a`, Unicode line separators, encoded variants |
| **Open redirect** | Parameter-based, path-based | `//evil.com`, `\/evil.com`, protocol-relative, `@` in URL |
| **NoSQL injection** | Operator injection, JavaScript injection | `$gt`, `$regex`, `$where`, `$ne` operator payloads |

### SSTI Detection Quick Reference

| Engine | Probe | Success |
|--------|-------|---------|
| Jinja2 / Twig | `{{7*7}}` | Returns `49` |
| Freemarker | `${7*7}` | Returns `49` |
| ERB | `<%= 7*7 %>` | Returns `49` |
| Smarty | `{7*7}` | Returns `49` |

---

## WAF Bypass Strategies

When a WAF blocks standard payloads, apply encoding chains:

| Technique | What It Does | Example |
|-----------|-------------|---------|
| Double URL encode | Encode the `%` itself | `%253Cscript%253E` |
| Unicode normalization | Equivalent Unicode chars | `\uFF1Cscript\uFF1E` (fullwidth) |
| HTML entities | Numeric/named entities | `&#x3C;script&#x3E;` |
| Case variation | Mixed case to dodge regex | `<ScRiPt>`, `SeLeCt` |
| Comment insertion | Break keyword detection | `SEL/**/ECT`, `UN/**/ION` |
| Whitespace alternatives | Non-standard whitespace | `\t`, `\n`, `\x0c` between SQL keywords |
| Content-Type confusion | Unexpected encoding | JSON payload with `application/xml` header |
| Parameter pollution | Duplicate parameters | `?q=safe&q=<script>` — server may use last value |
| Null byte injection | Break string parsing | `%00` between filtered terms |
| Chunked transfer | Fragment the payload | Split across HTTP chunks to evade pattern matching |

### WAF-Specific Notes

| WAF | Common Weakness | Approach |
|-----|----------------|----------|
| Cloudflare | Mutation XSS patterns | DOM clobbering, SVG event handlers |
| Akamai | Unicode normalization gaps | Fullwidth chars, overlong UTF-8 |
| AWS WAF | Rule set gaps in custom rules | Uncommon HTTP methods, header injection |
| ModSecurity CRS | Paranoia level thresholds | Encoding chains below default PL |
| Imperva | Event handler gaps | `onfocus`+`autofocus`, `onbeforetoggle` |

---

## Constraints

Specify any of these for tailored payloads:

| Constraint | Example | Effect |
|------------|---------|--------|
| **Blocked characters** | `< > ' "` blocked | Event handlers, template literals, backtick strings |
| **Max length** | 128 chars | Shorter payloads, external script loading |
| **Framework** | React, Angular, Django | Framework-specific sinks and bypass patterns |
| **WAF vendor** | Cloudflare, Akamai | Vendor-specific evasion techniques |
| **Database** | PostgreSQL, MySQL, MSSQL | DB-specific functions and syntax |
| **Input location** | Cookie, header, filename | Encoding and delivery adapted to context |
| **Content type** | JSON, XML, multipart | Payload formatted for the content type |

---

## Design Principles

- **Copy-paste ready** — Every payload can be used immediately without modification
- **Context-aware** — Payloads adapt to the WAF, framework, and constraints you specify
- **Escalation order** — Start with detection/confirmation, then escalate to impact demonstration
- **Bypass-first** — If you mention a WAF or filter, every payload accounts for it
- **Legal testing only** — All payloads are for authorized security testing within a bug bounty scope

---

## Tips

1. **Start with detection** — Confirm the vuln class before crafting full exploit payloads
2. **Specify the WAF** — A payload that works on bare metal will fail behind Cloudflare
3. **Mention blocked chars** — "Quotes are filtered" dramatically changes the payload strategy
4. **Include the database** — SQLi payloads differ significantly across PostgreSQL, MySQL, and MSSQL
5. **Use out-of-band** — When you can't see the response, use DNS/HTTP callbacks to confirm execution
6. **Chain with /quick-test** — Generate payloads here, then use `/quick-test` for the full test plan
