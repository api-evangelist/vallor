---
name: contract-redline-docx
description: >
  Produces a Word .docx with tracked changes (Word-native revision marks) of a
  contract — the file you actually send to opposing counsel. Wraps the output of
  /vallor:contract-redline (or builds the redline from scratch if no
  prior run exists), invokes python-docx + tracked-changes XML, and emits a
  reviewer-ready .docx with a cover memo. Use when the user says "redline in word",
  "send to opposing counsel", "tracked changes docx", "build the docx redline",
  "put this in Word", "make it a redline file", or after contract-redline has
  produced markdown that needs to ship.
argument-hint: "[contract-path-or-redline-md] [--reviewer-name <name>] [--mode external|internal]"
---

# /vallor:contract-redline-docx

## Purpose

Markdown redlines are great for diffing and review; opposing counsel wants a Word
file with revision marks. This skill produces that file — with Word's native
tracked-changes XML, the right reviewer name, the right author metadata, and a
cover memo at the top — so it lands the way a partner would have sent it.

## Inputs

Two paths, depending on prior state:

1. **Path A — redline already exists.** User points at a markdown redline produced
   by `/vallor:contract-redline` (or one written by hand). This skill
   converts the proposed-language changes to tracked Word revisions against the
   original source document. The original must be available (path passed in or
   located in the matter folder).

2. **Path B — fresh.** User points at a contract PDF/DOCX with no prior redline.
   This skill runs the redline workflow internally (same logic as
   `contract-redline`), then converts.

Path A is preferred — it lets the user review the redline in markdown first, then
ship. The skill prompts for Path B only when no markdown redline is found.

## Load context first

1. Read `~/.claude/plugins/config/vallor/CLAUDE.md`. Bounce on
   placeholders.
2. Determine the side (sales / purchasing).
3. Apply the **/design** skill — the .docx uses Times New Roman 11pt body, the
   plugin's footer, and the work-product header (or jurisdiction-correct equivalent).
4. Read the source contract (PDF or DOCX). For PDF inputs, extract text with
   layout preserved; convert to DOCX first using `pandoc` or `python-docx`.

## Ask the user for context (optional but encouraged)

Use the `AskUserQuestion` tool to calibrate the .docx before generating. The file ships to opposing counsel — a wrong author name or mode is embarrassing.

**Up front (before generating the docx):**
Ask 1-3 short questions when not obvious from the inputs or practice profile:
- Mode: external (for opposing counsel — strips internal markings) / internal (keeps `[review]` and reviewer notes)
- Reviewer attribution: which name/email to embed in tracked-changes XML (default: practice-profile reviewer)
- Cover memo tone: tight ("we propose the attached changes") / collaborative ("happy to discuss any of these") / firm ("the marked items are required")
- Whether to include explanatory comments inline or strip them to a separate redline cover

**Mid-flight (after applying the markdown redline):**
After applying changes to the docx, surface concrete asks before guessing on:
- Items the markdown flagged `[review]` — drop, soft-comment, or escalate before sending
- Edits that landed in oddly-formatted source paragraphs (tables, footnotes) — confirm before mangling
- Whether to suppress trivial whitespace / numbering corrections to keep the tracked-changes list focused

**Rules:**
- Every question must offer a reasonable default the user can pick to skip ahead.
- Ask follow-ups when answers reveal more depth or a non-default choice.
- Cap a single round at ~3 questions.
- Skip when the user has signaled a fast / default run.

## Workflow

### Step 1 — Confirm reviewer identity and mode

The tracked-changes XML embeds the author of each revision. Confirm before writing:

> "I'm going to write tracked changes attributed to **[reviewer name from practice
> profile]** with email **[email]**. Mode is **[external = for opposing counsel /
> internal = for in-house review]**. The external mode strips the work-product
> header and any `[review]` flags. Internal mode keeps them. Confirm?"

Wait for confirmation. If the user wants a different reviewer name (e.g., a senior
partner whose name should appear on the file), apply.

### Step 2 — Build the change set

Read the source DOCX and the redline markdown. For each redline entry:

