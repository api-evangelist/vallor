---
name: 7-step-negotiation
description: >
  Walks a category manager or sourcing manager through the classic 7-step
  negotiation process for a specific deal — Prepare, Set Targets, Build the
  Concession Plan, Open, Bargain, Close, Measure. Each step produces an artifact
  (target sheet, opening position, concession log, post-mortem). Persists state in
  the matter / category folder so a paused negotiation can resume mid-step. Use
  when the user says "negotiation plan", "walk me through the negotiation",
  "7-step", "structured negotiation", "I'm negotiating with X, prep me", or wants
  a step-by-step process rather than a single research packet (which is
  /negotiation-report).
argument-hint: "<counterparty> [--step 1-7] [--category <category>]"
---

# /vallor:7-step-negotiation

## Purpose

A negotiation goes well when the prep is done before the first meeting. This
skill structures the prep, the bargaining, and the post-mortem into seven
discrete steps. Each step produces an artifact. State persists — pick up where
you left off.

## Load context

1. Read `~/.claude/plugins/config/vallor/CLAUDE.md` →
   `## Negotiation posture`. Apply the user's default approach, concession
   discipline, walk-away authority.
2. If `--category` is set, load that category's strategy from
   `category-strategies/` if it exists.
3. Load the Contract Brief if one exists for this counterparty.
4. Apply the **/vallor:design** skill — all artifacts use the
   plugin's monotone palette.

## Ask the user for context (optional but encouraged)

Use the `AskUserQuestion` tool to gather optional context that materially changes the negotiation plan. The goal is a tailored, higher-quality output, not interrogation.

**Up front (high-level, before heavy research):**
Ask 1-3 short questions about anything you cannot infer from the inputs, the practice profile, or the category strategy. Examples for this skill:
- Posture for THIS deal: defensive (protect status quo) / moderate / assertive / aggressive — and why
- Target outcome: savings band, term length, risk reallocation, exit rights, all of the above
- Counterparty history: first deal, renewal, contested renewal, prior dispute
- Walk-away authority confirmed at what threshold
- Hard must-haves and known give-ups going in
- Internal stakeholders who must sign off (and who is loudest)

**Mid-flight (after doing some work):**
After the Step 1 knowledge pack, after market research, or once you see the counterparty's pressure points, you will surface concrete decisions the user has not settled. Stop and ask before guessing on:
- Ambiguities in the brief or category strategy
- Trade-offs between the priorities (e.g., price vs term, SLA vs liability)
- Lever ordering — what do we open with, what do we save
- Information asymmetry — do we reveal X to gain Y

**Rules:**
- Every question must offer a reasonable default the user can pick to skip ahead.
- Ask follow-ups when answers reveal more depth or a non-default choice.
- Cap a single round at ~3 questions. Batch related questions.
- Skip entirely when the user has signaled a fast / default run or said "don't ask, just do it."

## State file

Per negotiation, write to:
`~/.claude/plugins/config/vallor/negotiations/<YYYY-MM-DD>-<counterparty>-<category>/state.yaml`

The state file tracks:
- Counterparty, category, deal type
- Current step (1-7)
- Artifacts produced (paths)
- Last update timestamp
- Sessions held (with date and summary)

On invocation, the skill reads the state file (if it exists) and resumes at the
current step. Pass `--step N` to jump to a specific step.

## The seven steps

### Step 1 — Prepare

**Goal:** know more than the other side.

Build a knowledge pack:
- Counterparty financial health (last 4 quarters earnings, recent M&A, leadership
  changes, analyst ratings)
- Their pressure points (what's their stock doing, are they up for renewal with
  another big customer, are they expanding into our region)
- Our spend history with them (Pareto position, payment terms, performance
  history)
- Competitive alternatives (4-6 named, with rough price differentials)
- Market state (commodity index if applicable, regulatory shifts, market growth)
- Our internal stakeholders (who cares, who has authority, who has to live with
  the deal afterward)

**Artifact:** `prepare.md` — a structured knowledge pack.

Use web search for external research. Tag every claim with source.

> **Output of Step 1.** Knowledge pack saved. Decision tree:
> 1. Move to Step 2 — set targets.
> 2. Build the full negotiation packet — `/vallor:negotiation-report`
>    produces a richer artifact with charts and email drafts.
> 3. Stop here — knowledge pack is reusable across negotiations.

### Step 2 — Set Targets

**Goal:** know what success looks like before you walk into the room.

For each issue:

