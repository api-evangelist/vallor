---
name: sourcing-strategy
description: >
  Per-event sourcing strategy: incumbent vs competitive bidding, bid-floor
  decision, evaluation weights, sourcing channel (RFI → RFP, direct RFQ,
  negotiation-only), timeline, governance, communications plan. Translates a
  category-strategy playbook into the actual event plan. Use when the user says
  "sourcing strategy", "sourcing plan for [event]", "should we run an RFP",
  "incumbent vs competitive", "set up the sourcing event", or before launching
  any procurement event that benefits from a written approach.
argument-hint: "<category-or-event-name> [--mode rfi-rfp|direct-rfq|negotiation-only|reverse-auction]"
---

# /vallor:sourcing-strategy

## Purpose

The category strategy says "we should resource this every 3 years using a
competitive event." The sourcing strategy says "for THIS event, here's exactly
how we'll run it, who's invited, what the scoring is, when each milestone hits,
and what we tell the incumbent."

## Load context

1. Read `~/.claude/plugins/config/vallor/CLAUDE.md`:
   - `## Categories under management` — does this category have a strategy?
   - `## Sourcing approach` — default sourcing event, bid floor, RFQ template
     format, scoring weights, TCO drivers
   - `## Vendor / supplier classification`
2. If a category strategy exists, load it. The sourcing strategy is downstream
   of (and consistent with) the playbook.
3. Apply the **/vallor:design** skill.

## Ask the user for context (optional but encouraged)

Use the `AskUserQuestion` tool to calibrate the strategy to the specific event. Sourcing strategy decisions are reversible only at high cost (notifying suppliers, restarting timelines).

**Up front (before drafting the memo):**
Ask 1-3 short questions when not obvious from the category strategy:
- Mode (if not via `--mode`): RFI→RFP / direct RFQ / negotiation-only / reverse auction
- Incumbent strength: weak / par / strong — drives whether to invite incumbent and how
- Savings target band: 0-5% / 5-15% / 15-25% / aggressive 25%+
- Timeline pressure: standard 8-12 weeks / accelerated 4-6 weeks / urgent <4 weeks
- Political constraints: known stakeholder vetoes, suppliers we can't invite or exclude
- Confidentiality: announce the event publicly, invitation-only, or non-public

**Mid-flight (after picking mode and bidder list):**
After laying out the mode and bid floor, surface concrete asks before guessing on:
- Whether to use sealed-bid or English/Dutch auction (if a competitive event)
- Communication plan with the incumbent (early heads-up vs simultaneous invite)
- Pricing structure to require (unit-price / total-cost / index-linked / OPEX vs CAPEX)
- Whether to require a sustainability or DEI sub-score

**Rules:**
- Every question must offer a reasonable default the user can pick to skip ahead.
- Ask follow-ups when answers reveal more depth or a non-default choice.
- Cap a single round at ~3 questions.
- Skip when the user has signaled a fast / default run.

## Output — the Sourcing Strategy Memo

