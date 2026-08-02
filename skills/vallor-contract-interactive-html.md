---
name: contract-interactive-html
description: >
  Generates an interactive HTML view of a contract review with clause drill-down,
  severity filters, side-by-side comparison (their language vs proposed), and
  inline comment threads. Self-contained, vanilla JS, no frameworks. Use when the
  user says "interactive report", "interactive html", "let me click through this",
  "build the clickable version", "interactive review", or when a reviewer needs to
  explore a long redline rather than read top-to-bottom.
argument-hint: "[source-redline-md] [--mode side-by-side|stacked|drill-down]"
---

# /vallor:contract-interactive-html

## Purpose

A 30-page MSA with 80 redline entries is unreadable as a single column. This
skill turns the same content into a clickable view: sidebar of issues, click a
row to see the clause in context with their language next to the proposed
language, severity filter at the top, inline comment thread per clause.

It's the reviewer-facing counterpart to `contract-report-html` (which is the
read-only, printable distribution copy).

## Load context

1. Read `~/.claude/plugins/config/vallor/CLAUDE.md`.
2. **Apply the /design skill** — same palette, but the layout is two-column
   (sidebar 280px + main pane).
3. Read the source markdown redline. If the source is a contract PDF without a
   redline, surface: "I need a redline to build this. Run
   `/vallor:contract-redline` first."

## Ask the user for context (optional but encouraged)

Use the `AskUserQuestion` tool to calibrate the layout and emphasis. A long redline can be sliced many ways — questions up front avoid a rebuild.

**Up front (before rendering):**
Ask 1-3 short questions when not obvious from the redline:
- Primary reader: lawyer-reviewer / business owner / external counsel
- Layout preference: side-by-side / stacked / drill-down (if `--mode` not passed)
- Default severity filter on load: all / 🔴 only / 🔴+🟠
- Whether to include inline comment threads (yes/no)
- Whether to include the playbook reference column

**Mid-flight (after parsing the redline):**
Once you've indexed all issues, surface concrete asks before guessing on:
- Whether to group by clause vs by severity vs by section
- How to handle very long counterparty paragraphs (truncate with reveal vs full text)
- Whether to add a "deal-breakers only" preset filter

**Rules:**
- Every question must offer a reasonable default the user can pick to skip ahead.
- Ask follow-ups when answers reveal more depth or a non-default choice.
- Cap a single round at ~3 questions.
- Skip when the user has signaled a fast / default run.

## Layout

```
┌──────────────────────────────────────────────────────────────────────────┐
│  [Client] · Commercial Contracts                  [PRIVILEGED]            │
│  Acme Vendor MSA — Interactive Review                                      │
│  ─────────────────────────────────────────────────────────────────────    │
│  Recommendation: Push back   ·   N findings (3 ● HIGH, 8 ● MED, 2 ● LOW)  │
│                                                                            │
│  Filter: [All] [● BLOCKING] [● HIGH] [● MEDIUM] [● LOW]                   │
│                                                                            │
│  ┌─────────────────────┬───────────────────────────────────────────────┐ │
│  │ § 8.2 Indemnification│  CLAUSE 8.2 — INDEMNIFICATION    ● HIGH        │ │
│  │ ● HIGH               │  ─────────────────────────────────────────     │ │
│  │ Cap base too short   │                                                │ │
│  ├─────────────────────┤  THEIR LANGUAGE                                 │ │
│  │ § 10.1 Liability cap │  "Vendor's indemnification obligations are    │ │
│  │ ● BLOCKING           │   limited to direct damages and shall not     │ │
│  │ On Never-accept list │   exceed fees paid in the three (3) months    │ │
│  ├─────────────────────┤   preceding the claim."                         │ │
│  │ § 14 Audit (missing) │                                                │ │
│  │ ● MEDIUM             │  PROPOSED REPLACEMENT                           │ │
│  │ Insert audit right   │  "Vendor shall indemnify, defend, and hold     │ │
│  ├─────────────────────┤   harmless Customer..."                         │ │
│  │ ...                  │                                                │ │
│  │                      │  WHY                                            │ │
│  │                      │  Three-month cap base is on the Never-accept   │ │
│  │                      │  list per playbook position [Limitation of     │ │
│  │                      │  liability → Never accept].                    │ │
│  │                      │                                                │ │
│  │                      │  FALLBACK                                       │ │
│  │                      │  Cap base at 12 months of fees paid; carve     │ │
│  │                      │  out IP and data above the cap.                │ │
│  │                      │                                                │ │
│  │                      │  COMMENTS                                       │ │
│  │                      │  [+ add comment]                                │ │
│  └─────────────────────┴───────────────────────────────────────────────┘ │
│                                                                            │
│  Footer · PRIVILEGED & CONFIDENTIAL · Page X · [Reviewer]                 │
└──────────────────────────────────────────────────────────────────────────┘
```

## Interactivity

- **Sidebar:** Clickable list of findings. Severity dot on the left. Click a row
  to scroll the main pane to that clause.
- **Filter bar:** Vanilla JS show/hide by severity. State is preserved in the URL
  hash so a reviewer can bookmark "all blocking issues" and share the link.
- **Side-by-side mode (default):** Two columns inside the main pane — their
  language on the left, proposed on the right, with a synced scroll for long
  clauses.
- **Stacked mode:** Their language above, proposed below — better on narrow
  screens.
- **Drill-down mode:** Sidebar always visible; clicking expands the clause and
  collapses the others. Good for very long redlines (80+ entries).
- **Inline comments:** Each clause has a `[+ add comment]` button. Comments are
  stored in `localStorage` keyed by the file URL. Not shared — single-reviewer
  notes. Surfaces a warning if `localStorage` is empty when the file is opened
  from a different device.
- **Keyboard shortcuts:** `j` / `k` move between findings, `/` focuses the
  filter, `c` toggles the comment thread for the current clause.

## Tech constraints

- Single HTML file. Embedded CSS and JS.
- No frameworks — vanilla JS, ~150 lines.
- **Sanitize untrusted input.** Counterparty text, vendor names, OSS data — all
  set via `textContent`, never `innerHTML`. Use `DOMPurify` (vendored, ~20kb)
  if any retrieved content might contain HTML; otherwise plain `textContent`.
- Print-friendly: `@media print` collapses sidebar, expands all clauses, hides
  the comment widget.
- Works offline.

## Output

Save to:
`~/.claude/plugins/config/vallor/reports/<YYYY-MM-DD>-<slug>-interactive.html`

Open in browser if `--open` is passed.

## Decision tree

1. **Build the read-only PDF** — `/vallor:contract-report-html` then
   print to PDF.
2. **Build the approval deck** — `/vallor:contract-approval-deck`.
3. **Stop here** — file saved.

## Examples

```
/vallor:contract-interactive-html redlines/acme-msa.md
```

```
/vallor:contract-interactive-html redlines/acme-msa.md --mode drill-down
```
