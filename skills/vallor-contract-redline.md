---
name: contract-redline
description: >
  Produces a markdown redline of a contract against the practice profile playbook —
  side-by-side or unified diff, with proposed insertions, deletions, and explanatory
  comments tagged by severity. The text-first counterpart to /vallor:contract-redline-docx
  (which produces a tracked-changes .docx for sending to opposing counsel). Use when
  the user says "redline this contract", "show me the changes", "diff against the
  playbook", "mark up this MSA", "redline against our standard", or "build the
  redline" without specifying a format.
argument-hint: "[contract-path] [--against playbook|prior-version] [--mode unified|side-by-side]"
---

# /contract-redline

## Purpose

The deviation memo says what's wrong; the redline says what to change. This skill
produces a **proposed-language** redline in markdown — readable in a terminal,
diffable in git, pasteable into a CLM, and the input to `contract-redline-docx`
when the redline needs to leave the building.

## Load context first

1. Read `~/.claude/plugins/config/vallor/CLAUDE.md`.
   Stop on placeholders.
2. Determine the side (sales / purchasing). Same logic as `vendor-agreement-review`.
3. Read the Contract Brief if one exists — its must-haves and give-ups directly drive
   the redline.
4. Apply the /design skill for the rendered output's typography and tables.
5. If `--against prior-version`, locate the prior version in the matter folder or
   the CLM. Otherwise default to playbook.

## Ask the user for context (optional but encouraged)

Use the `AskUserQuestion` tool to calibrate the redline posture. The same contract gets a different markup at "first-pass with a strategic vendor" than at "final round with an off-pattern small vendor."

**Up front (before redlining):**
Ask 1-3 short questions when not obvious from the brief and practice profile:
- Posture: defensive (mirror playbook tightly) / moderate / assertive / aggressive
- Counterparty relationship: strategic partner / preferred / transactional
- Round: first-pass / second-pass / final-round (drives whether to plant fallbacks or push the must-haves)
- Format preference: unified / side-by-side
- Hot-buttons to elevate beyond the playbook (e.g., a specific data-residency requirement that's recent)

**Mid-flight (after the deviation map is built):**
Once you've mapped deviations, surface concrete asks before guessing on:
- Whether to redline every minor playbook drift or only items that meaningfully change risk
- How aggressive to be on counterparty's "comments only" zones (e.g., headings, definitions)
- Whether to insert proposed alternatives or just bracket the offending language
- Tone of inline comments: tight legal / explanatory / explanatory + business framing

**Rules:**
- Every question must offer a reasonable default the user can pick to skip ahead.
- Ask follow-ups when answers reveal more depth or a non-default choice.
- Cap a single round at ~3 questions.
- Skip when the user has signaled a fast / default run.

## Workflow

### Step 1 — Read the contract end-to-end

Per the practice profile's `## Large input` rule, record what you read in the
reviewer note. Don't pretend coverage you don't have.

### Step 2 — Build the deviation map

For each clause in the contract, classify:

| Class | Meaning | Action |
|---|---|---|
| **Match** | Playbook standard, no change | Skip |
| **Within tolerance** | Differs but within an accepted-fallback range | Flag in margin, no language change |
| **Deviation, fixable** | Differs and we have a known counter-position | Propose a redline |
| **Deviation, escalate** | Above the reviewer's authority per escalation matrix | Propose a redline AND flag for the escalation-flagger skill |
| **Deal-breaker** | On a Never-accept list | BLOCKING marker; propose deletion or rewrite |
| **Gap** | Missing clause that should be present | Propose insertion |

### Step 3 — Propose specific language

For each deviation, write the specific replacement. Don't say "consider revising
the indemnity to be more favorable." Say:

```
[Section 8.2 — Indemnification]

THEIR LANGUAGE:
"Vendor's indemnification obligations are limited to direct damages and shall not
exceed fees paid in the three (3) months preceding the claim."

PROPOSED REPLACEMENT:
"Vendor shall indemnify, defend, and hold harmless Customer from any third-party
claim arising from (a) Vendor's breach of its data protection obligations,
(b) Vendor's gross negligence or willful misconduct, or (c) any allegation that
the Services infringe a third party's intellectual property rights, without
regard to the limitation of liability set forth in Section 10."

WHY:
[One sentence — what this protects, drawn from the playbook position.]

FALLBACK if the counterparty pushes back:
"…without regard to the limitation of liability set forth in Section 10, except
that the aggregate cap for IP indemnification shall not exceed [2x annual fees]."

SEVERITY: ● HIGH
ESCALATION: [counsel approval — within authority] / [GC sign-off required]
```

### Step 4 — Render the document

Two modes:

**Unified diff** (`--mode unified`, default):
```markdown
# Redline — [Counterparty] [Agreement type]

[Reviewer note — see practice profile format]

---

## § 8.2 Indemnification  ● HIGH

- ~~"Vendor's indemnification obligations are limited to direct damages and
  shall not exceed fees paid in the three (3) months preceding the claim."~~
+ "Vendor shall indemnify, defend, and hold harmless Customer from any
  third-party claim arising from (a) Vendor's breach of its data protection
  obligations, (b) Vendor's gross negligence or willful misconduct, or (c) any
  allegation that the Services infringe a third party's intellectual property
  rights, without regard to the limitation of liability set forth in Section
  10."

> **Why:** Three-month cap base is on the Never-accept list. The proposed
> language carves IP, data, and gross negligence above the cap, consistent with
> playbook position.
> **Fallback:** [pre-positioned, from brief Q5]
> **Escalation:** within counsel authority.

---
```

**Side-by-side** (`--mode side-by-side`): two-column table, original on the
left, proposed on the right, marker in the middle column.

### Step 5 — Findings summary table

At the top of the redline (after the reviewer note, before the section-by-section
markup):

| § | Issue | Severity | Action | Approver |
|---|---|---|---|---|
| 8.2 | Cap base too short | HIGH | Replace | Counsel |
| 10.1 | Unlimited indemnity outflow | BLOCKING | Replace | GC |
| 14 | Missing audit right | MEDIUM | Insert | Counsel |

This is the data-heavy view. Per the practice profile's `## Outputs` rule, offer
the dashboard:

> 📊 **See this as a dashboard?** I'll build the interactive view with severity
> counts, an issue table, and a chart of where deviations cluster. Run
> `/vallor:contract-dashboard <this-redline-path>`.

### Step 6 — Save and offer next steps

Save to `~/.claude/plugins/config/vallor/redlines/<YYYY-MM-DD>-<counterparty>-<type>.md`.

Decision tree:

1. **Convert to .docx with tracked changes** — `/vallor:contract-redline-docx`
   produces the .docx counterparty-counsel will mark up.
2. **Build the approval deck** — for the deal-review committee.
3. **Draft the escalation note** — `/vallor:escalation-flagger` for the
   BLOCKING items.
4. **Draft the cover email** — short note to opposing counsel framing the redline.
5. **Stop here** — redline filed.

## What this skill does NOT do

- Doesn't produce a .docx with tracked changes. Use `contract-redline-docx`.
- Doesn't negotiate. The fallbacks are pre-positioned, not pre-conceded.
- Doesn't accept the counterparty's position silently. Match positions appear in
  the findings table marked `Match` so the reviewer can see what was read.

## Examples

```
/vallor:contract-redline acme-msa.pdf
```

```
/vallor:contract-redline acme-msa-v2.pdf --against prior-version --mode side-by-side
```
