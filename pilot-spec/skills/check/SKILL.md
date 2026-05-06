---
name: check
description: |
  Read-only drift detector. Diffs SPEC.md vs current code ∧ reports violations
  grouped by severity. ⊥ writes — suggests remedies via spec ∨ build skills,
  ⊥ invokes them. Triggers when user asks to check drift, audit spec, verify
  invariants, ∨ ask if code still matches spec. Phrasings: "check drift",
  "audit the spec", "does the code still match §V", "check invariants",
  "spec vs code", "is the spec still accurate?", "did the code drift?".
---

# check — drift report

Pure diagnostic. Reports violations. Writes nothing. User decides remedy.

## AUDIENCE

SPEC.md = LLM-facing artifact — ∀ reads/writes via Claude. humans operate through /sdd:* cmds, ⊥ hand-edit. /sdd:explain decodes glyph → prose ∀ human consumption. ∴ compression aggression (math-glyph, fragments, pipe tables) = feature ⊥ bug.

## LOAD

1. Read `SPEC.md`. If missing → "no spec, nothing to check." Stop.
2. Parse invocation args:
   - `§V` → check invariants only (default)
   - `§I` → check interfaces
   - `§T` → audit task status vs code
   - `--all` → all three

## CHECK §V — invariants

For each V<n>:

1. Translate invariant into verifiable claim about code.
2. Grep / read relevant files.
3. Classify: **HOLD** / **VIOLATE** / **UNVERIFIABLE**.
4. Record address + file:line evidence.

Recipes ! parametric — ⊥ name repo-literal paths beyond `SPEC.md`. Project-specific enforcement ∈ `.claude/commands/`.

## CHECK §I — interfaces

For each I item:

1. Locate implementation.
2. Classify:
   - **MATCH** — shape in code = shape in spec.
   - **DRIFT** — impl exists, shape differs.
   - **MISSING** — impl absent.
   - **EXTRA** — code exposes surface not in §I.

## CHECK §T — tasks

For each T<n>:

1. If `x`: verify claimed work present.
2. If `~`: note as in-progress.
3. If `.`: note as pending.
4. Flag `x` rows with no evidence as **STALE**.

## REPORT

Math-glyph. Grouped by severity.

```
## §V drift
V2 VIOLATE: auth/mw.go:47 uses `<` not `≤`. see §B.1.
V5 UNVERIFIABLE: no test covers ∀ req path.

## §I drift
I.api DRIFT: POST /x returns `{result}` not `{id}`. route.go:112.
I.cmd MISSING: `foo bar` absent from cli/*.go.

## §T drift
T3 STALE: status `x`, no middleware file exists.

## summary
2 violate. 1 missing. 1 stale. 1 unverifiable.
```

## REMEDY HINTS (used to populate the Next block, not as a separate output section)

Map each drift class to a candidate Next item — pick the most acute classes and surface them as numbered descriptions (no leading dispatch label):

- VIOLATE / DRIFT → `/sdd:spec bug: <description citing V.n>`.
- MISSING → `/sdd:build §T.n` if task exists; else `/sdd:spec amend §T` to add row.
- STALE → `/sdd:spec amend §T.n` to uncheck status.
- EXTRA → `/sdd:spec amend §I` to document, or `/sdd:spec bug: <surface> ⊥ ∈ §I` to record drift.

Never invoke fixes. Report only.

## OUTPUT — "Next" block

Every check response terminates with a `## Next` block, optionally preceded by a `## Hint` block. Shape (positional dispatch):

- Heading exactly `## Next`.
- Ordered numbered list, 1–5 items.
- Each item = atomic action description (one sentence ∨ phrase). ⊥ `Reply <token>` prefix, ⊥ leading dispatch label. Dispatch: user types `run <integer>` → execute item at index; cross-skill jumps via `run /<plugin>:<cmd> [args]`.
- **Actionable only.** Each item ! describe a real state transition. check is read-only ∧ holds no wait-state — items are slash-cmd follow-ups (`/sdd:spec bug: ...`, `/sdd:spec amend ...`, `/sdd:build §T.n`, `/sdd:check --all`). N=1 is legal when only one follow-up is sensible.
- No raw file:line edit refs. No §-ref imperatives. No compound items. No prose mid-list. No next-step directives outside the block (REMEDY HINTS table above is internal scaffolding).
- Items must be applicable to current state. Before suggesting `/sdd:build --next`, confirm ≥ 1 §T row has status `.` or `~`. If §T fully closed (all `x`) and audit is clean, suggest state-seeding cmds (`/sdd:spec` to add a row, `/sdd:spec bug: ...` to backprop) instead.

### Hint block (optional)

- Heading exactly `## Hint`. Precedes `## Next`.
- Free-form prose, ≤ 3 lines.
- **Default: skip Hint.** Emit only when context ⊥ derivable from `## Next` item descriptions alone:
  - (a) educational — e.g. drift-severity ordering (VIOLATE > DRIFT > MISSING > STALE > EXTRA), or which class warrants `/sdd:spec bug:` vs `amend`.
  - (b) recommends among items via hidden state.
  - (c) warns about side-effect ∨ precondition.
- ⊥ paraphrase Next items, ⊥ restate item count, ⊥ describe what `run 1` literally does.
- ⊥ contain new directives — directives live in Next.

Example after drift found:

```
## summary
1 violate. 1 drift.

## Hint

VIOLATE outranks DRIFT — record the V2 breach via item 1 before fixing the interface drift, so §B captures the cause not just the symptom.

## Next

1. /sdd:spec bug: V2 violation at auth/mw.go — record the drift
2. /sdd:spec bug: I.api DRIFT at route.go — record interface drift
```

Example after clean run with pending §T rows:

```
## summary
0 drift. spec ∧ code aligned.

## Next

1. /sdd:build --next — pick up the next pending §T row
2. /sdd:check --all later — re-audit interfaces and tasks
```

Example after clean run with §T fully closed (terminal state):

```
## summary
0 drift. spec ∧ code aligned. §T fully closed.

## Next

1. /sdd:spec — add a new §T row before building
2. /sdd:check --all later — re-audit
```

## NON-GOALS

- Zero writes. No SPEC.md edits. No code edits.
- No sub-agents. Main thread reads.
- No scores, no grades. Binary per item: holds or drifts.
