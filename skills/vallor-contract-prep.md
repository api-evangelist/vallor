---
name: contract-prep
description: >
  Drafts a first-pass contract document from a populated Contract Brief — pulls the
  matching template (MSA, SOW, NDA, order form, amendment), inserts the playbook's
  standard positions, hardcodes the must-haves, and pre-positions the give-ups as
  fallback language. Produces a clean draft ready for /vallor:review or
  /vallor:contract-redline. Use when the user says "prep a contract",
  "draft an MSA", "start the SOW", "build the first draft", "we're on our paper",
  "kick off the draft", or after /vallor:contract-brief has produced a brief.
argument-hint: "[--brief <path>] [--template msa|sow|nda|order-form|amendment]"
---

# /contract-prep

## Purpose

When the user is on their own paper (sales-side for a SaaS company, purchasing-side
for a procurement-heavy buyer), the first draft is mostly mechanical: load the
template, plug in the parties and dollar values, set the playbook positions, and
flag where this deal departs from standard. This skill does that — and stops before
anything subjective.

## Load context first

1. **Read the practice profile.** `~/.claude/plugins/config/vallor/CLAUDE.md`.
   If placeholders: stop, route to cold-start.
2. **Load the Contract Brief.** If `--brief` is passed, use that path. Otherwise,
   look in `~/.claude/plugins/config/vallor/briefs/` for
   the most recent. If no brief exists, stop and prompt:
   > "I need a Contract Brief before I draft. Run `/vallor:contract-brief`
   > first — 10 minutes — so the draft reflects what the business is actually trying
   > to do. Or paste the key facts inline and I'll work from those."
3. **Pick the template.** Inferred from `Deal type:` in the brief, or from
   `--template`. Templates live at:
   - `~/.claude/plugins/config/vallor/templates/<type>.md`
   - If none exists, this skill writes one based on the practice profile and asks
     the user to confirm it as the default going forward.
4. **Apply the /design skill** to whatever format the draft uses.

## Ask the user for context (optional but encouraged)

Use the `AskUserQuestion` tool to gather optional context the Contract Brief may not cover but that materially shapes the draft. The brief captured business intent; this fills in drafting choices.

**Up front (before drafting):**
Ask 1-3 short questions when not obvious from the brief and practice profile:
- Posture for THIS deal: tight playbook positions / standard / pre-positioned fallbacks (give-ups already softened)
- Counterparty leverage: small / equal / dominant — affects which fallbacks to plant
- Deal urgency: must-sign date or open-ended
- Hard must-haves on top of the standard playbook (e.g., specific carve-outs, MFN, exclusivity)
- Whether to include or omit optional sections (SLAs, change orders, security exhibits)

**Mid-flight (after picking the template and slot-filling):**
Once you have the variable slots identified, surface concrete asks before guessing on:
- Variables not in the brief (e.g., specific dollar denominators, term-renewal cadence, governing-law tie-breaker)
- Whether to include the more-aggressive position or the safer fallback when both are in the playbook
- Whether to leave optional clauses in (commented) for the reviewer or strip them

**Rules:**
- Every question must offer a reasonable default the user can pick to skip ahead.
- Ask follow-ups when answers reveal more depth or a non-default choice.
- Cap a single round at ~3 questions.
- Skip when the user has signaled a fast / default run.

## Workflow

### Step 1 — Confirm the template before drafting

> "I'm going to draft a [MSA / SOW / NDA / Order Form / Amendment] using your
> [template name or 'first-time template — I'll save it after we agree on it'].
> Side: [sales / purchasing]. Counterparty: [name]. Dollar value: [$X]. Sound right?"

Wait for confirmation. If the user corrects, apply and re-confirm.

### Step 2 — Identify variable slots

Read the template and identify every variable. Categorize each:

| Variable | Source |
|---|---|
| Party names, addresses, signatories | Brief + counterparty research |
| Dollar values, term, dates | Brief |
| Playbook positions (liability cap, indemnity, governing law, term renewal, etc.) | Practice profile |
| Must-haves (Q4 in the brief) | Brief — hardcode into the matching clauses |
| Give-ups (Q5 in the brief) | Brief — write as the standard position with a comment indicating which fallback language to apply during negotiation |
| Counterparty-specific terms | Stop here. Surface to user. |

### Step 3 — Draft

Produce the draft in the requested format:
- Markdown (default — clean, diffable, easy to redline downstream)
- `.docx` (when the user wants to send it for signature or to opposing counsel —
  invoke the `pandoc` toolchain or the `python-docx` library, see
  `references/docx-pipeline.md`)

**Drafting rules:**
- Use the playbook's standard position for every clause covered by the playbook.
- For must-haves: write the deal-specific position directly, with a marker comment:
  `<!-- must-have from brief: [Q4 item] -->`.
- For give-ups: write the playbook's *opening* position (not the fallback), and add
  a marker comment showing the fallback the team has pre-authorized:
  `<!-- give-up from brief: open with [standard], fall back to [fallback] in
  exchange for [reciprocal ask] -->`.
- For anything outside the playbook: write `[TO BE DETERMINED — surfaces to
  attorney for first decision]` and list it at the end.

### Step 4 — Deviation log

At the bottom of the draft (or as a separate `<slug>-deviation-log.md`), produce:

```markdown
# Deviation Log — [Counterparty] [Deal type]

Positions in this draft that depart from the practice profile's standard playbook.

| Clause | Standard playbook position | This draft's position | Source | Sign-off needed |
|---|---|---|---|---|
| [Clause] | [from CLAUDE.md] | [this draft] | [brief Q4 / brief Q5 / TBD] | [GC / counsel / paralegal — per escalation matrix] |
```

If the deviation log has rows, surface them above the draft when delivering:

> "**Heads up — three positions in this draft depart from your standard playbook.**
> They're recorded in the deviation log at the bottom. Highest is the [item], which
> per your escalation matrix needs [approver]'s sign-off. Want me to draft the
> escalation note?"

### Step 5 — Save and offer next steps

Save the draft to:
`~/.claude/plugins/config/vallor/drafts/<YYYY-MM-DD>-<counterparty>-<type>.md`

(Or the matter folder if workspaces are on.)

Decision tree:

> **What next? Pick one:**
> 1. **Send for internal review** — I'll route to [reviewer from escalation matrix]
>    with a short note explaining the deviations.
> 2. **Build an executive summary** — `/vallor:contract-exec-summary` —
>    one-page for the business owner before we ship the draft to the counterparty.
> 3. **Convert to .docx for sending** — `/vallor:contract-redline` will
>    produce the polished .docx; or I can do a quick pandoc-style conversion now if
>    no redlines are needed.
> 4. **Build the approval deck** — `/vallor:contract-approval-deck` —
>    PPTX for the deal-approval committee.
> 5. **Stop here** — draft is saved; resume when ready.

## What this skill does NOT do

- It doesn't negotiate. The give-ups are pre-positioned, not pre-conceded.
- It doesn't promise the counterparty will accept the draft. The deviation log
  surfaces what the user must defend.
- It doesn't run a deviation review against an incoming draft — that's
  `vendor-agreement-review`. This skill produces an outgoing draft.

## Examples

```
/vallor:contract-prep --brief briefs/2026-05-12-acme.md
```

```
/vallor:contract-prep --template sow
```

```
/vallor:contract-prep
```
