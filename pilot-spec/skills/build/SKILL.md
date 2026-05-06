---
name: build
description: |
  Plan-then-execute impl vs SPEC.md. Native single-thread loop, ⊥ sub-agents.
  On test ∨ build fail → auto-invokes backprop skill before retry; failed
  verification always considers if new §V invariant prevents recurrence.
  Triggers when user asks to build, implement, execute spec, ∨ tackle
  specific §T task. Phrasings: "build §T.<n>", "build --next", "implement
  next task", "run the build", "does the implementation run?", "is §T.<n>
  done?". Expects SPEC.md → exists; ⊥ → defers to spec skill.
---

# build — implement spec

Single-thread native plan→execute. You are main Claude. No swarm.

## AUDIENCE

SPEC.md = LLM-facing artifact — ∀ reads/writes via Claude. humans operate through /sdd:* cmds, ⊥ hand-edit. /sdd:explain decodes glyph → prose ∀ human consumption. ∴ compression aggression (math-glyph, fragments, pipe tables) = feature ⊥ bug.

## LOAD

1. Read `SPEC.md`. If missing → tell user to invoke the spec skill first. Stop.
2. Parse invocation args:
   - `§T.n` → that task only
   - `--next` → lowest-numbered row with status `.`
   - `--all` or empty → every `.` row in §T order
   - `--fast` → opt-in fast-path (skip PLAN gate when scope ≤ threshold ∧ wording mechanical); ignored under `--all`
   - `--plan` → force full PLAN-then-wait even when fast-path would auto-fire

## FAST-PATH (single-task only)

For `§T.n` ∨ `--next` (⊥ `--all`), check fast-path eligibility:

- Scope: ≤ 1 file edited ∧ ≤ 5 lines changed ∧ ⊥ §I touched ∧ ⊥ new §V cited beyond mechanical re-cites.
- Wording matches mechanical pattern: row task text leads w/ verb stem `swap` ∨ `strip` ∨ `drop` ∨ `sweep` and names literal targets.

Eligible ∧ user passed `--fast` ∨ no `--plan` flag → merge PLAN ∧ EXECUTE into one turn:

1. Announce inline plan (1–3 lines, cite §V, file, edit summary).
2. Edit. Verify. Auto-commit.
3. Emit Next block.

Ineligible ∨ `--plan` flag → run normal PLAN-then-wait flow below.

`--all` always runs normal PLAN gate per fast-path carve-out — batch decisions earn confirmation.

## PLAN

Native plan mode. For chosen task(s):

1. Cite every §V invariant that applies. Plan must respect all.
2. Cite every §I interface touched. Plan must preserve shape.
3. List files to create / edit.
4. List tests to add or update (one per invariant touched).
5. Name verification command (test, build, lint).

Show plan. Wait for user OK unless auto mode.

## EXECUTE

Per task in order (status flips `.` → `x` direct):

1. Edit code per plan.
2. Run verification command.
3. **Pass** → flip §T.n status `.` → `x`; auto-commit. Next task.
4. **Fail** → invoke backprop skill. Do NOT retry blindly. Status stays `.`.

## FAIL → BACKPROP

On test/build failure:

1. Read failure output.
2. Ask: is failure (a) my code bug, (b) spec wrong, or (c) unspecified edge case?
3. If (a) → fix code, re-run. No spec change.
4. If (b) or (c) → invoke spec skill with `bug: <cause>` first, let it update §V and §B, then resume build against updated spec.

Rule: never silently fix root-cause without considering backprop. §B is the memory that stops recurrence.

## WRITE POLICY

- Only flip §T status. No other SPEC.md edits from build.
- Other spec edits → invoke spec skill.
- Auto-commit on `.` → `x` per §T row; ⊥ user prompt. Message: `T<n>: <goal line>` + §V cites.
- Stage explicit `git add <listed-paths>` per plan; ⊥ `git add -A` — pre-existing dirty working-tree state ⊥ bundled.
- `/sdd:build --all` chains plan-once → ∀ §T row {edit → verify → commit} autonomously.
- FAIL → ⊥ commit; FAIL→BACKPROP runs first.

## VERIFICATION

Task `x` only if:
- Verification command exits 0.
- New test(s) added per plan.
- No §V invariant regressed (run full test suite at end).

## OUTPUT — "Next" block

Every build response terminates with a `## Next` block, optionally preceded by a `## Hint` block. Mirrors `commands/build.md`. Shape (positional dispatch):

- Heading exactly `## Next`.
- Ordered numbered list, 1–5 items.
- Each item = atomic action description (one sentence ∨ phrase). ⊥ `Reply <token>` prefix, ⊥ leading dispatch label. Dispatch: user types `run <integer>` → execute item at index; cross-skill jumps via `run /<plugin>:<cmd> [args]`.
- **Actionable only.** Each item ! describe a real state transition. PLAN waits on user → execute / revise / abort bind to items by position. EXECUTE-pass auto-commits → ⊥ commit item; Next leads w/ /sdd:check ∨ /sdd:build --next ∨ /sdd:spec follow-ups. After backlog cleared, omit ceremonial items — suggest `/sdd:spec` to seed instead.
- No raw file:line edit refs. No §-ref imperatives. No compound items. No prose mid-list. No next-step directives outside the block.
- Items must be applicable to current state. After EXECUTE pass, before suggesting `/sdd:build --next`, confirm ≥ 1 remaining §T row with status `.`. If backlog cleared, suggest `/sdd:spec` (seed) instead.

### Hint block (optional)

- Heading exactly `## Hint`. Precedes `## Next`.
- Free-form prose, ≤ 3 lines.
- **Default: skip Hint.** Emit only when context ⊥ derivable from `## Next` item descriptions alone:
  - (a) educational — e.g. "FAIL→backprop is auto, ⊥ optional" (system mechanic users may ⊥ know).
  - (b) recommends among items via hidden state.
  - (c) warns about side-effect ∨ precondition.
- ⊥ paraphrase Next items, ⊥ restate item count, ⊥ describe what `run 1` literally does.
- ⊥ contain new directives — directives live in Next.

Example after PLAN (awaiting `run 1` to execute; Hint skipped — "execute / revise / abort" is self-explanatory):

```
## Next

1. execute the plan as shown
2. revise scope (e.g. `run 2 files`, `run 2 tests`)
3. abort
```

Example after EXECUTE pass with pending §T rows (commit already auto-fired):

```
## Next

1. /sdd:build --next — start the next pending §T row
2. /sdd:check — verify the new invariant holds across the codebase
3. /sdd:spec amend §T.<n> — re-scope before continuing
```

Example after EXECUTE pass with backlog cleared (terminal state):

```
## Next

1. /sdd:check — verify the new invariant holds across the codebase
2. /sdd:spec — add the next §T row before building again
```

## NON-GOALS

- No sub-agents. No parallel workers. Main thread only.
- No progress dashboards. `cat SPEC.md | grep §T` is the dashboard.
- No speculative work beyond chosen task scope.
