# Prioritization Rubrics

Shared scoring frameworks used across hunt planning, triage, and target selection.

---

## Hunt Target Scoring

Used by: `/hunt-plan`, `hunt-session` agent, `/triage-findings`

```
Score = (Bounty Value × Find Probability) / (Competition Level × Time Investment)

Where:
- Bounty Value: expected payout for the vuln class on this program (use reward table)
- Find Probability: how likely based on tech stack and disclosed patterns
  - High: tech stack known-vulnerable, no disclosed fixes, similar bugs paid before
  - Medium: standard tech, some disclosed fixes, moderate testing history
  - Low: hardened target, extensive disclosure history, security-mature org
- Competition Level:
  - Low (1.0): new program, niche tech, <20 active researchers
  - Medium (1.5): established program, common tech, 20-100 researchers
  - High (2.0): popular program, mainstream tech, 100+ researchers
- Time Investment: estimated hours to test this area (1-8 scale)
```

## Automation Resistance Tiers

Used by: `vuln-patterns` skill, `hunt-session` agent

| Tier | Focus Level | Description |
|------|------------|-------------|
| **A: Human-Only** | Prioritize | Business logic, multi-tenant isolation, payment flows, subscription bypass |
| **B: Human-Advantaged** | Good ROI | Auth chains, complex SSRF, race conditions with business impact, AI/LLM agent exploitation |
| **C: Commodity** | Deprioritize | Basic XSS/SQLi/SSRF, subdomain takeovers, missing headers, known CVE patterns |

> **Key stat:** Business logic is 45% of all bounty awards (Intigriti 2026). Focus on Tier A.

## Finding Reportability Assessment

Used by: `/triage-findings`, `report-review` agent

| Factor | Weight | High Score | Low Score |
|--------|--------|-----------|-----------|
| **Exploitability** | 30% | Working PoC, no conditions | Theoretical, requires specific setup |
| **Impact** | 25% | ATO, data breach, financial loss | Info disclosure of non-sensitive data |
| **Novelty** | 15% | New technique, no disclosed patterns | Common vuln class, many disclosures |
| **Duplicate Risk** | 15% | Low competition, niche feature | Popular target, common endpoint |
| **Report Quality** | 15% | Clear repro, concrete impact, CVSS justified | Vague steps, theoretical impact |

## Severity Calibration

Quick reference for honest CVSS scoring:

| What You Found | Realistic Severity | Common Inflation |
|----------------|-------------------|-----------------|
| Read other user's public profile data | Low (3.1-3.9) | Medium |
| Read other user's private data (email, phone) | Medium-High (5.3-7.5) | Critical |
| Modify other user's data | High (7.5-8.5) | Critical |
| Full account takeover | Critical (9.0-9.8) | — (usually accurate) |
| Self-XSS without delivery | Informative (0) | Medium |
| Reflected XSS with CSP bypass | Medium-High (5.4-6.1) | Critical |
| SSRF reading cloud metadata | High (7.5-8.6) | Critical with "RCE possible" |
| Open redirect without chain | Low (3.1-4.3) | Medium |

> **Rule:** Score what the finding **demonstrates**, not what you hope the impact could be.
