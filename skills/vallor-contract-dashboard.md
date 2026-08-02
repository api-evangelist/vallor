---
name: contract-dashboard
description: >
  Generates a multi-tab interactive HTML dashboard across a portfolio of contracts —
  severity counts, renewal timeline, deviation heat map, spend exposure, cancel-by
  calendar, supplier concentration. Print-friendly tabs. Self-contained. Use when
  the user says "contract dashboard", "portfolio view", "dashboard for all
  contracts", "show me everything renewing", "build the contracts dashboard", or
  wants a CFO/CPO-grade view of the entire contract base, not a single contract.
argument-hint: "[--source clm|local-folder|register] [--filter renewals|deviations|all]"
---

# /vallor:contract-dashboard

## Purpose

The renewal-tracker tells you what's renewing. The review skills produce
per-contract memos. This skill produces the portfolio view — every active
contract, every deviation, every cancel-by deadline, every dollar of exposure
— in a multi-tab interactive HTML dashboard a CFO, CPO, or GC can walk
through in 5 minutes.

## Load context

1. Read `~/.claude/plugins/config/vallor/CLAUDE.md`.
2. **Apply the /design skill** — monotone palette. Charts use `#1F3A5F` bars
   on `#E5E5E5` gridlines. No pie charts (proportions are tables).
3. Source data:
   - **`--source clm`** — pull from the Ironclad / Agiloft MCP. Requires the
     CLM integration to be connected.
   - **`--source local-folder`** — read the plugin's local register at
     `~/.claude/plugins/config/vallor/renewal-register.yaml`
     and the matter folder for review memos.
   - **`--source register`** (default) — uses whatever's already in the local
     register (the default for in-house users without a CLM connector).
4. Build the dataset before rendering. The dashboard is a function of the data;
   the data isn't a function of the dashboard.

## Ask the user for context (optional but encouraged)

Use the `AskUserQuestion` tool to scope the dashboard. Portfolio dashboards are easy to overbuild — questions up front prevent rendering five tabs the user did not want.

**Up front (before pulling the dataset):**
Ask 1-3 short questions when not obvious from the inputs:
- Audience: CFO / CPO / GC / cross-functional ops
- Portfolio scope: all active contracts / single business unit / single category / single vendor
- Focus: renewals timeline / risk concentration / spend exposure / supplier concentration / deviations heat map
- Time window: next 90 days / next 12 months / lifetime portfolio
- Refresh cadence the dashboard needs to support: one-off snapshot / weekly / monthly

**Mid-flight (after pulling and shaping the dataset):**
After the dataset is built, you will see real gaps. Stop and ask before guessing on:
- Records with missing fields — drop, flag, or backfill from the source
- Severity thresholds for the heat map (the playbook may not have one for this portfolio)
- Whether to roll up subsidiaries / business units or keep them separate

**Rules:**
- Every question must offer a reasonable default the user can pick to skip ahead.
- Ask follow-ups when answers reveal more depth or a non-default choice.
- Cap a single round at ~3 questions.
- Skip when the user has signaled a fast / default run.

## Dashboard structure — five tabs

### Tab 1 — Overview

Top section: four big-number cards.

```
ACTIVE CONTRACTS        ANNUAL EXPOSURE      CANCEL-BY < 30 DAYS    BLOCKING ISSUES
        217                $48.2M                     3                    2
```

Each card is a 1pt navy rule on the left, near-black number, `#5B5B5B` label.

Below: three blocks.
- **Renewal timeline (90 days):** Sparkline-style timeline with markers for
  each upcoming cancel-by date. Click a marker to drill into the contract row
  in Tab 2.
- **Deviation severity distribution:** Horizontal bar chart with four bars
  (Blocking / High / Medium / Low). `#7B1F1F` for the top two, `#5A5A2A` for
  medium, `#3A3A3A` for low. No pie chart.
- **Spend concentration (Pareto):** Top-10 counterparties by ACV, bar chart
  with cumulative line at 80%.

### Tab 2 — Contracts

The portfolio table. Sortable columns, filterable header row.

| Counterparty | Type | Side | ACV | Term | Cancel-by | Renewal | Last review | Severity | Owner |
|---|---|---|---|---|---|---|---|---|---|
| ... | ... | ... | ... | ... | ... | ... | ... | ● ... | ... |

Each row clickable — drills to that contract's `contract-report-html` if one
exists, or surfaces "no review on file — run `/vallor:review`."

### Tab 3 — Renewals

Calendar view — next 90 days, grouped by week. Each contract appears as a row
under its cancel-by week, with the cancel-by date in bold red if 0-13 days, in
amber if 14-44 days, in near-black if 45-89 days.

Also: a "Renewals at risk" callout — contracts where the cancel-by has passed
without action, or where notice has been given but renewal is still in
play.

### Tab 4 — Deviations

The deviation heat map. Rows are clauses (Liability cap, Indemnity, Data
protection, Term, Governing law, etc.); columns are counterparties. Cells
are color-coded by severity. Click a cell to see the deviation detail and
the proposed redline.

Filter bar: show only deviations on the Never-accept list / Currently in
negotiation / Settled.

### Tab 5 — Risk register

Cross-contract risk register. Each row is a risk that surfaced in a review:
cybersecurity gaps, unlimited-liability exposure, single-source supplier
dependence, jurisdiction concentration. For each: which contracts contribute,
total exposure, recommended action.

## Interactivity

- Tabs switched by vanilla JS (URL hash sync).
- Column sorting, header-row filters, severity badge toggles.
- Cell clicks drill to detail; back button works as expected.
- All charts are Chart.js (loaded via CDN — the only CDN dependency in this
  plugin's HTML outputs; the per-contract reports don't need it).
- Keyboard: `1`-`5` switch tabs.

## Tech constraints

- Single HTML file (the data is inlined as a JSON object in a `<script>`
  block).
- All counterparty names, contract titles, and any other untrusted content
  rendered with `textContent`. Vendor-provided URLs scheme-checked. No
  `innerHTML` ever.
- Print-friendly: `@media print` shows all tabs sequentially, drops the
  chart canvases (replaced with the underlying table for accessibility),
  hides interactive chrome.
- Works on screens ≥768px. Below that, the dashboard collapses to a single
  column with a "this dashboard is designed for tablet and up" notice.

## Output

Save to:
`~/.claude/plugins/config/vallor/dashboards/<YYYY-MM-DD>-portfolio.html`

## Decision tree

1. **Slice by team / business unit** — re-run with `--filter` to scope.
2. **Schedule the renewal-watcher agent** — keeps the dashboard fresh; surfaces
   high-priority items to Slack.
3. **Export the underlying CSV** — for finance / FP&A who want the data flat.
4. **Stop here** — dashboard is saved.

## Refresh cadence

The dashboard is a snapshot. The user-facing footer states:

```
Dashboard built [YYYY-MM-DD HH:MM] · Data as of [last-review date] · Refresh: re-run /vallor:contract-dashboard
```

If the renewal-watcher agent is scheduled, it can regenerate the dashboard
weekly and post the new URL to the configured Slack channel.

## Examples

```
/vallor:contract-dashboard
```

```
/vallor:contract-dashboard --source clm
```

```
/vallor:contract-dashboard --filter renewals
```
