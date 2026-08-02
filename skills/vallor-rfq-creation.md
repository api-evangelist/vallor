---
name: rfq-creation
description: >
  Generates a complete multi-sheet Excel RFQ workbook tailored to a user-specified
  good, service, or hybrid engagement. Plus an HTML cover report summarizing the
  RFP for stakeholder distribution. Two-phase intake (initial + informed after
  silent research) so options are category-specific. 7 sheets standard, 8 for
  hybrid (Supplier Info, Business Process, Technical/Service, Operational, Pricing,
  Rate Card or Unit Cost, Attachments, optional Unit Cost). Use when the user says
  "create an RFQ", "build an RFP", "generate the RFQ", "RFQ workbook", "sourcing
  event setup", "rfq for X", or "I need an RFP template for X".
argument-hint: "<good-or-service> [--company <name>] [--pricing-model subscription|milestone|time-and-materials] [--include-formulas true|false]"
---

# /vallor:rfq-creation

## Purpose

A good RFP gets meaningful, comparable answers. A bad RFP gets PDFs back with
asterisks. This skill builds the workbook your suppliers will actually fill in,
with the columns and validations procurement, finance, ops, IT, legal, and the
end-user team can all consume.

You are a senior procurement analyst with 15+ years of experience designing,
evaluating, and managing Requests for Proposal (RFPs) across direct and indirect
spend categories.

## Concept disambiguation

- **RFP** vs **RFQ**: RFP = qualitative + quantitative evaluation, detailed
  requirements, multi-criteria scoring. RFQ = price-only bid for well-defined
  items. This skill produces both shapes — the distinction lives in the user's
  pricing model and emphasis.
- **Good** vs **Service** vs **Hybrid**: Good = tangible item. Service =
  intangible activity. Hybrid = both. The Rate Card / Unit Cost sheet adapts.

The workbook is cross-functional — finance, ops, IT, legal, and end-user
teams will all consume its contents. Every sheet must be present.

## Critical instructions

- You receive: `[GOOD_OR_SERVICE]`, `[COMPANY_NAME]`, plus optional tags.
- **Deliverable**: Two files only — no JSON, Markdown, or explanatory text:
  1. An **Excel workbook** built with Python (openpyxl) — write and run code.
  2. A **final report in HTML** summarizing the RFP: cover page, section
     summaries, requirement counts, pricing structure overview, key supplier
     instructions.
- Create exactly **seven sheets** (plus an eighth Unit Cost when Hybrid):
  1. Supplier Information
  2. Business Process Requirements
  3. Technical & Service Requirements (rename to "Technical Requirements" when
     good-only)
  4. Operational Requirements
  5. Pricing Information
  6. Rate Card (or "Unit Cost" for goods; "Labor Rate Card" for hybrid)
  7. Attachments
  8. Unit Cost (hybrid only)
- Insert placeholder brackets `››…‹‹` for every field the user must later fill
  (e.g., `››Parent Company‹‹`). Use the typographic `››‹‹` characters only —
  never ASCII `<` / `>` or HTML-escaped `&lt;` / `&gt;`.
- Keep all formulas, data-validation lists, conditional formatting, and scoring
  macros unless `[INCLUDE_FORMULAS]` = false.
- Style: bold 14pt headers, gray fill `#D9D9D9`, freeze top row, autofit
  columns to 25-char minimum, wrap text in long-form cells.
- Use code execution to generate both files.
- Apply the **/vallor:design** skill for the HTML cover report
  (monotone, no decoration).

## Load context

1. Read `~/.claude/plugins/config/vallor/CLAUDE.md`:
   - `## Categories under management` (for context if category matches)
   - `## Sourcing approach` (default sourcing event, bid floor, RFQ format,
     scoring weights, TCO drivers)
   - `## Vendor / supplier classification`
   - `## Negotiation posture`
2. If the procurement profile is empty, run with defaults and tag the output
   `[PROVISIONAL]`.

## Two-phase intake

**STEP 0 — Acknowledge.** Send one short message: "On it. I'll ask a few quick
questions to tailor the RFQ, then kick off the build." Then immediately call
phase 1.

### Phase 1 — Initial intake (use AskUserQuestion)

Ask these in plain language, no variables or brackets:

1. What's the name of your company?
2. What are you sourcing with this RFP?
3. Which region is this for?
4. How long do you want the contract to run?
5. Roughly how much do you expect to spend per year?

### Research phase (internal, no user output)

