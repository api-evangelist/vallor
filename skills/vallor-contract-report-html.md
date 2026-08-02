---
name: contract-report-html
description: >
  Generates a polished, printable, standalone HTML report from a contract review,
  redline, or amendment-history output. Self-contained — embedded CSS, no external
  dependencies, board-ready monotone design. Print-friendly. Use when the user says
  "report", "html report", "create the report", "polished report", "print this for
  the partner", "single-page contract report", "save as html", or wants a clean
  reading copy of a review for distribution.
argument-hint: "[source-md] [--audience legal|business|board] [--output path]"
---

# /vallor:contract-report-html

## Purpose

A markdown review memo is great for the working file; a board reader wants HTML
that reads like a firm-published client alert. Same content, dramatically
different polish. This skill takes the source markdown and renders it.

## Load context

1. Read `~/.claude/plugins/config/vallor/CLAUDE.md`.
2. **Apply the /design skill** — the report follows the monotone palette, Source
   Serif Pro body, 720px content width, navy `#1F3A5F` masthead rule.
3. Read the source — typically a markdown review memo, redline, or amendment
   history. Validate that the source has a reviewer note, a findings section,
   and a recommendation; if not, surface the missing pieces and ask.

## Ask the user for context (optional but encouraged)

Use the `AskUserQuestion` tool to calibrate the polish and emphasis for the destination audience.

**Up front (before rendering):**
Ask 1-3 short questions when not obvious from inputs:
- Audience (if not via `--audience`): legal / business stakeholder / board / external counsel
- Sections to lead with (recommendation first vs findings first vs reviewer note first)
- Length: full report / executive-first (long-form behind a fold) / two-page
- Whether to include the deviation table inline or as an appendix
- Branding: include client logo, neutral, or work-product-only

**Mid-flight (after structuring the report):**
After laying out sections, surface concrete asks before guessing on:
- Findings that read fine in markdown but lose meaning out of context (e.g., paragraph-level cross-references) — rewrite or footnote
- Whether to include charts/visualizations or keep prose-only
- Whether to hyperlink to source clause numbers (if available) or stay flat

**Rules:**
- Every question must offer a reasonable default the user can pick to skip ahead.
- Ask follow-ups when answers reveal more depth or a non-default choice.
- Cap a single round at ~3 questions.
- Skip when the user has signaled a fast / default run.

## Output structure

A single HTML file. Sections in order:

### 1. Masthead

```
[Client Name] · Commercial Contracts                  [PRIVILEGED & CONFIDENTIAL]
─────────────────────────────────────────────────────────────────────────────
[Document title — e.g., "Acme Corp Vendor MSA — Review Memorandum"]
Prepared [date] · [reviewer] · [matter or deal ID]
```

### 2. Reviewer note

Per the practice profile's `## Outputs` format. Single block, above everything
else. Collapses to one line if all green.

### 3. Recommendation (above the fold)

Pulled from the source. Bordered with a 4px-left navy rule:

```
RECOMMENDATION
[Sign / Sign with conditions / Push back / Walk]

[One paragraph rationale.]
```

### 4. Findings

Pulled from the source's findings table. Rendered as an HTML table — severity
dot characters in the left column, sortable columns (vanilla JS, no
frameworks), zebra-striped if more than 10 rows.

### 5. Section-by-section detail

The body of the review. Each `## § X` block becomes a styled section with a
severity marker, the original language (italicized, indented), the proposed
language (bordered), and the rationale.

### 6. Appendix — full deviation log

Collapsed by default. Click to expand. Useful for the reviewer who wants to
trace a specific clause.

### 7. Footer

```
PRIVILEGED & CONFIDENTIAL — ATTORNEY WORK PRODUCT
Page X of Y · Prepared [date] · [reviewer]
```

## Tech constraints (per /design skill)

- Single HTML file. All CSS in a `<style>` block at the top. No external CSS.
- No JS frameworks. Sorting/filtering uses vanilla JS, ~50 lines max.
- **Sanitize every value** that originated outside this session (counterparty
  text, vendor names, OSS package names). Use `textContent`, never `innerHTML`.
  Scheme-check URLs (`http:` / `https:` / `mailto:` only).
- Print-friendly: `@media print` rules collapse the appendix, reset colors to
  near-black, drop the JS chrome.
- Works offline. No CDN dependencies for the basic report (the dashboard skill
  loads Chart.js via CDN — this one doesn't need it).

## CSS scaffold

Use the variables defined in `skills/design/SKILL.md`. The full block to drop
into `<style>` is in that file's "Quick reference" section.

## Output

Save to:
`~/.claude/plugins/config/vallor/reports/<YYYY-MM-DD>-<slug>.html`

Open the file in the user's browser if `--open` is passed (use `open` on macOS,
`xdg-open` on Linux, `start` on Windows). Otherwise return the path.

## Decision tree after delivery

1. **Print to PDF** — `cmd-P` in browser; the print stylesheet handles the rest.
2. **Build the dashboard view** — `/vallor:contract-dashboard` for
   the portfolio-level view across many contracts.
3. **Build the approval deck** — `/vallor:contract-approval-deck` for
   the PPTX version.
4. **Stop here** — report is saved.

## Audience switches

- `--audience legal` (default) — keeps work-product header, `[review]` markers,
  full citation tags.
- `--audience business` — strips work-product header, replaces `[review]` with
  `[needs decision]`, hides citation tags inline (moves them to a footnote
  block at the end).
- `--audience board` — strips everything granular; keeps recommendation,
  three-number summary, top 5 findings, decision options. Same data, much less
  detail.

## Examples

```
/vallor:contract-report-html review-memos/acme-msa.md
```

```
/vallor:contract-report-html redlines/acme-msa.md --audience board
```
