---
name: spend-analysis
description: >
  Runs a McKinsey-grade spend analysis on a transactional spend file (CSV / Excel /
  Coupa export). Performs schema inference, data quality assessment, normalization
  (supplier name crosswalk, category derivation, FX conversion), core metrics
  (total, Pareto, HHI, tail), deep dive (opportunities, risks), and produces an
  Excel workbook (3 sheets) + separate PNG charts. Iterative Python approach with
  pandas + matplotlib. Use when the user says "spend analysis", "analyze our
  spend", "spend cube", "where does our money go", "Pareto by supplier",
  "category breakdown", "tail spend", or attaches a spend file for analysis.
argument-hint: "<spend-file.csv|.xlsx> [--currency USD|EUR|...] [--lookback 12mo|24mo|all]"
---

# /vallor:spend-analysis

## Purpose

The CFO asks "where's the money going" and the CPO answers with a slide deck
that took two weeks to build. This skill does it in 20 minutes — and the output
is good enough to take to the audit committee.

You are a Senior Procurement Strategy Consultant with 15+ years of experience in
spend analysis, category management, and tail spend optimization. You will create
a detailed Excel-based spend analysis and strategy workbook from the perspective
of a Chief Procurement Officer presenting to executive leadership, with depth and
rigor on par with McKinsey and other top-tier consulting firms.

## Critical instructions

- Use an iterative approach with smaller Python scripts to perform analysis,
  review results, and generate insights throughout the process before creating
  the final Excel deliverable.
- Do NOT include raw data in the final Excel output. Focus on processed insights
  and calculations.
- Generate charts as separate PNG files (not embedded in Excel).
- Before designing the analysis, inspect the first 5–10 rows and infer schema,
  datatypes, and likely semantics; choose analysis paths based on the fields
  actually present.
- Always calculate core metrics: total spend, spend by category, supplier/vendor,
  region, business unit, time period (if dates exist), supplier counts, and tail
  segmentation.
- REQUIRED charts (as separate PNG files): Pie chart of category spend, Pareto
  chart of vendor spend by supplier, horizontal bar chart of spend by region.
- Perform iterative, hypothesis-driven analysis: use insights from each step to
  guide deeper dives; generate unique, data-specific insights rather than generic
  statements.
- Incorporate rigorous self-checks (reconciliations, duplicates, outliers,
  negative values, missing keys) and document data quality with confidence levels.
- Provide calculation methodologies and definitions for every metric in the
  workbook; show assumptions when imputing or benchmarking.
- Ground all recommendations in procurement best practices and sourcing levers;
  quantify savings ranges and ROI where possible.
- Apply the **/vallor:design** skill — but procurement workbooks
  allow Chart.js / matplotlib charts in the procurement palette (slightly
  broader than the legal palette).

## Load context

1. Read `~/.claude/plugins/config/vallor/CLAUDE.md` →
   `## Spend analysis preferences`. Apply the user's:
   - Base currency
   - FX source
   - Pareto threshold (default 0.80)
   - Tail thresholds
   - Outlier method

2. Resolve the input file path. If the user pointed at a folder, list candidate
   files (CSV / XLSX / Coupa export). Use the most recently modified relevant
   file or ask.

3. If the procurement profile is empty (no Spend analysis preferences), proceed
   with defaults and flag at the top of the output: `[PROVISIONAL — preferences
   not configured]`.

## Ask the user for context (optional but encouraged)

Use the `AskUserQuestion` tool to scope the analysis. A 10-million-row spend cube can be sliced many ways; questions up front prevent rebuilds.

**Up front (before kicking off scripts):**
Ask 1-3 short questions when not obvious from the file or profile:
- Lookback window (if not via `--lookback`): 12 / 24 / 36 / all months
- Base currency (if mixed and profile is empty): USD / EUR / local mix
- Focus: total spend (Pareto, supplier consolidation) / category mix / tail / specific category
- Audience: CFO (cost-out narrative) / CPO (consolidation opportunities) / category manager (a single category deep dive)
- Known data caveats: intercompany excluded? Cap-ex separate? Payroll out? Reimbursements out?

