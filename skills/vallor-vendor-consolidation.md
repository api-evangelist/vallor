---
name: vendor-consolidation
description: >
  Identifies consolidation opportunities across the active vendor base —
  overlapping scopes, dual-source savings, single-source risk that should be
  split. Cross-walks spend data, category data, and vendor analyses to produce
  a consolidation register with recommended moves (consolidate from N to M
  suppliers, develop a secondary source, exit a vendor). Outputs HTML dashboard
  and Excel register. Use when the user says "vendor consolidation", "consolidate
  suppliers", "reduce supplier base", "consolidation opportunities", "supplier
  rationalization", "where do we have overlap", or after a spend analysis surfaces
  high supplier count in a category.
argument-hint: "[--category <category>] [--mode consolidate|diversify|both] [--target-reduction <pct>]"
---

# /vallor:vendor-consolidation

## Purpose

A 1,800-supplier base usually contains 200-400 vendors that could be
consolidated into 60-120 with no operational impact and meaningful price
benefits — and 10-30 single-source vendors that should be split for resilience.
This skill finds both.

## Load context

1. Read `~/.claude/plugins/config/vallor/CLAUDE.md`:
   - `## Vendor / supplier classification → Consolidation principle`
   - `## Categories under management`
   - `## Risk management → Single-source register`
2. Load the most recent spend analysis if one exists. If not, prompt:
   > "I need a spend dataset to find consolidation opportunities. Run
   > `/vallor:spend-analysis <spend-file>` first, or paste the
   > vendor list with annual spend by category."
3. Apply the **/vallor:design** skill.

## Ask the user for context (optional but encouraged)

Use the `AskUserQuestion` tool to gather optional context that materially changes which consolidations to recommend. Consolidation calls are political — calibration matters.

**Up front (before building the candidate list):**
Ask 1-3 short questions when not obvious from the inputs:
- Scope (if not via `--category`): all categories / single category / a named list
- Mode (if not via `--mode`): consolidate (reduce supplier count) / diversify (split single-sources) / both
- Target supplier-count reduction band (if not via `--target-reduction`): 10-20% / 20-30% / 30-50% / aggressive
- Political constraints: untouchable vendors (incumbent relationships, parent-company affiliates), required exits
- Time horizon: 12 months / 3 years / strategic (5+ years)
- Risk appetite: prefer resilience (more suppliers) / prefer leverage (fewer suppliers)

**Mid-flight (after building the candidate list):**
Once you've built the candidate consolidations, surface concrete asks before guessing on:
- Groups where the consolidation rationale is weak (overlap is partial, switching cost high) — keep, drop, or flag
- Single-source vendors where the user may already have a planned dual-source
- How aggressive to be on "preferred" tier consolidations vs "strategic" tier
- Whether to model headline savings vs run sensitivity (best case / base / worst)

**Rules:**
- Every question must offer a reasonable default the user can pick to skip ahead.
- Ask follow-ups when answers reveal more depth or a non-default choice.
- Cap a single round at ~3 questions.
- Skip when the user has signaled a fast / default run.

## Workflow

### Step 1 — Build the consolidation candidate list

For each category in the spend file:
- Group vendors by sub-category / SKU type / service domain.
- Identify groups with 3+ vendors covering similar scopes.
- For each group, surface:
  - Total spend across the group
  - Spend per vendor (Pareto)
  - Overlap factor (how much of vendor A's work could vendor B do)
  - Switching cost estimate

### Step 2 — Build the diversification candidate list

For each single-source vendor:
- Annual spend with us
- Category criticality (Kraljic quadrant)
- Geographic concentration risk
- Their financial health (cross-reference vendor-analysis if available)
- Alternative suppliers in the market

### Step 3 — Score and rank

Consolidate scoring (per group):
- Dollar opportunity (estimated price reduction × consolidated spend)
- Operational simplicity (fewer POs, fewer audits, less governance)
- Risk delta (does consolidating raise single-source risk?)
- Effort (qualification, switching cost, change management)

Diversify scoring (per single-source vendor):
- Risk reduction (what's the exposure if they fail)
- Cost premium (how much will dual-source actually cost)
- Time to qualify (months)
- Geographic / political reasoning

### Step 4 — Recommend moves

For each opportunity, produce a recommended move:
- **Consolidate from N to M** — name the M suppliers, allocation %, switching
  plan, estimated savings, timeline.
- **Develop secondary** — name the target secondary, allocation %, qualification
  plan, estimated incremental cost, risk reduction.
- **Exit** — name the vendor to exit, transition plan, sunk cost recovery.

### Step 5 — Risk overlay

Highlight any move that:
- Reduces supplier count BELOW the practice profile's bid-floor for that
  category.
- Concentrates spend with a vendor whose financial health is concerning.
- Eliminates a relationship that was preserved for political / relationship
  reasons (flag for human review before executing).

## Output

Save to:
`~/.claude/plugins/config/vallor/consolidation-analyses/<YYYY-MM-DD>/`

Files:
- `dashboard.html` — interactive multi-tab view (Overview · Consolidate ·
  Diversify · Risk overlay · Roadmap).
- `register.xlsx` — flat consolidation register, finance-friendly.
- `executive-summary.md` — one-page recommendation.

### HTML dashboard tabs

**Tab 1 — Overview.** Big-number cards:
- Active suppliers
- Categories analyzed
- Estimated savings from consolidation
- Estimated diversification investment
- Net opportunity

**Tab 2 — Consolidate.** Sortable table of consolidation candidates. Each row
clickable to a detail card with the recommended move, the named target
suppliers, the timeline.

**Tab 3 — Diversify.** Sortable table of single-source vendors with risk score
and recommended action.

**Tab 4 — Risk overlay.** Heat map: category × risk dimension. Categories where
consolidation would raise risk are flagged.

**Tab 5 — Roadmap.** 12-month Gantt-style plan with moves prioritized by
opportunity × effort.

## Output rules

- Standalone HTML, Chart.js via CDN.
- Sanitize vendor names from the spend file (`textContent` only).
- All monetary values in the practice profile's base currency.
- Print-friendly.

## Decision tree after delivery

1. **Build the sourcing strategy** for a specific consolidation move —
   `/vallor:sourcing-strategy`.
2. **Open the renegotiation** — `/vallor:negotiation-report` for
   the consolidator's incumbent.
3. **Develop the secondary source** — `/vallor:rfq-creation` for
   the qualification event.
4. **Update the procurement profile** — append findings to the single-source
   register.
5. **Stop here** — analysis is saved.

## Examples

```
/vallor:vendor-consolidation
```

```
/vallor:vendor-consolidation --category "Logistics" --target-reduction 30
```