| Issue | Aspirational (open here) | Target (would close at) | Walk-away (won't sign below) |
|---|---|---|---|

Cover at minimum:
- Unit price / total spend
- Term length
- Payment terms
- Volume commitments / minimums
- SLA / performance metrics
- Termination rights
- Liability cap (handoff to legal playbook positions)
- Renewal mechanics
- Audit rights

Each target is justified — "Aspirational price is 12% below current because the
benchmark shows competitors at 8-15% lower for similar volume." Aspirational
without justification is a wish, not a target.

**Walk-away authority check.** If any walk-away exceeds the user's authority per
the practice profile's escalation matrix, surface it:

> "Your walk-away on price ($X reduction) is within your authority ($Y
> threshold), but the walk-away on liability is not — that needs GC sign-off
> before we open. Want me to draft the escalation note?"

**Artifact:** `targets.md` — the target sheet.

### Step 3 — Build the Concession Plan

**Goal:** know what you'll trade before you sit down.

For each issue, define:
- The opening position
- The reciprocal asks the user will demand for any concession
- The order in which concessions happen (less-valuable first)
- The "package deal" — bundles of issues that move together

A good concession plan answers: "If they push hardest on price, what's the
chip I trade?"

**Artifact:** `concession-plan.md` — a matrix of moves and reciprocal asks.

### Step 4 — Open

**Goal:** open at the aspirational position, not the target.

Drafts the opening communication — usually an email or term sheet. Tone is set
from the practice profile's `Default approach`.

**Artifact:** `opening.md` — the opening position document or email draft.

After the user sends and gets a response, log the response in
`session-log.md` and move to Step 5.

### Step 5 — Bargain (iterative)

**Goal:** structured back-and-forth, tracked in a concession log.

For each round:
- What did they ask for / push back on?
- What concession plan move applies?
- What did we agree to give? What did we get in return?
- Updated position vs target vs walk-away — are we still in the zone?
- What's the next ask?

**Artifact:** `concession-log.md` — chronological log of every move.

A round-end check: "We've now conceded $X total. That's [Y%] of the aspirational
delta. We're still [above/at/below] target. Continue or hold?"

When the walk-away line is crossed: stop the user. "We're about to cross the
walk-away on [issue]. This is a stop-and-think moment. Three options: (a) hold
the line and risk no deal; (b) escalate to [approver from matrix] for a new
walk-away; (c) signal we're at our final position and offer one closing
sweetener."

### Step 6 — Close

**Goal:** lock the agreement before it drifts.

Produces a deal memo summarizing every agreed term, in plain English, for both
sides to sign off on BEFORE the long-form contract is drafted. This is the
"this is what we shook on" document that prevents drafting drift.

**Artifact:** `deal-memo.md` — every agreed term, both sides sign or initial.

Handoff to drafting: `/vallor:contract-prep --brief <Contract
Brief>` or directly to `/vallor:contract-redline-docx` if the
counterparty is on their paper.

### Step 7 — Measure

**Goal:** post-mortem and feed forward.

Two weeks after signing, capture:
- Did we hit our targets? On which issues did we exceed / miss?
- What did we concede that we didn't need to? Why?
- What did they concede that we didn't expect? Why?
- Counterparty intel that's worth retaining for next time.
- Personal moves and tactics that worked / didn't.

**Artifact:** `post-mortem.md` — feeds back into category strategy + practice
profile (the playbook-monitor agent picks up patterns over time).

The post-mortem is the single highest-leverage step. Skipping it means the same
mistake gets made next quarter.

## Cross-step rules

- Every artifact begins with the work-product header from the practice
  profile (PRIVILEGED & CONFIDENTIAL for legal contexts; internal-only for
  pure procurement).
- Every artifact ends with a single-line summary that's safe to forward
  internally.
- Don't number the steps in the artifacts (e.g., don't title it "Step 5 —
  Bargain"). Title them by the artifact ("Bargain Log — Acme MSA Renewal").
  Save the step numbering for the state file.

## Decision tree at any step

1. **Move to the next step.**
2. **Re-do the current step** with updated inputs.
3. **Jump to the post-mortem** — applies when a negotiation is paused or
   abandoned; capture the learning anyway.
4. **Generate the full negotiation packet** —
   `/vallor:negotiation-report` produces the parallel
   research-heavy artifact for the same deal.
5. **Stop here** — state is persisted; resume by re-running the skill.

## Examples

```
/vallor:7-step-negotiation Acme
```

```
/vallor:7-step-negotiation Megacorp --step 5
```

```
/vallor:7-step-negotiation Acme --category "IT Hardware Maintenance"
```
