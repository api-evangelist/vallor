---
name: design
description: >
  Visual design standards for every artifact this plugin produces — HTML reports, dashboards,
  PPTX approval decks, redline .docx files, stakeholder summaries. Locks output to a
  trusted, monotone, no-gradient look. Load this skill BEFORE generating any visual
  deliverable. Use when the user says "make this look good", "format this for the board",
  "what colors should I use", "design this report", "design the dashboard", or whenever
  any other skill in this plugin is about to write HTML, PPTX, or DOCX output.
---

# Commercial Legal — Design System

Legal work product is judged on trust before it's judged on content. A document with a
gradient, a swooshy hero image, or a "Generated with AI" stamp loses the room before the
first paragraph is read. This skill is the design floor every other skill in the plugin
inherits.

## Design philosophy

**The work is the work.** No decoration. No personality. The reviewing partner, the GC,
the procurement director, the board member — they read this in print, on a Bloomberg
terminal next to a Reuters alert, or on a Surface tablet at the M&A war room. The
document should look like it came from inside the law firm, not from a SaaS marketing
team.

Three tests every output must pass:
1. **Black-and-white printable.** If it loses meaning when printed in monochrome on plain
   paper, it fails. No color-only signals.
2. **No surprise.** The reader has seen documents like this 1,000 times. Theirs is
   indistinguishable from a Skadden cover memo or a Clifford Chance deal-room export.
3. **The numbers are the heroes.** A risk count, a savings figure, a renewal date — the
   eye lands there in under 2 seconds.

## Color palette — the only colors allowed

**Foreground / structure (used 95% of the time):**
- `#0B0B0B` — body text (near-black, not pure black)
- `#2A2A2A` — secondary headings and labels
- `#5B5B5B` — captions, metadata, footnotes
- `#9A9A9A` — muted borders, table dividers
- `#E5E5E5` — light dividers, table row separators
- `#F7F7F7` — page background tint (optional; default to pure white)
- `#FFFFFF` — page background

**Semantic accent — used SPARINGLY (a single dot or a single rule, never a fill):**
- `#1F3A5F` — primary accent (deep navy). Use for the masthead rule, the document
  title underline, a single section divider. Never a button. Never a gradient.
- `#7B1F1F` — caution / blocking (deep maroon). Use only on the `[BLOCKING]` finding
  marker dot or the `cancel-by` date. A maximum of three uses per document.
- `#5A5A2A` — caution / medium (muted olive). The medium-severity dot.
- `#3A3A3A` — settled / green-equivalent (we don't use green; "no issue" is the
  absence of a marker, not a green dot).

**Banned:**
- Gradients of any kind. Hard stop.
- Drop shadows beyond `0 1px 0 #E5E5E5` (a hairline rule).
- Border radii above 2px on any element.
- Any color outside the palette above.
- Emoji in body text, headings, or labels (allowed only in the legacy `[BLOCKING]` /
  `[HIGH]` markers in the `## Outputs` section of the practice profile, which use
  text labels — never decorative emoji like ✨ 🚀 💡).
- Drop caps, italicized headings, script fonts, all-caps section titles longer than
  one line.
- Any image that isn't a chart, a logo, or a verified document scan.

## Typography

**Body:** Either of these stacks (pick by environment):
- HTML web/report: `font-family: "Source Serif Pro", Georgia, "Times New Roman", serif;`
  for body; `"Inter", "Helvetica Neue", Arial, sans-serif;` for tables and metadata.
- PPTX: Calibri 11pt body, Calibri 18-24pt section heads.
- DOCX (redline): Times New Roman 11pt body, **bold** for inserted text and
  strikethrough for deleted (driven by track-changes XML, not styling).

**Hierarchy (HTML):**
- `h1` document title: 22pt, navy `#1F3A5F`, single 1pt rule underneath.
- `h2` section: 16pt, near-black `#0B0B0B`, no underline.
- `h3` subsection: 13pt, medium `#2A2A2A`.
- Body: 11pt, line-height 1.55.
- Footnote / caption: 9pt, `#5B5B5B`.

**Letter-spacing and weight:** Default. No tracking. Bold only where the law firm
would bold — defined terms, deal-breaker dollar amounts, dates with consequences.

## Layout

- Single column. Max content width 720px (HTML) or 6.5" (DOCX).
- Margins: 1" all around (DOCX/PPTX); 32px (HTML).
- Tables: 1pt rules, `#9A9A9A` borders, alternating row backgrounds only if more
  than 10 rows (`#FFFFFF` / `#F7F7F7`).
