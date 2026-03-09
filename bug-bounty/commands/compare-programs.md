---
name: compare-programs
description: Compare bug bounty programs side-by-side — rewards, scope, response times, competition — and get a recommendation on where to hunt
arguments:
  - name: programs
    description: "Programs to compare (e.g., 'Shopify vs GitHub' or 'Uber vs Airbnb vs Lyft')"
    required: true
---

# /compare-programs

Compare two or more bug bounty programs side-by-side to decide where to invest your hunting time. Evaluates rewards, scope breadth, response metrics, competition level, and historical payouts.

## Usage

```
/compare-programs <program 1> vs <program 2> [vs <program 3>]
```

Compare: $ARGUMENTS

---

## How It Works

```
┌──────────────────────────────────────────────────────────────┐
│                   COMPARE PROGRAMS                            │
├──────────────────────────────────────────────────────────────┤
│  ✓ Research each program via web search                      │
│  ✓ Side-by-side comparison matrix                            │
│  ✓ Reward range comparison by severity tier                  │
│  ✓ Scope breadth and asset diversity                         │
│  ✓ Response time and triage quality assessment               │
│  ✓ Competition and saturation estimate                       │
│  ✓ ROI recommendation based on hunter profile                │
└──────────────────────────────────────────────────────────────┘
```

---

## What I Need From You

**Required:** At least two program names or domains to compare.

**Helps a lot:**
- Your specialization: "I'm good at IDOR and API bugs"
- Time budget: "I have 10 hours this week"
- Experience level: "I've been hunting for 6 months"
- Platform preference: "I prefer HackerOne"

---

## Output

```markdown
## Program Comparison: [Program A] vs [Program B]

**Generated:** [Date]
**Hunter Profile:** [If provided]

---

### Side-by-Side

| Factor | [Program A] | [Program B] |
|--------|-------------|-------------|
| **Platform** | [HackerOne/Bugcrowd/etc.] | [Platform] |
| **Type** | [Public/Private/VDP] | [Type] |
| **Safe Harbor** | [Yes/Partial/No] | [Status] |
| **Scope Size** | [# of assets] | [# of assets] |
| **Critical Bounty** | [$range] | [$range] |
| **High Bounty** | [$range] | [$range] |
| **Medium Bounty** | [$range] | [$range] |
| **Low Bounty** | [$range] | [$range] |
| **Avg. Response Time** | [days] | [days] |
| **Avg. Triage Time** | [days] | [days] |
| **Disclosed Reports** | [count] | [count] |
| **Competition** | [High/Medium/Low] | [Level] |
| **Tech Stack** | [Key technologies] | [Technologies] |

---

### Reward Analysis

| Severity | [Program A] | [Program B] | Edge |
|----------|-------------|-------------|------|
| Critical | $[range] | $[range] | [Winner] |
| High | $[range] | $[range] | [Winner] |
| Medium | $[range] | $[range] | [Winner] |
| Low | $[range] | $[range] | [Winner] |

**Bonus Programs / Special Events:** [Any active bonus multipliers]

---

### Scope Comparison

**[Program A]:**
- [Key in-scope assets and what makes them interesting]
- [Notable exclusions]

**[Program B]:**
- [Key in-scope assets and what makes them interesting]
- [Notable exclusions]

**Edge:** [Which offers more attack surface or better scope coverage]

---

### What Pays (Disclosed Reports Analysis)

**[Program A] — Top paid vuln types:**
1. [Vuln type] — [typical bounty]
2. [Vuln type] — [typical bounty]

**[Program B] — Top paid vuln types:**
1. [Vuln type] — [typical bounty]
2. [Vuln type] — [typical bounty]

---

### Competition Assessment

**[Program A]:** [Analysis — how saturated, how many active hunters, what areas are over-tested]
**[Program B]:** [Analysis]

---

### Recommendation

**Hunt on: [Winner]**

**Why:**
- [Key factor 1]
- [Key factor 2]
- [Key factor 3]

**If you specialize in [user's focus]:** [Tailored recommendation]
**If you have limited time ([user's budget]):** [Time-optimized recommendation]

**Alternative strategy:** [e.g., "Split time: 70% on Program A for higher payouts, 30% on Program B for easier wins"]
```

---

## Execution Flow

### Step 1: Parse Request

```
Identify programs to compare:
- "Shopify vs GitHub" → Two-program comparison
- "Uber vs Lyft vs DoorDash" → Three-way comparison
- "Compare HackerOne programs paying > $10k for criticals" → Filtered comparison
```

### Step 2: Research Each Program

```
For each program, use the program-research skill to gather:
1. Program profile (platform, type, launch date)
2. Scope (assets, breadth, restrictions)
3. Reward structure (bounty ranges by severity)
4. Response metrics (triage time, resolution time)
5. Disclosed reports (top vuln types, payout patterns)
6. Community reputation (hunter feedback)
```

### Step 3: Normalize and Compare

```
1. Align data into comparable format
2. Calculate reward-per-effort estimates
3. Assess competition level for each
4. Factor in hunter's stated specialization
5. Consider time budget constraints
```

### Step 4: Recommend

```
1. Score each program: reward × scope × responsiveness ÷ competition
2. Factor in hunter-specific advantages
3. Generate actionable recommendation
4. Suggest time allocation if splitting effort
```

---

## Examples

```
/compare-programs Shopify vs GitHub
/compare-programs Uber vs Lyft vs DoorDash
/compare-programs GitLab vs GitHub — I specialize in IDOR and have 8 hours
/compare-programs Cloudflare vs Fastly — I'm good at cache and HTTP desync bugs
```

---

## Tips

1. **Mention your specialty** — the best program depends on what bugs you find
2. **Include time budget** — a rich program with slow triage may not be worth it if you only have a few hours
3. **Consider your experience** — some programs are more beginner-friendly than others
4. **Look beyond top bounties** — consistent medium payouts with fast triage can beat rare criticals with months of waiting
5. **Chain with /hunt-plan** — after choosing, immediately build a plan for the winner
