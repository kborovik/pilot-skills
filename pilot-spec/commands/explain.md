---
description: Math-glyph → prose. Expand any SPEC.md citation into plain English. Read-only.
argument-hint: [§T.n | §V.n | §B.n | §I.<key> | §G | §C | --next]
---

# /sdd:explain — decompress spec into prose

Inverse of the `glyph` skill (math-glyph encoder). Human-facing. Reads SPEC.md, expands one citation into plain English with cited context. Writes nothing.

## AUDIENCE

SPEC.md = LLM-facing artifact — ∀ reads/writes via Claude. humans operate through /sdd:* cmds, ⊥ hand-edit. /sdd:explain decodes glyph → prose ∀ human consumption. ∴ compression aggression (math-glyph, fragments, pipe tables) = feature ⊥ bug.

## LOAD

1. Read `SPEC.md`. If missing → "no spec, nothing to explain." Stop.
2. Parse `$ARGUMENTS`:
   - `§T.n` / `§V.n` / `§B.n` / `§I.<key>` → that row
   - `§G` / `§C` → that section in full
   - `--next` or empty → lowest-numbered §T row with status `.`
3. If citation absent → list valid ids in target section. Stop.

## EXPAND

For the chosen citation:

1. Quote the raw math-glyph line(s) verbatim in a code block.
2. Restate in plain English. No math-glyphs, no fragments — full sentences.
3. Pull in cited siblings:
   - §T row → expand every §V and §I it cites.
   - §V row → list §T tasks that cite it and §B bugs that reference it.
   - §B row → expand the §V it broke and the fixing §T.
   - §I row → name §V invariants that constrain it.
   - §G / §C → no cross-cites; just prose.
4. Close with one line: what the reader should now understand.

## OUTPUT SHAPE

```
## §T.3 — add auth middleware

> T3|.|add auth mw|V1,I.api

In plain English: this task adds an authentication middleware that runs before
every request reaches its handler.

Cited invariants:
- §V.1 — every request must pass an auth check before the handler runs.

Cited interfaces:
- §I.api — POST /x returns 200 with {id:string}; the middleware must not
  change this shape.

Status: not started (`.`).

Bottom line: implement a middleware that enforces §V.1 without altering §I.api.

## Hint

§T.3 is pending — typical next step is item 1 to start work, or item 2 if you want to read the cited invariant first.

## Next

1. /sdd:build §T.3 — start implementation
2. /sdd:explain §V.1 — read the cited invariant in prose
```

## OUTPUT — "Next" block

Every `/sdd:explain` response terminates with a `## Next` block, optionally preceded by a `## Hint` block. Shape (positional dispatch):

- Heading exactly `## Next`.
- Ordered numbered list, 1–5 items.
- Each item = atomic action description (one sentence ∨ phrase). ⊥ `Reply <token>` prefix, ⊥ leading dispatch label (`Reply`, `Run`). Dispatch: user types `run <integer>` → execute item at index; cross-skill jumps via `run /<plugin>:<cmd> [args]` (discriminator: 1st arg digit vs `/`).
- **Actionable only.** Each item ! describe a real state transition. `/sdd:explain` is read-only ∧ holds no wait-state — items are slash-cmd follow-ups (`/sdd:build`, `/sdd:check`, another `/sdd:explain`). N=1 is legal when only one follow-up is sensible.
- No raw file:line edit refs. No §-ref imperatives. No compound items. No prose mid-list. No next-step directives outside the block.
- Items must be applicable to current state. Suggest `/sdd:build §T.n` only when the cited row's status is `.`. Closed rows (status `x`) → suggest `/sdd:explain --next` to read the next pending row, or `/sdd:check --all` to audit.

### Hint block (optional)

- Heading exactly `## Hint`. Precedes `## Next`.
- Free-form prose, ≤ 3 lines. Educational context (when to use which citation form, gotcha on closed-vs-pending rows) or recommendation among Next items ("if you don't recall what's pending, item 1 jumps to the next open §T").
- ⊥ contain new directives — directives live in Next.
- ⊥ emit when nothing useful to add. Empty Hint = ceremonial filler.

Example for a closed §T row (terminal state):

```
## §T.3 — add auth middleware

> T3|x|add auth mw|V1,I.api

In plain English: this task added an authentication middleware...

Status: complete (`x`).

Bottom line: §V.1 is enforced by the middleware shipped under §T.3.

## Hint

Closed rows are historical. `run 1` skips to live work; `run 2` audits whether the closed task drifted out of code.

## Next

1. /sdd:explain --next — read the next pending §T row
2. /sdd:check --all — audit whether the closed task still holds
```

The "Bottom line" sentence stays — it summarizes the citation, it does not direct action. Action lives only in the Next block; pre-action context lives in the optional Hint block. Heading text `## Next` and `## Hint` are plain markdown and do not violate the prose-only-output rule: math-glyphs ⊥ allowed in body, but plain markdown headings are not math-glyphs.

## NON-GOALS

- Zero writes. No SPEC.md edits. No code edits.
- No code reads. Spec-only. (Use `/sdd:check` if you want spec-vs-code.)
- No math-glyphs in output. Prose is the whole point.
- No multi-citation expansion in one call. One id per invocation; loop if needed.
