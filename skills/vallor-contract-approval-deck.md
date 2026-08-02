---
name: contract-approval-deck
description: >
  Generates a board-grade PPTX deck for the contract approval committee — cover,
  recommendation, financial summary, deviations table, risk register, decision-ask
  slide. 6-10 slides, monotone, no decoration. Uses the plugin's /design system.
  Use when the user says "approval deck", "deck for the committee", "approval pptx",
  "presentation for the board", "deck for sign-off", "executive deck for this
  contract", or after any review/redline skill when a deck is needed for sign-off.
argument-hint: "[contract-or-review-path] [--audience committee|board|exec-staff] [--single-slide]"
---

# /vallor:contract-approval-deck

## Purpose

When a contract needs sign-off from a deal committee, board, or exec staff, the
ask is always the same shape: here's what we're buying/selling, here's what it
costs, here's where it deviates from standard, here's the risk, and here's the
decision we need. This skill produces a deck that asks for that decision in 6-10
slides. No more.

## Load context

1. Read `~/.claude/plugins/config/vallor/CLAUDE.md`.
2. Read the source: a contract draft, a review memo, or both. Read the Contract
   Brief if one exists — its risk posture and business owner drive the framing.
3. Read the deviation log (from `contract-prep` or `contract-redline`) — it
   becomes the deviations slide.
4. **Apply the /design skill** — the deck uses Calibri 11/18/24pt, navy
   `#1F3A5F` rules, no other colors except the severity markers. No gradients,
   no rounded rectangles, no images other than logos. Single column layouts.

## Ask the user for context (optional but encouraged)

Use the `AskUserQuestion` tool to calibrate the deck to its audience and decision. A 6-slide ask to a board reads differently than a 10-slide ask to an exec staff.

**Up front (before drafting slides):**
Ask 1-3 short questions when the answers aren't obvious from the brief or review:
- Audience: deal committee / board / exec staff / mixed
- The single decision being asked: sign as-is / sign with redlines / escalate / walk
- Length preference: tight 6-slide ask vs full 10-slide context
- Sensitivities to soften or amplify (e.g., dollar exposure, IP risk, exit-rights, vendor relationship)
- Speaker: who is presenting and how much they know about the deal

**Mid-flight (after building the deviations and risk slides):**
Once the deviations table is drafted, surface concrete asks before guessing on:
- Which deviations to elevate to the recommendation slide
- Whether to include alternatives (walk-away cost) on a separate slide
- How to frame the ask if the recommendation is a conditional yes

**Rules:**
- Every question must offer a reasonable default the user can pick to skip ahead.
- Ask follow-ups when answers reveal more depth or a non-default choice.
- Cap a single round at ~3 questions.
- Skip when the user has signaled a fast / default run.

## Deck structure (6-10 slides, no more)

### Slide 1 — Cover

```
[CLIENT NAME] · Contract Approval                            [PRIVILEGED &
                                                              CONFIDENTIAL]
─────────────────────────────────────────────────────────────────────────────

[Counterparty] · [Agreement type]
[Total contract value] · [Term length]

Prepared for: [committee / board / exec staff]
Date: [today]
Owner: [reviewing attorney]
```

A single 1pt navy rule under the masthead. Vallor + customer logos in the
bottom-right corner at 24px height (use real PNG files; never a text span — see
`feedback_real_logos_in_brand_output`).

### Slide 2 — Recommendation (the decision ask)

A single sentence at the top:

```
RECOMMENDATION: Approve · Approve with conditions · Push back · Walk
```

Below: three bullets explaining the recommendation. Each bullet ≤15 words.
Below that: a single "what we're asking for today" line:

```
DECISION REQUESTED TODAY: [Authorize counsel to execute / authorize counter-proposal at $X / authorize escalation to GC / etc.]
```

This is THE slide the committee remembers. Every other slide supports it.

### Slide 3 — Commercial summary

Two-column layout:

| Left (deal shape) | Right (financials) |
|---|---|
| Counterparty: [name] | Annual value: [$X] |
| Type: [MSA / SaaS / SOW / etc.] | Total contract value: [$Y] |
| Scope: [one-line description] | Payment terms: [Net 30 / annual upfront / etc.] |
| Term: [start — end] | Cap on liability: [$Z or X months fees] |
| Renewal: [auto / fixed / evergreen] | Worst-case exposure: [the number that matters] |
| Notice period: [days] | Exit cost: [the cancel-fee or zero] |

