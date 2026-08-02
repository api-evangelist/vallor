---
name: contract-exec-summary
description: >
  Produces a one-page executive summary of a contract or contract review — for a
  CEO, CFO, GM, or business owner who needs to decide in 90 seconds whether to
  sign, push back, or escalate. Outputs a single-page markdown (with PDF/PPTX
  export options) using the plugin's /design system. Use when the user says
  "exec summary", "summarize this for the CEO", "one-pager for the board",
  "C-suite version", "send this up", or after any review/redline skill produces a
  detailed memo that needs a shorter version up the chain.
argument-hint: "[contract-or-review-path] [--audience CEO|CFO|GM|board] [--format md|pdf|pptx]"
---

# /contract-exec-summary

## Purpose

A GC's job is partly translation: turning a 30-page memo into a paragraph the CEO
will actually read. This skill does that — and only that. It is not a stakeholder
summary (which is two paragraphs for procurement or a department head). It is the
shortest, sharpest version: one page, one recommendation, three numbers.

## Audience calibration

| Audience | Cares about | Doesn't care about |
|---|---|---|
| **CEO** | Strategic fit, dollars, risk of NOT doing the deal, biggest single liability | Clause-by-clause |
| **CFO** | Total spend, payment terms, exit cost, indemnity exposure, audit rights | Indemnity scope subtleties |
| **GM / business owner** | Can my team use this, what's the timeline, what's the exit | Liability cap math |
| **Board** | Strategic posture, precedent, governance signal | Anything below 🔴 |

Default to CEO if unspecified.

## Load context first

1. Read `~/.claude/plugins/config/vallor/CLAUDE.md`.
2. Read the source — either:
   - A contract draft (from `contract-prep` or an inbound PDF/DOCX),
   - A review memo (from `vendor-agreement-review` / `nda-review` / `saas-msa-review`),
   - Or both, if the user has run a review of an inbound draft.
3. Read the Contract Brief if one exists (it contains the business owner's name and
   risk posture — both inform tone).
4. Apply the **/design** skill for the page layout. **No emoji, no decoration, no
   marketing language.** A board summary that looks like a SaaS newsletter loses
   the room.

## Ask the user for context (optional but encouraged)

Use the `AskUserQuestion` tool to calibrate the summary. A CEO read is different from a CFO read; a one-page board sign-off is different from a quick GM heads-up.

**Up front (before drafting):**
Ask 1-3 short questions when not obvious from the inputs:
- Audience (if not passed via `--audience`): CEO / CFO / GM / board / mixed
- Decision being asked: sign / push back / escalate / informational only
- Sensitivities to amplify or soften (dollar exposure, IP, data, term length, exit)
- Format preference: markdown / PDF / one-slide PPTX

**Mid-flight (after reading the source):**
Once you've seen the full review or draft, surface concrete asks before guessing on:
- Which 3 issues actually move the needle for THIS reader (the review has 15; the page has room for 3)
- Whether to lead with the recommendation or the risk
- Whether to include a fallback ("if you push back, here's what we'd accept")

**Rules:**
- Every question must offer a reasonable default the user can pick to skip ahead.
- Ask follow-ups when answers reveal more depth or a non-default choice.
- Cap a single round at ~3 questions.
- Skip when the user has signaled a fast / default run.

## Output: one page, four blocks, no more

```markdown
# [Counterparty] · [Agreement type]

**Recommendation:** [Sign / Sign with deviation log / Push back / Walk]
**Total exposure:** [$X over [term]]
**One thing the reader must know:** [The single sentence the partner would lead with.]

---

## What this is

[Three sentences. What does the agreement let us do, what does it commit us to,
what does it cost.]

## Why it matters now

[Two sentences. Strategic context — why this deal, why this counterparty, why now.
Drawn from the brief's Q1 and Q3.]

## The three things that move the needle

| Issue | What it means | Position |
|---|---|---|
| [Highest-severity finding] | [What it costs us in plain terms] | [Sign / negotiate / walk] |
| [Second] | [Cost] | [Position] |
| [Third] | [Cost] | [Position] |

## If you say yes, here's what you're agreeing to

- **Spend:** [annual / total / payment schedule]
- **Term and exit:** [length, renewal, cancel-by, termination-for-convenience window]
- **Worst-case exposure:** [the cap — translated into dollars]
- **What we have to do:** [the top 2-3 ongoing obligations, plain English]

---

⚠️ Reviewer note: exec summary derived from [review memo / draft] · audience: [audience] · [full memo available at <path>] · contract brief at <path>
```

**Page limit.** If the markdown rendered as PDF is more than one US Letter page at
11pt, cut. The point of the summary is the cut. If the user wants more, point
them at the full memo.

## Format options

- **Markdown** (default) — saved to `~/.claude/plugins/config/vallor/exec-summaries/<slug>.md`.
- **PDF** — invoke the docx/PDF pipeline (see `references/docx-pipeline.md`).
- **PPTX** — single slide. Calls out to `/vallor:contract-approval-deck`
  with the `--single-slide` flag for the cover slide only.

## Tone and writing rules

- One sentence per point. No "the agreement provides that…" — "Acme gets X.
  We owe Y."
- No legalese. If a defined term is needed, define it inline in plain English.
- Numbers are bolded. Dates are bolded. Names are not.
- No "considerations." A summary that says "considerations" is a memo pretending
  to be a summary.
- The recommendation goes at the top, not the bottom. The reader who only reads
  the first line should still get the answer.

## What this skill does NOT do

- Doesn't produce a redline. That's `contract-redline`.
- Doesn't replace the full memo. The full memo is the work. This is the cover.
- Doesn't soften the recommendation. A "push back" is "push back," not
  "consider exploring opportunities to refine."

## Examples

```
/vallor:contract-exec-summary review-memos/acme-msa.md
```

```
/vallor:contract-exec-summary drafts/2026-05-12-acme-msa.md --audience CFO --format pdf
```

```
/vallor:contract-exec-summary --audience board
```