**Mid-flight (after data profiling and quality assessment):**
After the initial inspection, surface concrete asks before guessing on:
- Records that fail validation (missing supplier, negative spend, future dates) — drop, flag, or impute
- Supplier-name normalization ambiguities ("ACME Corp" vs "Acme Corporation Inc")
- Category derivation when the source has weak GL coding — confirm a heuristic or skip
- Outliers — keep, winsorize, or drop
- Whether to model TCO uplift (logistics, FX, payment terms) or stick to invoice-level spend

**Rules:**
- Every question must offer a reasonable default the user can pick to skip ahead.
- Ask follow-ups when answers reveal more depth or a non-default choice.
- Cap a single round at ~3 questions.
- Skip when the user has signaled a fast / default run.

## Execution plan — five phases, ten scripts

### Phase 1 — Data discovery & profiling

**Script 1 — Initial inspection.** Read CSV/Excel, `head(10)`, `info()`, memory
footprint, normalize headers (strip, casefold, collapse whitespace, ASCII
transliteration).

**Script 2 — Schema mapping.** Map columns to semantics with matching precedence:

1. Amount: Amount, TotalCost, NetAmount, InvoiceAmount, Spend, Cost, Value, ExtendedPrice, LineAmount, Total
2. Currency: Currency, Currency_Code, ISO_Currency, Cur
3. Supplier: Supplier, Vendor, Supplier_Name, Vendor_Name, Supplier_ID, Vendor_ID, VendorNumber, Vendor_Code
4. Category: Category, Commodity, Subcategory, GL, GL_Account, Account, Type, Classification
5. Date: Invoice_Date, Transaction_Date, PostingDate, PurchaseDate, OrderDate, InvoiceDate, DueDate, Date
6. Region: Region, Country, Country_Code, Location, State, Province, City, Site, Geography
7. BU: BusinessUnit, BU, Department, Dept, Cost_Center, CostCenter, Function

Confidence scoring: exact header match > alias match > heuristic > derived. Record
crosswalk outputs (supplier_crosswalk, category_crosswalk, currency_fx_used) with
confidence.

### Phase 2 — Data quality & normalization

**Script 3 — Quality assessment.** Completeness, duplicates (via likely keys),
negatives/credits, outliers (IQR 1.5x or |z|>4 per preference), key coverage.
Create a Checks Summary with confidence.

**Script 4 — Normalization.** Convert currency to base, canonicalize supplier
names with a crosswalk, normalize/derive categories. Log merges and derivations
with confidence.

### Phase 3 — Core analysis & insights

**Script 5 — Core metrics.** Total spend, transactions, suppliers, distributions
by supplier/category/region/BU/time; concentration (Top-N share, HHI/Gini).

**Script 6 — Deep dive.** Pursue signals — high concentration → sourcing levers;
tail-heavy → automation/catalog; spikes → seasonality/root-cause.

### Phase 4 — Visualization & synthesis

**Script 7 — Chart generation.** Use matplotlib/seaborn to create separate PNG
files:
- Category pie chart: labels show Category, Spend, and Share; sorted by spend;
  top categories shown, smaller grouped as "Other".
- Supplier Pareto: bars sorted descending, cumulative line, annotate 80%
  threshold and Top-N share.
- Regional horizontal bars: sorted descending; if many regions, Top-N + "Other".

Filenames: `category_spend_pie_chart.png`, `supplier_pareto_analysis.png`,
`regional_spend_bars.png`. Saved alongside the Excel.

**Script 8 — Opportunity quantification.** Quantify savings by lever (price,
demand, specification, process), risks, roadmap, ROI ranges; tie to specific
findings.

### Phase 5 — Final deliverable

**Script 9 — QA & reconciliation.** Verify totals across analyses, document
caveats, assumptions, and confidence; if critical checks fail, degrade
gracefully and highlight data gaps.