- No icons in tables. The dot signals (●) are text characters with a color, not
  images.

## Patterns

### Cover masthead (HTML / PPTX)

```
[CLIENT-NAME]                                       [PRIVILEGED & CONFIDENTIAL]
─────────────────────────────────────────────────────────────────────────────
Contract Review Memorandum                                          [DATE]
[Counterparty] — [Agreement type]
```

A single 1pt navy rule under the masthead. Nothing else.

### Severity markers (text-only, color-coded)

- `● BLOCKING` (color `#7B1F1F`)
- `● HIGH` (color `#7B1F1F`)
- `● MEDIUM` (color `#5A5A2A`)
- `● LOW` (color `#3A3A3A` — same as body)

Each marker is followed by a colon and one sentence. No icons, no badges.

### Pull quotes

Wrap the deal-breaker in a 4px left rule (`#1F3A5F`), 16px padding, italic,
`#2A2A2A`. One per document maximum.

### Charts (when a dashboard has them)

- Bar charts: bars in `#1F3A5F`, gridlines in `#E5E5E5`, axis labels in `#5B5B5B`.
- Sparklines and risk-distribution charts: same palette, 1pt strokes, no fills.
- Pie / donut charts: BANNED for legal work product (proportions are tables, not pie
  slices). Procurement plugin allows them sparingly; this one does not.

### Tables

- Header row: bold, near-black on `#F7F7F7`.
- Cell padding: 8px vertical, 12px horizontal.
- No vertical rules between columns. Horizontal `#E5E5E5` 1pt rule between rows.
- Right-align numeric columns. Left-align text columns.

## Boilerplate that lives in every deliverable

**Footer (every page):**

```
PRIVILEGED & CONFIDENTIAL — ATTORNEY WORK PRODUCT
Page X of Y · Prepared [DATE] · [Reviewer name from practice profile]
```

(Or the jurisdiction-correct equivalent — see `CLAUDE.md` → `## Outputs`.)

**Header (HTML reports and PPTX only):**

```
[CLIENT-NAME] · Commercial Contracts
```

11pt, `#5B5B5B`, right-aligned, no decoration.

## How other skills load this

Every visual-output skill in this plugin must start with:

> Read `skills/design/SKILL.md` first. Apply the palette, typography, and layout
> rules verbatim. Banned items are banned. If the requested output conflicts with the
> design system, surface the conflict and ask before overriding.

The design system is non-negotiable for plugin-generated artifacts. The user can
override after delivery; the skill never overrides on its own.

## Quick reference (paste at the top of an HTML doc)

```html
<style>
  :root {
    --ink:#0B0B0B; --ink-2:#2A2A2A; --mute:#5B5B5B; --line:#E5E5E5;
    --rule:#9A9A9A; --bg:#FFFFFF; --tint:#F7F7F7;
    --navy:#1F3A5F; --crit:#7B1F1F; --warn:#5A5A2A;
  }
  body { font: 11pt/1.55 "Source Serif Pro", Georgia, serif; color: var(--ink); background: var(--bg); margin: 32px auto; max-width: 720px; padding: 0 16px; }
  h1 { font-size: 22pt; color: var(--navy); border-bottom: 1px solid var(--navy); padding-bottom: 6px; }
  h2 { font-size: 16pt; color: var(--ink); margin-top: 28px; }
  h3 { font-size: 13pt; color: var(--ink-2); margin-top: 20px; }
  table { width: 100%; border-collapse: collapse; font-family: Inter, Arial, sans-serif; font-size: 10pt; }
  th { background: var(--tint); text-align: left; padding: 8px 12px; font-weight: 600; }
  td { padding: 8px 12px; border-top: 1px solid var(--line); }
  .num { text-align: right; font-variant-numeric: tabular-nums; }
  .dot-crit { color: var(--crit); }
  .dot-warn { color: var(--warn); }
  .footer { font-size: 9pt; color: var(--mute); border-top: 1px solid var(--line); margin-top: 32px; padding-top: 8px; }
  .pull { border-left: 4px solid var(--navy); padding: 12px 16px; color: var(--ink-2); font-style: italic; margin: 16px 0; }
</style>
```

The same variable names appear in every plugin-produced HTML file. Consistency means
a partner who has read one report has read them all.