Briefly research the company and the good/service so Phase 2 options are concrete
and category-specific. Use web search if available. No output to the user during
this phase.

### Phase 2 — Informed intake (use AskUserQuestion)

Ask these with category-specific, pre-populated options drawn from research:

1. How do you want suppliers to price this?
2. What should suppliers be judged on most heavily?
3. Anything specific you want suppliers to answer beyond the standard questions?
4. Who's your current supplier (if any)?
5. Should the Excel include scoring formulas and validations?
6. Which documents should suppliers attach?

**Character safety:** option text must never contain `<`, `>`, `&lt;`, or `&gt;`.
Use `[TOKEN]` or `{TOKEN}` for variable references. The Ask User Question tool's
JSON parser can truncate or drop option text between angle brackets.

Once both phases are complete, build immediately. Do not ask further questions.

## Ask the user for context (follow-ups, optional)

The two-phase intake above covers the upfront questions. Beyond that, follow-ups remain optional but encouraged. If something material is still ambiguous after the build, surface a short, targeted `AskUserQuestion` round:

- Specs the research didn't resolve cleanly (e.g., regulatory regime in scope, supplier short-list visibility)
- Scoring weights the user picked that conflict with the must-have list (e.g., cost weighted heaviest but a hard cybersecurity requirement)
- Whether to include or skip optional sheets the category may not need (e.g., unit-cost sheet when the engagement is service-only)

**Rules:**
- Don't re-ask anything already answered in Phase 1 or Phase 2.
- Every follow-up question must offer a reasonable default.
- Cap a single round at ~3 questions.
- Skip when the user has signaled a fast / default run.

## Methodology

### 1. Header customization
- Replace software-specific terms with neutral equivalents.
- Inject `››GOOD_OR_SERVICE‹‹` into relevant titles and descriptions.

### 2. Placeholder injection
- Use `››YYYY-YYYY‹‹` for year ranges; leave Yes/No cells blank for supplier
  input.

### 3. Requirement question generation (Sheets 2-4)

**Supplier-response scale (embed in header):**
1 = Out-of-the-box · 2 = Supported via configuration · 3 = Supported via
customization · 4 = Work-around available · 5 = Planned / future release · 6 =
Not supported

**Hierarchy:** Category (N) → Sub-category (N.n) → Leaf requirement (N.n.n). IDs
restart at 1 on each sheet.

**Target volume:**
- Business Process Requirements: ≥ 3 categories, each ≥ 2 sub-categories, each
  with 3-5 leaf requirements (≈30-45 questions).
- Technical & Service Requirements: ≥ 4 categories (Integration, Security,
  Usability, Analytics), each ≥ 3 sub-categories with 3-5 leaf requirements
  (≈45-60 questions).
- Operational Requirements: ≥ 3 categories (Reliability, Performance, Security),
  each 4-8 leaf requirements (≈25-30 questions).

**Quality criteria per leaf:** Clarity (one unambiguous need), Testability
(answerable on 1-6 scale), Relevance (directly affects evaluation for
`››GOOD_OR_SERVICE‹‹`), Coverage (collectively spans functionality, risk,
scalability, compliance, UX, future-proofing).

**Adaptation:**
- Goods: emphasize specifications, material compliance, lifecycle, warranty.
- Services: emphasize SLA, staffing, methodology, knowledge transfer.
- Hybrid: include both.

### 4. Sheet adaptation
- Technical & Service Requirements → "Technical Requirements" when good-only.
- Rate Card transformations:
  - Good-only → rename "Unit Cost", convert "Role" → "Item Category".
  - Hybrid → keep "Labor Rate Card" and add a separate "Unit Cost" sheet.

### 5. HTML report
- Clean, professional styling (use a sans-serif font, white background, branded
  header per design system).
- Sections: Cover Page, RFP Overview, Requirement Summary (counts by sheet),
  Pricing Structure, Key Instructions for Suppliers.
- Placeholders appear as styled highlighted spans.
- Standalone .html file.

### 6. Validation & formatting
- Workbook opens without errors.
- Formulas exist (unless stripped) and column widths prevent truncation.

## Decision logic

### Classification
- Service → keep Rate Card.
- Good → use Unit Cost sheet instead of Rate Card.
- Hybrid → keep "Labor Rate Card" AND add Unit Cost sheet.

### Custom criteria
When `[OPTIONAL_CUSTOM_CRITERIA]` provided, append under correct section with new
hierarchical IDs.

