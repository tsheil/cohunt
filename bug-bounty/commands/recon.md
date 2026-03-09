---
name: recon
argument-hint: "[target-domain]"
description: Run reconnaissance on a target — fingerprint tech stack, enumerate subdomains, detect WAF/CDN, and map the attack surface
arguments:
  - name: target
    description: "Target domain to recon (e.g., 'example.com')"
    required: true
---

# /recon

Run full external reconnaissance on a web target. This command triggers the `target-recon` skill and produces a structured recon report.

## Usage

```
/recon <target domain>
```

Recon target: $ARGUMENTS

---

## What This Does

Runs a complete external recon workflow against the target:

1. **HTTP fingerprinting** — headers, cookies, server software, error pages
2. **Technology detection** — framework, CMS, libraries, language
3. **Subdomain enumeration** — crt.sh, DNS lookups, status checks
4. **Security headers audit** — CSP, CORS, HSTS, X-Frame-Options
5. **Common paths** — robots.txt, sitemap.xml, .well-known/security.txt
6. **WAF/CDN detection** — provider identification, bypass potential

Use the `target-recon` skill to execute this recon. Follow its full execution flow and output format.

---

## Output

Produce the structured recon report defined in the `target-recon` skill, including:
- Target profile (IP, hosting, CDN, WAF)
- Technology stack table
- Security headers assessment
- Subdomain inventory
- Interesting paths
- Attack surface summary with prioritized next steps

---

## Tips

- **Root domains cast a wider net** — "example.com" finds more than "www.example.com"
- **Chain with /hunt-plan** — recon first, then `/hunt-plan` to turn findings into action
- **Mention the program** — "for their HackerOne program" helps scope the work
- **Ask for depth** — add "deep" or "quick" to adjust thoroughness
