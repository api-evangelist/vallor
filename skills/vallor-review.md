---
name: review
description: >
  Review a vendor agreement, NDA, or SaaS subscription against your playbook.
  Identifies the agreement structure from titles, routes to the right review skill
  (vendor-agreement-review, nda-review, saas-msa-review), and integrates the output
  into a single memo. Use when the user says "review this contract", "check this
  MSA", "is this NDA okay", "look at this SaaS agreement", or attaches an inbound
  agreement for review.
argument-hint: '[file path | Drive link | [CLM ID] | paste text]'
---

# /review

Reviews an inbound agreement against the playbook in `~/.claude/plugins/config/vallor/CLAUDE.md`. Identifies the agreement structure from titles, selects the appropriate skill(s), and — if confirm_routing is enabled — checks with the user before proceeding.

## Instructions

1. **Load `~/.claude/plugins/config/vallor/CLAUDE.md`.** If placeholders present, stop and prompt: "Run `/vallor:cold-start-interview` first — I need to learn your playbook before I can review against it."

   Also read `~/.claude/plugins/config/vallor/CLAUDE.md` → `## Review preferences` → `confirm_routing`. If the field is missing, treat it as `true`.

2. **Get the agreement:** From file path, Drive link, [CLM ID], or pasted text. If none provided, ask.

3. **Read the document structure — titles first.**

   Before reading the body, extract:
   - The main agreement title (e.g., "Master Services Agreement", "Non-Disclosure Agreement")
   - All exhibit, schedule, addendum, and attachment titles (e.g., "Exhibit A — Data Processing Addendum", "Schedule 1 — Subscription Order Form", "Annex B — Service Level Agreement")

   This is the routing signal. Do not rely on body keywords alone — a 40-page MSA with "confidential" throughout is not an NDA.

4. **Select the skill(s) based on document structure.**

   Map each identified document or section to a skill:

   | Document / section title contains | Skill |
   |---|---|
   | Non-Disclosure, NDA, Confidentiality Agreement (as the *main* agreement) | **nda-review** |
   | Master Services Agreement, Professional Services, Statement of Work, Consulting Agreement | **vendor-agreement-review** |
   | Subscription, SaaS, Cloud Services, Order Form with auto-renewal, Software License with recurring fees | **saas-msa-review** (overlay on vendor-agreement-review) |
   | Data Processing Addendum, DPA, Data Processing Agreement (as exhibit or standalone) | note for **vendor-agreement-review** → data protection section |
   | Service Level Agreement, SLA (as exhibit) | note for **saas-msa-review** → SLA section |

   Multiple skills may apply. Common combinations:
   - MSA + DPA exhibit → vendor-agreement-review, with DPA noted
   - SaaS subscription + Order Form + SLA exhibit → saas-msa-review (covers all three)
   - MSA + Order Form with auto-renewal → vendor-agreement-review + saas-msa-review overlay

   When the structure is genuinely ambiguous after reading titles (e.g., a document titled "Agreement" with no exhibits listed), read the first two pages of the body to resolve it — then stop and route.

5. **Confirm routing if enabled.**

   If `confirm_routing` is `true` in `~/.claude/plugins/config/vallor/CLAUDE.md` (or field is absent):

   ```
   I'm going to review this as: [agreement type(s)].

   Documents identified:
   - [Main agreement title] → [skill]
   - [Exhibit A title] → [how it will be handled]
   - [Exhibit B title] → [how it will be handled]

   Sound right? (yes / no — or tell me what I got wrong)
   ```

   Wait for confirmation before proceeding. If the user corrects the routing, apply their instruction and proceed.

   If `confirm_routing` is `false`: proceed silently. Log the routing decision at the top of the review memo so the user can see what was applied.

6. **Run the skill(s).** Follow each skill's workflow fully. If multiple skills apply, run them in sequence and integrate the output into a single memo — don't produce separate memos.

7. **Check for escalations:** If any issue exceeds the reviewer's authority per the `~/.claude/plugins/config/vallor/CLAUDE.md` matrix, invoke **escalation-flagger** to route and draft the ask.

8. **Offer follow-ups:**
   - Stakeholder summary for the business owner
   - Redline .docx with tracked changes
   - [CLM] record creation (if connected)
   - Add to renewal register (if auto-renewal found)

## Ask the user for context (optional but encouraged)

Use the `AskUserQuestion` tool to gather optional context the document itself does not reveal. The review is sharper when calibrated to the deal context.

**Up front (before deep review):**
Ask 1-3 short questions when not obvious from the document or practice profile:
- Side: sales (we're the vendor) / purchasing (we're the customer) — if not obvious from whose paper
- Deal value and term, if not in the doc, plus expected annualized spend
- Counterparty relationship: strategic / preferred / transactional / unknown
- Time pressure: same-week / standard / open-ended
- Hot-buttons or known concerns the user already wants emphasized (e.g., data residency, IP assignment, exit rights)
- Whether a Contract Brief or matter context already covers this (skip the questions if yes)

**Mid-flight (after first pass of the document):**
Once you've read the agreement and indexed deviations, surface concrete asks before guessing on:
- Issues that depend on facts not in the doc (e.g., data flow, sub-processor list, integration scope) — ask the user instead of guessing
- Which 3-5 issues to elevate to the executive summary
- Whether to redline now or stop at the review memo
- Whether to escalate items that sit at the threshold rather than over it

**Rules:**
- Every question must offer a reasonable default the user can pick to skip ahead.
- Ask follow-ups when answers reveal more depth or a non-default choice.
- Cap a single round at ~3 questions.
- Skip when the user has signaled a fast / default run.

## Configuring confirm_routing

Add to `~/.claude/plugins/config/vallor/CLAUDE.md` → `## Review preferences`:

```markdown
## Review preferences

confirm_routing: true   # Set to false to skip routing confirmation and proceed automatically
```

The cold-start interview should ask about this preference. Default is `true` — confirmation on. As trust builds, the user can set it to `false`.

## Examples

```
/vallor:review vendor-msa.pdf
```

```
/vallor:review https://drive.google.com/file/d/ABC123
```

```
/vallor:review
[paste agreement text]
```

## Output

Full review memo per the skill's format. Routing decision logged at the top. Deviation-by-deviation, specific redline language, named approver. Saved where `~/.claude/plugins/config/vallor/CLAUDE.md` → House style says work product goes.