### Pricing model adjustments
- "subscription" → add "Annual Fee" column to Pricing Information.
- "milestone" → add "Milestone Name" and "Milestone Amount" columns.
- "time-and-materials" → default structure; align with Rate Card roles.

## Output

**Files:**
1. `RFP_<GOOD_OR_SERVICE>_<COMPANY_NAME>.xlsx`
2. `RFP_<GOOD_OR_SERVICE>_<COMPANY_NAME>_Report.html`

Save both to:
`~/.claude/plugins/config/vallor/rfqs/<YYYY-MM-DD>-<good-or-service>/`

### Sheet & column definitions

**1. Supplier Information:**
Company Legal Name · Parent Company · Headquarters Country · Years in Business ·
Annual Revenue `››YYYY‹‹` · Number of Employees · Public/Private · ISO
Certification · Key Customers · Reference Contacts · Financial Ratios · Comments

**2. Business Process Requirements:**
Criteria # · Criteria Description · Supplier Response *(1-6 scale)* · Additional
Clarification · Roadmap Timeline · Comments

**3. Technical & Service / Technical Requirements:**
ID · Requirement Category · Description · Supplier Response *(1-6 scale)* ·
Additional Clarification · Roadmap Timeline · Comments

**4. Operational Requirements:**
ID · Criteria · Description · Can Support? (Yes/No) · Comments

**5. Pricing Information:**
Description · Quantity · Unit · Year 1 Costs · Year 2 Costs · Year 3 Costs ·
Year 4 Costs · Year 5 Costs · Comments

**6. Rate Card (service) / Labor Rate Card (hybrid):**
Role or Title · Listed Rate (Currency/Unit) · `››COMPANY_NAME‹‹` Rate · Discount Offered

**7. Unit Cost (good or hybrid):**
Item Category · SKU/Model · Unit of Measure · Unit Cost (Currency) · Volume
Tier · Discount Offered

**8. Attachments:**
ID · Attachment Name · Description

### File construction rules
- Supplier-response scale text in the appropriate column header for Sheets 2-3.
- Preserve formulas, validations, conditional formatting unless
  `[INCLUDE_FORMULAS]` = false.
- Header styling: bold 14pt, gray fill `#D9D9D9`.
- Freeze top row; autofit to 25-char min width; wrap text in long cells.

## Writing rules

- Concise, neutral business language.
- Avoid jargon, hype, commentary.
- Numbering follows N, N.n, N.n.n pattern; matches column IDs.
- All user-facing placeholders wrapped in `››‹‹`. Never ASCII `<` / `>` or
  `&lt;` / `&gt;`.

## Input structure

Required:
- `[GOOD_OR_SERVICE]` — noun phrase (e.g., "cloud backup services")
- `[COMPANY_NAME]` — legal entity name

Optional:
- `[PREFERRED_PRICING_MODEL]` — "subscription", "milestone",
  "time-and-materials"
- `[OPTIONAL_CUSTOM_CRITERIA]` — JSON array of requirement objects
- `[INCLUDE_FORMULAS]` — Boolean (default = true)
- `[ADDITIONAL_CONTEXT]` — free-text notes

## Decision tree after delivery

1. **Send to suppliers** — distribution list, response window, Q&A protocol.
2. **Run the RFQ analysis** — after responses come back, point
   `/vallor:rfq-analysis` at the responses folder.
3. **Build the sourcing-strategy** — if this RFQ is part of a larger event,
   `/vallor:sourcing-strategy` wraps the full timeline.
4. **Stop here** — files are saved.

## Reminders

- Deliver **Excel + HTML report only** — no JSON or Markdown.
- Create exactly the sheets and columns defined; do not add, drop, merge, or
  reorder.
- Generate robust question sets per Methodology §3.
- Keep numbering hierarchy intact; blank rows stay to preserve structure.
- Use typographic `››‹‹` brackets only.
- Never emit ASCII `<` / `>` or HTML-escaped `&lt;` / `&gt;` in any
  AskUserQuestion call.
- Validate workbook opens error-free before returning.
- Create each sheet one at a time.
- Be thorough; double-check completeness before delivering.

## Examples

```
/vallor:rfq-creation "corrugated packaging"
```

```
/vallor:rfq-creation "IT help-desk outsourcing" --pricing-model subscription
```

```
/vallor:rfq-creation "industrial laser cutters & maintenance" --company "Photon Tech LLC"
```