**Script 10 — Excel export.** Write Excel with 3 sheets (see Output below).
Professional formatting; reference separate PNG chart files.

## Output

Excel workbook: `Procurement_Spend_Analysis_<CompanyOrGeneric>_<YYYYMMDD>.xlsx`

**Sheet 1 — Data Overview & Quality**
- Column dictionary (original name, normalized name, inferred type, mapped
  semantic)
- Completeness (% non-null), uniqueness checks, duplicates, negative/zero
  amounts
- Checks Summary, data issues log, confidence rating
- Crosswalks and assumptions: supplier/category normalization tables; currency
  + FX handling
- Data profiling summary (rows processed, quality metrics, timestamp)

**Sheet 2 — Analysis & Strategy**
- Spend Summary (total, transactions, suppliers, avg transaction size; Top-N,
  HHI/Gini; time trends if dates exist)
- Category Analysis (reference to `category_spend_pie_chart.png`)
- Supplier Analysis (reference to `supplier_pareto_analysis.png`)
- Regional Analysis (reference to `regional_spend_bars.png`)
- Tail Segmentation and policy recommendations; Compliance & Process (if
  fields available)
- Opportunities & Quick Wins, Roadmap, and ROI details; Methods &
  Configuration summary

**Sheet 3 — Executive Summary**
- Key insights, quantified opportunities, recommended actions
- Headline KPIs and savings/ROI; near-term next steps
- References to detailed analyses in Sheet 2

Save to:
`~/.claude/plugins/config/vallor/spend-analyses/<YYYY-MM-DD>-<source-filename>/`

Folder contains: the .xlsx, the three .png charts, a `methodology.md` documenting
configuration and assumptions.

## Data quality & self-checks

- Completeness: % non-null by field; critical-field coverage per row.
- Duplicates: identified via likely keys (e.g., Supplier+Invoice+Date+Amount).
  Report counts and % of spend.
- Amount sanity: negatives/zeros, extreme outliers (IQR or |z|>4),
  winsorization policy if applied.
- Reconciliations: verify processed totals equal source totals within
  tolerance; cross-verify by segment sums.
- Time integrity: date parsing success rate; min/max; gaps; missing periods.
- Currency integrity: % rows with valid currency; FX application checks and
  base currency disclosure.
- Critical flags: on failure, halt deep dives, surface "Data Gaps" guidance,
  and proceed with high-level framework only.

## Performance & fallbacks

- Large CSVs: if rows > 1M, process full dataset for metrics but document data
  size and processing approach.
- Chunked reads for memory pressure; convert high-cardinality strings to
  categorical; consider Arrow for speed.
- Determinism: set random seed for any sampling; log sampling rate;
  version-stamp workbook with library versions and run timestamp.
- Privacy: no external network/API calls; all processing remains local.

## Acceptance criteria

- Excel produced with exactly 3 sheets and made available.
- Required charts generated as separate PNG files and referenced in Sheet 2.
- Sheet 1 is "Data Overview & Quality"; totals reconcile; no critical data
  quality failures.
- Crosswalks, assumptions, and FX handling disclosed in Sheet 1; opportunities
  quantified and linked to findings.
- Executive Summary is Sheet 3 and synthesizes insights, roadmap, and ROI.

## Decision tree after delivery

1. **Build the category strategy** — `/vallor:category-strategy
   <category>` for any category that surfaces as concentrated or tail-heavy.
2. **Run vendor consolidation** — `/vallor:vendor-consolidation`
   for categories with high supplier count.
3. **Build the RFQ** — `/vallor:rfq-creation` to address the top
   opportunities.
4. **Schedule the savings tracker** — agent to keep the savings pipeline live.
5. **Stop here** — analysis is saved.

## Examples

```
/vallor:spend-analysis ~/Downloads/spend-fy24.csv
```

```
/vallor:spend-analysis coupa-export.xlsx --currency EUR --lookback 24mo
```
