---
name: contract-brief
description: >
  Captures the business intent behind a contract before any drafting happens. A short
  intake interview — who, what, why, when, how much, what's the risk tolerance — and
  produces a one-page Contract Brief that every downstream skill (drafting, redlining,
  approval deck, exec summary) reads. Use when the user says "fill in a contract brief",
  "kick off a new contract", "intake this deal", "start a contract", "I have a deal to
  paper", "contract prep", or whenever a new agreement is about to be drafted or
  negotiated.
argument-hint: "[--deal-type vendor|customer|partner|nda|sow] [--counterparty <name>] [--existing-brief <path>]"
---

# /contract-brief

## Purpose

The most expensive mistake in contracting is realizing on draft 3 that nobody asked the
business what they were actually trying to do. This skill front-loads the question.
Ten minutes of intake at the top saves an hour of redlining at the bottom.

The output is a single page — markdown, plain — saved to the matter folder. Every
downstream skill (contract-redline, contract-exec-summary, contract-approval-deck,
review) reads it before doing anything.

## Load context first

1. Read `~/.claude/plugins/config/vallor/CLAUDE.md`. If
   placeholders are present, stop and prompt for cold-start.
2. Apply the **/design** skill — even though the brief is mostly text, the file uses
   the standard masthead and footer.
3. Determine the **side** (sales / purchasing) using the same logic as `vendor-agreement-review`.
   If ambiguous, ask.

## Intake — six questions, fixed order

Ask these one at a time, in order. Don't batch them; the answer to one informs the
next. Use the user's words — don't paraphrase.

1. **What are we trying to accomplish?**
   > "What does this contract let us do that we can't do today? In one sentence."

2. **Who's the counterparty, and what's the relationship?**
   > "Who are we signing with? Have we worked with them before? New, renewal, or
   > expansion?"

3. **What's at stake — money, time, or strategy?**
   > "Approximate ACV or total contract value, even a range. Term length. And is this
   > a 'must close by Friday' deal or a 'no rush, get it right' deal?"

4. **What are the must-haves?**
   > "Anything that, if we don't get it, the deal doesn't close on our side? Pricing
   > floor, data residency, exclusivity, an SLA, an exit right?"

5. **What are the give-ups we're willing to make?**
   > "Where can we move? Which playbook positions are we already prepared to soften,
   > and what do we want in return?"

6. **Who owns this on the business side, and what's their risk tolerance for this deal?**
   > "Who do I send the stakeholder summary to? Are they conservative, balanced, or
   > 'just close it'?"

After question 6, ask one consolidating question:

> "Anything that doesn't fit the questions above but I should know? Pending litigation,
> a regulatory deadline, a related vendor we're consolidating, an exec who has strong
> feelings about this one?"

## Output: Contract Brief

Write to `~/.claude/plugins/config/vallor/briefs/<slug>.md`
(or the active matter folder if matter workspaces are on). Slug format:
`<YYYY-MM-DD>-<counterparty-kebab-case>`.

```markdown
# Contract Brief — [Counterparty]

**Side:** [sales / purchasing]
**Deal type:** [MSA / SaaS / NDA / SOW / order form / amendment / other]
**Status:** intake complete · ready for [drafting / review / negotiation]
**Date:** [YYYY-MM-DD]
**Business owner:** [name]
**Reviewing attorney:** [name from practice profile]
**Risk posture for this deal:** [conservative / balanced / aggressive — from Q6]

---

## Business intent

[Verbatim or near-verbatim from Q1. One sentence.]

## Counterparty

- Name: [legal entity]
- Relationship: [new / existing / renewal / expansion]
- Prior agreements: [list any found in the CLM or matter folder]
- BigCo vs startup: [tag — affects negotiation latitude]

## Commercial shape

- ACV / TCV: [$X / range / unknown — flag if unknown]
- Term: [length, renewal mechanic]
- Deadline pressure: [hard date / quarter-end / no pressure]
- Notable: [tooling owned, exclusivity ask, multi-year discount, etc.]

## Must-haves (deal-breakers on our side)

- [Item 1 from Q4 — verbatim where possible]
- [Item 2]
- ...

## Give-ups (where we can move, and what we want in return)

| We'll move on… | If we get… |
|---|---|
| [item] | [item] |

## Open questions / context

- [Item from the consolidating question]

---

## Side-specific playbook check

Before drafting begins, the following playbook positions apply to this deal. Pulled
from `~/.claude/plugins/config/vallor/CLAUDE.md` →
`## Playbook` → `### [Sales / Purchasing]-side playbook`:

- **Limitation of liability:** [the practice profile's standard cap and carveouts]
- **Indemnification:** [position]
- **Data protection:** [position]
- **Term and termination:** [position]
- **Governing law / venue:** [position]
- **The one thing:** [the deal-breaker check]

[BLOCKING items already visible from intake: list them here. Otherwise: "No blocking
items visible at intake."]

---

## Next steps

The next step on the decision tree:

1. **Draft from our paper** — I'll produce a first-draft MSA/SOW/order form using the
   practice profile defaults, with the must-haves above hardcoded and the give-ups
   pre-positioned as fallbacks.
2. **Review their paper** — they've sent a draft. Run `/vallor:review
   <file>`; the review will read this brief first.
3. **Build a redline** — they sent terms; I'll diff against the playbook and produce
   a tracked-change `.docx`.
4. **Stop here** — brief is filed; resume when the document arrives.

---

⚠️ Reviewer note: brief intake only · no document reviewed · playbook positions
loaded from practice profile · attorney to confirm risk posture for Q6
```

## After writing the brief

- Show the brief to the user.
- Ask: "Anything wrong? I'll edit before we move on."
- Offer the four next-step options above.
- If matter workspaces are on, update `matters/<slug>/matter.md` with a one-line
  pointer to the brief.

## Re-running on an existing brief

If the user passes `--existing-brief <path>`, load it, show the current state, and
ask which of the six questions they want to update. Re-write only the changed
sections. Append a `## Revisions` block at the bottom with the date and a one-line
summary of what changed.

## Constraints

- This skill is intake only. It does NOT draft contract language. The output is a
  Contract Brief, not a draft.
- It does NOT make legal judgments — it captures business intent for legal to act on.
- It runs before any document exists OR alongside a draft to capture intent the
  document doesn't reflect.

## Examples

```
/vallor:contract-brief
```

```
/vallor:contract-brief --deal-type vendor --counterparty Acme
```

```
/vallor:contract-brief --existing-brief briefs/2026-05-12-acme.md
```
