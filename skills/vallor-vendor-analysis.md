---
name: vendor-analysis
description: >
  Deep-dive analysis of a single vendor — financial health (10-Q / 10-K / D&B,
  ratings, recent earnings), operational performance (delivery, quality, NPS),
  technical fit (capability gap, roadmap alignment), risk (cyber, geographic,
  regulatory, ESG), and opportunity (consolidation, expansion, renegotiation).
  Outputs HTML report + Excel + executive recommendation. Use when the user says
  "vendor analysis", "supplier deep dive", "tell me about [vendor]", "review
  vendor X", "is X a strategic partner", or before a major renewal / consolidation
  / risk-review decision.
argument-hint: "<vendor-name> [--depth quick|standard|deep] [--lens financial|operational|risk|all]"
---

# /vallor:vendor-analysis

## Purpose

Before a renewal, a consolidation decision, an audit, or an executive briefing,
the question is the same: who is this vendor, really? This skill produces a
360-degree view.

## Load context

1. Read `~/.claude/plugins/config/vallor/CLAUDE.md`:
   - `## Vendor / supplier classification` — what tier is this vendor today?
   - `## Risk management → Supply risk dimensions` — what we always track
   - `## Categories under management` — which categories does this vendor serve?
2. Read the matter / vendor folder for prior reviews, contract briefs,
   negotiation history.
3. Apply the **/vallor:design** skill.

## Ask the user for context (optional but encouraged)

Use the `AskUserQuestion` tool to gather optional context that materially changes the analysis. A vendor deep-dive for a renewal is different from one for a consolidation decision.

**Up front (before research):**
Ask 1-3 short questions when not obvious from the inputs and profile:
- Decision driving this analysis: renewal / consolidation / risk review / audit response / new-vendor evaluation
- Depth (if not via `--depth`): quick (one-pager) / standard / deep (full report + appendices)
- Lens (if not via `--lens`): financial / operational / risk / strategic fit / all
- Specific concerns the user already has (e.g., recent missed SLA, leadership change, M&A rumor)
- Time horizon: 12-month decision / 3-year strategic / immediate (today)

**Mid-flight (after the profile and initial research):**
Once you've pulled financials and operational data, surface concrete asks before guessing on:
- Conflicting signals (e.g., strong financials, weak delivery performance) — emphasize which
- Whether to model alternative scenarios (continue / consolidate / split / exit)
- Whether to include a recommendation or stay neutral (some users want the data, not the call)
- Cybersecurity / compliance depth — surface-level certifications or pull into a separate sub-analysis

**Rules:**
- Every question must offer a reasonable default the user can pick to skip ahead.
- Ask follow-ups when answers reveal more depth or a non-default choice.
- Cap a single round at ~3 questions.
- Skip when the user has signaled a fast / default run.

## Workflow

### Step 1 — Profile

Build the headline profile:
- Legal entity, HQ, geographic footprint
- Public vs private, parent company
- Annual revenue, headcount
- Categories supplied to us
- Annual spend with us (from spend-analysis if available)
- Our share of their book (rough estimate from public data)

### Step 2 — Financial health

Pull and synthesize:
- Last 4 quarters of earnings (if public)
- D&B Paydex, credit rating, recent rating actions
- Recent acquisitions, divestitures, leadership changes
- Stock price / analyst sentiment (if public)
- Recent news (last 12 months) — material events

Tag financial pressure points and growth areas that affect our relationship.

### Step 3 — Operational performance

For each category they supply:
- Delivery on-time-in-full (OTIF) trend
- Quality (PPM, returns, audit findings)
- Service incident history (if services / SaaS)
- SLA compliance (if applicable)
- Responsiveness (escalation cycle time)

Pull from CLM, ERP, ticketing systems if connected; otherwise ask the user for
the data file.

### Step 4 — Technical / capability fit

- Their stated capabilities vs what we use
- Roadmap alignment (their published roadmap vs our planned needs)
- Capability gaps (areas where they're not the best — and an alternative is
  growing)
- Innovation pipeline (R&D spend, patents, new products)

### Step 5 — Risk

For each of our standard risk dimensions:

| Dimension | Signal | Score (1-5) | Note |
|---|---|---|---|
| Financial | [D&B + earnings trend] | | |
| Geographic / geopolitical | [HQ + sourcing footprint] | | |
| Quality | [PPM trend, recalls] | | |
| Cybersecurity | [SOC 2 / ISO 27001 / breaches] | | |
| ESG / regulatory | [scope 1-3, EUDR, conflict-minerals, etc.] | | |
| Concentration (our share of their book) | [...] | | |
| Concentration (their share of our spend) | [...] | | |

### Step 6 — Opportunity

Three types:
- **Consolidate** — they could absorb work we have spread across other vendors.
- **Renegotiate** — they're in a weak position; we have leverage now.
- **Diversify** — they're a single-source risk; we should develop a backup.

Each opportunity scored: estimated dollar impact × effort × time.

### Step 7 — Recommendation

A single executive recommendation:
- **Keep + grow** (strategic partner)
- **Maintain** (preferred supplier; nothing material to change)
- **Renegotiate** (open the contract; specific levers)
- **Diversify** (develop secondary source; reduce exposure)
- **Exit** (transition off; timeline)

## Output

Save to:
`~/.claude/plugins/config/vallor/vendor-analyses/<YYYY-MM-DD>-<vendor-slug>/`

Files:
- `report.html` — interactive multi-section HTML (Overview · Financial ·
  Operational · Risk · Opportunity · Recommendation). Chart.js for charts.
- `data.xlsx` — the underlying tables, sortable, finance-friendly.
- `recommendation.md` — one-page executive memo.

### HTML report structure

Single page, six anchored sections (not tabs — this is a read-top-to-bottom doc):

1. **Profile** — masthead + headline cards (annual spend, our share of their
   book, tenure, current tier).
2. **Financial health** — earnings line chart, key metrics, recent events
   timeline.
3. **Operational performance** — OTIF / quality trend lines, SLA compliance
   bars.
4. **Technical fit** — capability radar (us-required vs vendor-delivered).
5. **Risk** — heat map across dimensions.
6. **Recommendation** — bordered callout with the recommendation and a single
   "What I'd do today" line.

## Sources

Every external claim is sourced. Numbered list at the end. URL + access date.
Tag confidence (`[verified]` / `[reported]` / `[inferred]`).

If a research connector (Westlaw, CourtListener, etc.) isn't available, web
search is used and tagged. If neither is available, the report explicitly says
so in the reviewer note — no silent supplementation.

## Decision tree after delivery

1. **Open the renegotiation** — `/vallor:negotiation-report` builds
   the prep packet.
2. **Run consolidation** — `/vallor:vendor-consolidation` looks
   for overlap across the vendor base.
3. **Update the procurement profile** — promote / demote the tier if the
   analysis warrants.
4. **Build the savings tracker** — log the opportunity into the savings
   pipeline.
5. **Stop here** — analysis is saved.

## Examples

```
/vallor:vendor-analysis Acme
```

```
/vallor:vendor-analysis Megacorp --depth deep --lens risk
```