### Slide 4 — Why this deal (strategic context)

Pulled from the Contract Brief Q1 + Q3. Three bullets:
- Business intent (what does this let us do)
- Strategic context (why this counterparty, why now)
- Deadline / urgency (if any)

### Slide 5 — Deviations from standard

Three-column table. Severity dot in the left margin (text characters, no images):

| Position | Standard | This deal | Severity |
|---|---|---|---|
| Liability cap | [playbook position] | [this deal's position] | ● HIGH |
| Indemnity | [position] | [position] | ● MEDIUM |
| ... | ... | ... | ● ... |

Pulled directly from the deviation log. If more than 6 rows, this becomes its
own slide and we cap at "top 6 by severity" with "5 more deviations in the
appendix" line.

### Slide 6 — Risk register

A short table — max 5 risks. For each:

| Risk | Probability | Impact ($) | Mitigation |
|---|---|---|---|

Risks come from the review memo's findings table, filtered to high+blocking
severity. If no significant risks, this slide is replaced by an "Assessed risk:
within tolerance" callout.

### Slide 7 — Negotiation history (optional)

Only if there's a meaningful history (this is v3 of the draft, the counterparty
came back twice, etc.). Three bullets:
- Where we started
- Where we are now
- What we conceded vs gained

Skip if the deck is for a first-pass approval.

### Slide 8 — Approval matrix

Per the practice profile's escalation matrix, which approvers are needed for
this deal:

```
PRE-EXECUTION APPROVERS NEEDED:
  ● [Approver 1 name and role]        [their authority threshold]
  ● [Approver 2]                       [threshold]

EXECUTING SIGNER:                       [from practice profile]

POST-EXECUTION DISTRIBUTION:            [CLM record, business owner, etc.]
```

### Slide 9 — The one question

A single line at the top:

```
ONE QUESTION I'D ASK THAT ISN'T IN MY CHECKLIST:
```

Below: 1-3 sentences pulled from the review memo's "One question" section. This
is the slide that catches the second-order risk a framework wouldn't prompt for.

### Slide 10 — Decision options (the close)

Four options in a 2x2 grid:

1. **APPROVE** — Sign as-is, execute by [date]
2. **APPROVE WITH CONDITIONS** — Sign once [the 1-2 specific changes] land
3. **COUNTER** — Authorize [reviewer] to propose [the specific counter-position]
4. **WALK** — Authorize closing this deal

The recommended option is bordered with a 2pt navy rule. The others are 1pt
gray.

## Output

Save to:
`~/.claude/plugins/config/vallor/approval-decks/<YYYY-MM-DD>-<counterparty>-approval.pptx`

Use `python-pptx`. Reference implementation at `references/pptx-pipeline.md`.

## Single-slide mode

`--single-slide` produces just slide 1 — used by `contract-exec-summary` when the
user wants a cover slide for a one-pager packet.

## Constraints

- 10 slides max. If the deck is creeping toward 12, cut. Use an appendix for
  detail.
- No animations, no transitions, no fade-ins.
- No emoji on slides (the severity dots are text characters with color, not
  emoji).
- Every slide has the footer:
  `PRIVILEGED & CONFIDENTIAL — ATTORNEY WORK PRODUCT · Page X · [Owner]`
- Speaker notes: keep them short. Each slide gets 2-3 sentences of notes — what
  the presenter should say if asked to expand. Don't write a script.
- Single body font (Calibri). No second font, no italic accents, no script.

## Decision tree after delivery

1. **Open in PowerPoint and review** — confirm rendering before sending.
2. **Convert to PDF** — for board distribution where editability isn't wanted.
3. **Build the executive one-pager** — `/vallor:contract-exec-summary`
   if a parallel one-pager is needed.
4. **Stop here** — deck is saved.

## Examples

```
/vallor:contract-approval-deck review-memos/acme-msa.md
```

```
/vallor:contract-approval-deck --audience board
```

```
/vallor:contract-approval-deck drafts/acme-msa.md --single-slide
```
