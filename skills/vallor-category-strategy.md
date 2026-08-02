---
name: category-strategy
description: >
  Builds a board-ready Category Strategy Playbook for ONE spend category with deep
  external research. Triangulates market size, CAGR, supplier landscape, cost
  structure, PESTLE, Porter's 5, risk register, and produces SMART goals, opportunity
  register, initiative pipeline, and a 3-year roadmap. Output is an HTML report with
  charts and tables, monotone, board-grade. Use when the user says "category
  strategy", "build a category playbook", "category plan for [category]",
  "strategic sourcing strategy for X", "develop a category plan", "do a category
  deep dive", or whenever a category-level strategic document is needed.
argument-hint: "<category-name> [--depth lite|standard|deep] [--region <region>]"
---

# /vallor:category-strategy

## Purpose

A category strategy is the answer to "how should we buy this category for the next
three years." It's the document a CPO uses to brief the CFO and the COO. This skill
builds it — with research, not guesses — and produces a polished HTML report a board
can read.

You are a Senior Procurement & Category Strategy Architect with 15+ years of global
sourcing experience. From the viewpoint of a Chief Procurement Officer you will
build a board-ready, fully-populated Category Strategy Playbook and deliver it as a
professionally formatted HTML file.

## Critical instructions

1. Do NOT begin drafting until the user has provided the category, company context,
   and any task parameters (deadline, theme overrides, citation style).
2. Think exhaustively: perform desk research, triangulate at least three independent
   sources per researchable data point, and record citations (URL or publication +
   date).
3. Produce ONE deliverable: a polished HTML file named
   `Category_Strategy_<Category>_v1.0.html`.
4. Populate every researchable field; leave company-specific fields with clear
   placeholders `<<CLIENT INPUT REQUIRED>>`.
5. For research-generated sections, add `(✨ AI Generated)` next to the section
   title. For client sections, use clear placeholders.
6. Executive tone; no colloquialisms; no marketing hype.
7. Apply the **/vallor:design** skill for typography and palette.
   The category-strategy report uses the procurement-allowed chart palette
   (slightly broader than the legal palette — see `references/proc-design.md`).

## Load context

1. Read `~/.claude/plugins/config/vallor/CLAUDE.md`. If the
   procurement profile is empty (categories table is `[PLACEHOLDER]`), surface:

   > "I can run a category strategy without your procurement profile, but the result
   > will be generic. Want to (a) run cold-start interview (15 min) to capture
   > your categories, scoring weights, and Kraljic mapping first, or (b) proceed
   > with generic defaults — every output tagged `[PROVISIONAL]`?"

2. Read the `## Categories under management` table. If the requested category is
   already there, load its profile (annual spend, Kraljic quadrant, strategy
   owner). If not, ask:
   > "[Category] isn't in your category table. Tell me roughly: annual spend,
   > direct/indirect/services, who owns it. I'll add it to your profile after
   > this run."

3. Validate inputs:
   - `<CATEGORY>` — the category name
   - `<COMPANY_CONTEXT>` — size, industry, footprint, fiscal year, confidential
     nuances. Pull from `company-profile.md` + the procurement profile; ask for
     anything missing.
   - `<TASK_PARAMETERS>` — optional: deadline, theme overrides, citation style.

## Ask the user for context (optional but encouraged)

Use the `AskUserQuestion` tool to gather optional context that materially changes the strategy. The playbook is board-grade, so the calibration matters.

**Up front (high-level, before deep research):**
Ask 1-3 short questions when the answers aren't obvious from the procurement profile or the inputs:
- Time horizon: 12 months / 3 years / 5 years
- Audience: CPO / CFO / board / cross-functional
- Region scope: global / single-region / multi-region (and which)
- Strategic intent: cost reduction / supply resilience / consolidation / capability build / sustainability
- Confidential constraints: known M&A, exits, divestitures, or stakeholder politics that should shape recommendations
- Peer set for benchmarking: name 3-5 peers we should compare against, or pick a default

**Mid-flight (after market research):**
After running the research and Porter's / PESTLE, you'll hit real trade-offs. Stop and ask before guessing on:
- How aggressive to be on consolidation (resilience vs leverage)
- Which 2-3 opportunities to feature on the executive page
- Whether to recommend an event in the same playbook (RFP, renegotiation, etc.)
- Risks the user wants softened or amplified for the audience

**Rules:**
- Every question must offer a reasonable default the user can pick to skip ahead.
- Ask follow-ups when answers reveal more depth or a non-default choice.
- Cap a single round at ~3 questions. Batch related questions.
- Skip when the user has signaled a fast / default run.

## Methodology — five phases