```markdown
# Sourcing Strategy — [Category / Event]

**Owner:** [strategy owner from category table]
**Sponsor:** [exec sponsor]
**Date:** [today]
**Category:** [category]
**Annual spend:** [from category profile]
**Kraljic quadrant:** [strategic / leverage / bottleneck / non-critical]

---

## Why we're running this event

[Two sentences — the strategic context. Pulled from the category strategy's
"Why this matters now" if one exists.]

## Sourcing mode

**Mode:** [RFI → RFP / Direct RFQ / Negotiation only / Reverse auction]

**Rationale:**
- [Why this mode, not the alternatives.]
- [Reference: market structure, supplier base, timeline, dollars at stake.]

## Incumbent treatment

**Incumbent:** [name, current spend, current contract end]
**Approach:**
- [ ] Invite to bid (competitive, normal terms)
- [ ] Invite to bid (with a heads-up that we're testing the market)
- [ ] Negotiate first; bid if we don't get target outcome
- [ ] Replace (don't invite)

**Rationale:** [Why this approach. Often this is the single most
politically-loaded decision in the event.]

**Notice / communication:** [What we'll tell the incumbent and when.]

## Bid floor — who's invited

| Supplier | Tier | Why invited | Disqualifier (if any) |
|---|---|---|---|

Minimum suppliers: **[N]** (from practice profile bid floor; default 3)

## Scoring weights (event-specific overrides)

| Dimension | Practice default | This event | Why differ |
|---|---|---|---|
| Capability | 30% | [X%] | [reason if different] |
| Technical / Quality | 25% | ... | ... |
| Supply chain / Logistics | 20% | ... | ... |
| TCO | 15% | ... | ... |
| Partnership | 10% | ... | ... |

If weights differ materially from the practice default, surface the why — a
weight change reflects a strategy decision, not a typo.

## Total cost of ownership model

TCO drivers we'll quantify in scoring:
- [ ] Unit price
- [ ] Logistics + freight
- [ ] Import duty / tariff
- [ ] Quality cost (PPM × cost per defect)
- [ ] Payment-term value
- [ ] Currency exposure
- [ ] Switching cost (if replacing incumbent)
- [ ] Tooling / qualification investment

Pulled from `## Sourcing approach → TCO drivers` in the practice profile.

## Timeline

| Milestone | Owner | Date |
|---|---|---|
| Kickoff with stakeholders | [name] | [date] |
| Issue RFP / RFQ | [name] | [date] |
| Q&A window closes | [name] | [date] |
| Responses due | [name] | [date] |
| Scoring complete | [name] | [date] |
| Shortlist + supplier presentations | [name] | [date] |
| BAFO request | [name] | [date] |
| BAFO due | [name] | [date] |
| Recommendation to approvers | [name] | [date] |
| Award + notification | [name] | [date] |
| Contract execution | [name] | [date] |

## Governance

- **Decision team:** [names + roles]
- **Approval authority for award:** [per practice profile walk-away matrix]
- **Conflicts of interest:** [any team member relationships disclosed]
- **Cone of silence:** dates the team is non-communicative with bidders

## Communication plan

| Audience | Channel | When | Message owner |
|---|---|---|---|
| Internal stakeholders | [Slack / email] | [date] | [owner] |
| Bidders | [Email / portal] | [milestone] | [owner] |
| Incumbent (if separate from bidders) | [...] | [...] | [...] |
| Board / exec staff | [Quarterly readout] | [...] | [...] |

## Risk register

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| [Incumbent finds out and price-protects] | [...] | [...] | [...] |
| [No qualified bidders respond] | [...] | [...] | [...] |
| [Supplier financial distress mid-event] | [...] | [...] | [...] |
| [Specification ambiguity drives unscoreable responses] | [...] | [...] | [...] |

## Success criteria

The event succeeds if:
- [ ] [Target unit price] achieved
- [ ] [Target TCO] achieved
- [ ] [Term] locked in
- [ ] [Payment terms] improved
- [ ] At least [N] viable suppliers identified for future events
- [ ] [Other event-specific success markers]

## Approval

| Approver | Role | Signed | Date |
|---|---|---|---|
| [name] | Category owner | | |
| [name] | Finance | | |
| [name] | Business sponsor | | |
| [name] | CPO | | |

---
```

## Decision tree after delivery

1. **Build the RFQ workbook** — `/vallor:rfq-creation` matches the
   event mode.
2. **Open the negotiation prep** — `/vallor:7-step-negotiation`
   begins Step 1 (prepare) if Mode is "Negotiation only".
3. **Notify the incumbent** — draft the communication per the plan above.
4. **Stop here** — strategy is saved and signed off.

## Examples

```
/vallor:sourcing-strategy "Cloud infrastructure — FY26"
```

```
/vallor:sourcing-strategy "Steel coils — Mexico plant" --mode reverse-auction
```