| Markdown action | DOCX action |
|---|---|
| `~~text~~` (strikethrough) | `<w:del>` revision, attributed to reviewer, dated today |
| `**bold inserted text**` block following the strike | `<w:ins>` revision, same author/date |
| `> WHY: …` annotation | `<w:comment>` on the affected span, attributed to reviewer |
| `> FALLBACK: …` | `<w:comment>` with prefix `[Fallback]` so opposing counsel sees it as a position, not a demand |
| `● HIGH` / `● BLOCKING` severity markers | Stripped in external mode; preserved as `[INTERNAL: severity]` comments in internal mode |

Use `python-docx` for the file itself and direct OOXML manipulation for the tracked
changes (python-docx alone doesn't write `<w:ins>` / `<w:del>` natively). The
reference implementation lives at `references/docx-tracked-changes.md`.

### Step 3 — Assemble the cover memo

A short cover memo precedes the file in the message but is NOT embedded in the
file. Format:

```
Hi [opposing counsel name],

Attached please find our redline to the [agreement type]. The key positions are
summarized below; happy to discuss any of them.

  • [Issue 1 in one sentence — neutral business framing]
  • [Issue 2]
  • [Issue 3]

The substantive changes are in tracked changes throughout. Comments mark spots
where I've offered fallback language we'd accept.

[Reviewer name]
[Firm / company]
```

The cover memo is the first thing the user copies into email. It is plain text,
no markdown ornamentation. It is generated separately so the user can edit it
before sending.

### Step 4 — Write the .docx

Output path:
`~/.claude/plugins/config/vallor/redlines/<YYYY-MM-DD>-<counterparty>-<type>-redline.docx`

(Or matter folder if workspaces are on.)

Validation before writing:
- All redline entries map to a span in the source. If a span can't be located,
  surface it: "I can't find this language in the source — line 412 of the
  redline. Was the original clause renumbered between drafts?" Don't silently
  drop the change.
- Word can open the file (test by re-reading it with python-docx after writing).
- The `<w:settings>` block has `<w:trackChanges/>` set so changes appear as
  revisions, not committed edits.

### Step 5 — Deliver

> **Redline.docx is ready.** Saved to `[path]`.
>
> **Author:** [Reviewer]
> **Mode:** [external/internal]
> **Revisions:** [N insertions, M deletions, K comments]
>
> The cover memo (above) is plain text — copy it into your email when you send
> the file.

Decision tree:

1. **Send via DocuSign / outbound email** — if the user wants to ship now.
2. **Apply revisions to the .docx and produce a clean v2** — for internal sign-off
   before shipping (the v2 reflects all changes accepted; the redline.docx is the
   tracked-change version).
3. **Build the approval deck** — `/vallor:contract-approval-deck`.
4. **Stop here** — file is saved.

## Constraints

- The skill never sends the file. The user sends. The skill produces.
- It never modifies the source contract — always writes a new file.
- It never invents language. Every insertion must trace back to a position in the
  practice profile, the contract brief, or a position the user explicitly stated.

## External mode rules

When `--mode external`:
- Strip the work-product header from the .docx itself (it stays on internal
  copies and on the cover memo template, but never on the file going to opposing
  counsel).
- Strip all `[review]` and `[BLOCKING]` severity markers from comments — these
  are internal-only. Reframe the underlying position in neutral business terms.
- Strip any internal escalation references ("GC approval required") — these are
  internal-only.
- Keep substantive comments that explain the proposed position (e.g., "Carving
  IP indemnification above the cap is standard practice for SaaS at this scale").

## Internal mode rules

When `--mode internal`:
- Keep the work-product header at the top of the document.
- Keep severity markers in comments (`[● HIGH] Cap base on Never-accept list`).
- Keep escalation references.
- Add a footer to every page: `INTERNAL DRAFT — NOT FOR EXTERNAL DISTRIBUTION`.

## Examples

```
/vallor:contract-redline-docx redlines/acme-msa.md
```

```
/vallor:contract-redline-docx acme-msa.pdf --mode external
```

```
/vallor:contract-redline-docx --reviewer-name "Jane Smith, General Counsel" --mode external
```