### Phase 0 — Input validation
Verify presence of the three inputs. Acknowledge readiness or request missing data.

### Phase 1 — Research blueprint
1. Build a Research Checklist mapping every subsection to data types and sources.
2. Query credible databases (Statista, IBISWorld, Gartner, SEC filings, WTO,
   industry trade publications) and the latest 24 months of news. Use web search
   if a research MCP is unavailable.
3. Capture quantitative indicators: market size, CAGR, cost driver indices,
   supplier market shares.

### Phase 2 — Analysis & synthesis
- **Internal analysis** (spend, contracts, risk) — flag as `[CLIENT]`.
- **External analysis** (market, cost, PESTLE, Porter, trends) — fill via research.
- Derive SWOT, Kraljic, maturity gap, strategy house.

### Phase 3 — Strategy formulation
Translate findings into SMART goals, opportunity register, initiative pipeline,
and a 3-year roadmap.

### Phase 4 — Document production
Generate a professional HTML document with: cover page, table of contents, figures,
tables, captions. Charts use the procurement design palette. References list at end.

### Phase 5 — Quality gate
Verify: section presence, citation count ≥ 15, spelling, clear placeholder marking.

## Section coverage — Category Strategy Playbook

**Research sections (✨ AI Generated):** 1.2, 1.3, 1.4, 2.1.1, 2.2.2, 2.3.5, 2.3.6,
2.3.7, 2.4.1, 2.4.2, 2.4.3, 2.4.4, 2.4.5, 2.4.6, 2.4.7, 2.5.3, 2.6.3, 2.6.4
**Client sections (require input):** 1.1, 2.1.2, 2.2.1, 2.2.3, 2.3.1, 2.3.2,
2.3.3, 2.3.4, 2.3.8, 2.5.1, 2.5.2, 2.5.4, 2.5.5, 2.6.1, 2.6.2

```
1. Overview, Definitions and Orientation
   1.1 Document Overview                                 [CLIENT]
   1.2 What is Category Management?                      [RESEARCH]
   1.3 Components of Category Strategy                   [RESEARCH]
   1.4 Category Strategy Source List                     [RESEARCH]
2. Category Strategy Outline
   2.1 Category Profile
       2.1.1 Category Definition & Taxonomy              [RESEARCH] (UNSPSC/NAICS)
       2.1.2 Category Scope                              [CLIENT]
   2.2 Stakeholder Management & Governance
       2.2.1 Stakeholder Identification                  [CLIENT] (Power×Interest)
       2.2.2 Roles and Responsibilities                  [RESEARCH] (RACI)
       2.2.3 Governance & Engagement Plan                [CLIENT]
   2.3 Internal Analysis
       2.3.1 Spend Analysis                              [CLIENT] (Pareto)
       2.3.2 Vendor Profiling                            [CLIENT]
       2.3.3 Contract Analysis                           [CLIENT]
       2.3.4 Payment Terms Analysis                      [CLIENT]
       2.3.5 Category Maturity Assessment                [RESEARCH] (5-level)
       2.3.6 SWOT Analysis                               [RESEARCH] (2×2)
       2.3.7 Category Segmentation                       [RESEARCH] (Kraljic)
       2.3.8 Segmentation Summary Table                  [CLIENT]
   2.4 External Analysis
       2.4.1 Market Overview & Trends                    [RESEARCH] (TAM/CAGR)
       2.4.2 Vendor Landscape                            [RESEARCH] (market share)
       2.4.3 Industry Cost Structure                     [RESEARCH]
       2.4.4 PESTLE Analysis                             [RESEARCH]
       2.4.5 Porter's Five Forces                        [RESEARCH]
       2.4.6 Risk Analysis                               [RESEARCH] (register)
       2.4.7 Recent News & Market Intelligence           [RESEARCH] (last 24 mo)
   2.5 Formulating Category Strategy
       2.5.1 Business Requirements                       [CLIENT]
       2.5.2 Category Goals                              [CLIENT] (SMART)
       2.5.3 Opportunity Assessment                      [RESEARCH] (benefit×effort)
       2.5.4 Identify Initiatives                        [CLIENT]
       2.5.5 Sourcing Pipeline                           [CLIENT]
   2.6 Execution Plan
       2.6.1 Execution Timeline                          [CLIENT] (Gantt)
       2.6.2 Detailed Execution Plan                     [CLIENT] (WBS)
       2.6.3 Savings Tracking Template                   [RESEARCH]
       2.6.4 Maintenance & Revision                      [RESEARCH]
```

## Framework library (mini-prompts for research sections)

| Section | Framework | Mini-prompt |
|---|---|---|
| 1.2 | CIPS/ISM definition | "Define category management in 120 words citing CIPS and ISM." |
| 1.3 | Analyse → Plan → Execute cycle | "Core components of category strategy, ≤150 words." |
| 1.4 | Annotated bibliography | "Table of 10 authoritative sources (analyst, gov, academic, trade) with URLs." |
| 2.1.1 | UNSPSC / NAICS hierarchy | "Formal definition and taxonomy levels for category (≤150 words)." |
| 2.2.2 | RACI matrix | "RACI table for five key stakeholder roles." |
| 2.3.5 | 5-Level model (Ad-hoc → World-Class) | "Five maturity levels and criteria in a table." |
| 2.3.6 | 2×2 SWOT grid | "SWOT grid with ≤6 bullets per quadrant." |
| 2.3.7 | Kraljic matrix | "Sub-segments to Kraljic quadrants + rationale (≤150 words)." |
| 2.4.1 | TAM/SAM/CAGR | "Market size, CAGR, top 3 trends with citations (≤200 words)." |
| 2.4.2 | Top-supplier table | "Leading suppliers with estimated market shares." |
| 2.4.3 | Cost driver pie data | "Cost structure percentages for pie chart." |
| 2.4.4 | PESTLE table | "PESTLE table with 2 factors per dimension." |
| 2.4.5 | Porter force rating | "Rate each force High/Med/Low + one-sentence justification." |
| 2.4.6 | Risk register | "5-row risk register with probability + mitigation." |
| 2.4.7 | News digest | "3 recent (<24 months) news items + implications." |
| 2.5.3 | Benefit×effort | "5 opportunities ranked by benefit + effort." |
| 2.6.3 | Savings tracker | "Savings tracker template with example line." |
| 2.6.4 | Governance cadence | "Maintenance activities across four quarters." |

## Decision logic

For each subsection {S}:
1. Is S a RESEARCH section?
   - Yes → Retrieve and verify public data → Write narrative + visuals → Insert
     citations → Add `(✨ AI Generated)` to title.
   - No → Insert `<<CLIENT INPUT REQUIRED>>` placeholder.
2. Visual needed?
   - Yes → Generate chart/table; if numeric source unavailable, flag `<<DATA NEEDED>>`.
3. Content length rules:
   - Tier-1 sections (1.1, 2.3, 2.4, 2.5) = 250-350 words narrative + visuals.
   - Tier-2 subsections = 120-200 words.

Edge cases:
- Market data > 24 months old → append reliability note.
- Conflicting sources → state the range and cite both.

## Output

A single self-contained HTML file:
`~/.claude/plugins/config/vallor/category-strategies/Category_Strategy_<Category>_v1.0.html`

Document elements:
- Cover page with title and corporate styling (Vallor + customer logos in
  bottom-right; pull from `~/.claude/plugins/config/vallor/logos/` per the
  `feedback_real_logos_in_brand_output` reference — use actual PNG files, not
  text spans).
- Comprehensive table of contents.
- All sections properly formatted with headings, subheadings, charts, tables,
  visual elements with captions.
- Consistent formatting per the design system.
- In-line citations and a references section (≥15 citations).
- Clear placeholders for client sections.

Output rules:
- Single HTML file, embedded CSS/JS, Chart.js via CDN.
- Sanitize counterparty / supplier names that originated outside this session.
- Print-friendly via `@media print`.
- Executive summary first, analytics next, visuals fully captioned.
- Active voice, precise verbs, no filler.
- Standard financial notation (USD 2.3 M, CAGR 4.2 %).
- Cite sources in-line (Author, Year) with full reference list at end.

## Input structure

Required:
- `<CATEGORY>` — e.g., "Global IT Hardware Maintenance"
- `<COMPANY_CONTEXT>` — size, industry, geographical footprint, fiscal year,
  confidential nuances

Optional:
- `<TASK_PARAMETERS>` — deadline, theme overrides, citation style

## Decision tree after delivery

1. **Add to the procurement profile** — append the category to the
   `## Categories under management` table in CLAUDE.md.
2. **Run the sourcing strategy** — `/vallor:sourcing-strategy` to
   translate this playbook into an event-by-event sourcing plan.
3. **Run the spend analysis** — `/vallor:spend-analysis` if you
   haven't already; the strategy's "Internal Analysis" section will be richer.
4. **Build the savings dashboard** — track the initiatives identified in 2.5.4.
5. **Stop here** — strategy is saved.

## Examples

```
/vallor:category-strategy "Global IT Hardware Maintenance"
```

```
/vallor:category-strategy "Corrugated Packaging — North America" --depth deep
```

```
/vallor:category-strategy "Facilities Management" --region "Latin America"
```
