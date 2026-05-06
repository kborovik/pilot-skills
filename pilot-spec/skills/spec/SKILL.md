---
name: spec
description: |
  Create, amend, ∨ backprop bugs into SPEC.md @ repo root. Sole mutator of
  project spec. Always-gated by `core:socratic` skill — concrete intent
  passes ≤ 1 turn, vague intent triggers dialogue until convergence; mode
  (NEW ∨ DISTILL ∨ BACKPROP ∨ AMEND) emitted by gate post-convergence,
  ⊥ user-typed prefix dispatch. Triggers when user asks to write spec,
  start new spec, distill spec from existing code, add invariants, amend
  a section, ∨ record a bug. Common phrasings: "write the spec for...",
  "new spec", "distill spec from code", "spec this idea", "import existing
  repo", "spec from this codebase", "pull invariants out of code", "this
  bug keeps biting", "post-mortem on Y". Follows glyph skill for math-glyph
  encoding ∧ pipe-table shape of §T ∧ §B.
---

# spec — spec mutator

The `glyph` skill (math-glyph encoder) applies to all writes here.

## AUDIENCE

SPEC.md = LLM-facing artifact — ∀ reads/writes via Claude. humans operate through /sdd:* cmds, ⊥ hand-edit. /sdd:explain decodes glyph → prose ∀ human consumption. ∴ compression aggression (math-glyph, fragments, pipe tables) = feature ⊥ bug.

## DISPATCH

**Step 0 (precondition):** `git diff --quiet SPEC.md` ⊢ continue; ⊥ → bail w/ "SPEC.md has uncommitted changes; commit ∨ stash first" (auto-commit on apply assumes clean baseline).

Engage `core:socratic` gate w/ user input as intent. Gate runs single-question loop until convergence triple matches one mode:

- **NEW** ≡ goal ∧ (≥ 1 invariant ∨ ≥ 1 task)
- **DISTILL** ≡ explicit "build from code" intent (gate exits ≤ 1 turn — walks repo, no further interrogation)
- **BACKPROP** ≡ symptom ∧ surface ∧ recurrence-class
- **AMEND** ≡ §-target ∧ delta

Two paths (SPEC.md presence is the only branch — mode is gate byproduct, ⊥ user-typed prefix):

1. **⊥ `SPEC.md` @ repo root** → gate restricted to {NEW, DISTILL}; post-convergence → run mode-specific procedure below.
2. **`SPEC.md` exists** → gate ranges over {BACKPROP, AMEND, NEW}; post-convergence → run mode-specific procedure. NEW available but rare → require explicit re-init confirmation before overwrite.

Concrete first-turn input → gate passes ≤ 1 turn (zero-friction); vague → dialogue continues until convergence. ⊥ skip flag, ⊥ prefix back-doors.

## NEW — idea → spec

Input: user idea.

Steps:
1. Extract goal (1 line, math-glyph). → §G.
2. List constraints user stated or implied. → §C.
3. List external surfaces user named. → §I.
4. Propose initial invariants. → §V (numbered V<n>).
5. Break goal into ordered tasks. → §T pipe table, all status `.`, ids T<n>.
6. §B section with header row only (`id|date|cause|fix`).

Show user full file. On user OK → write `SPEC.md` ∧ auto-commit `git add SPEC.md` w/ message `init SPEC.md (V1..V<n>, T1..T<m>)`; ⊥ user prompt for commit step.

## DISTILL — code → spec

Walk repo. Produce §G (infer from README/package.json/main entry), §C (infer from stack), §I (enumerate public APIs/CLIs/configs), §V (derive from tests and assertions), §T (one task per known TODO or missing test), §B (empty).

Math-glyph everywhere. Flag uncertain items with `?` in text so user can confirm. On user OK → write `SPEC.md` ∧ auto-commit `git add SPEC.md` w/ message `init SPEC.md from code`.

## BACKPROP — bug → §B + §V

Input: gate-produced triple (symptom ∧ surface ∧ recurrence-class).

Steps:
1. Parse bug description.
2. Find root cause (read relevant code).
3. Decide: would a new invariant catch recurrence? If yes → draft `V<next>`.
4. Append §B row: `B<next>|<date>|<cause>|V<N>`.
5. Append new invariant to §V.
6. If fix also changes behavior → add/update §T rows.
7. Show diff. On user OK → write `SPEC.md` ∧ auto-commit `git add SPEC.md` w/ message `backprop §B.<n>(+) + §V.<N>(+): <one-line cause>`; ⊥ user prompt for commit step.

Rule: every bug gets a §B entry. Invariant optional but preferred.

## AMEND — targeted edit

Input: gate-produced §-target ∧ delta.

Read that section. Show current. Ask user what changes. On user OK → write `SPEC.md` ∧ auto-commit `git add SPEC.md` w/ message `amend §<S>.<n>(+): <one-line>`; ⊥ user prompt for commit step.

Never silently rewrite sections user did not name.

## OUTPUT RULES

- Math-glyph format per `glyph` skill.
- Preserve identifiers, paths, code verbatim.
- Numbering monotonic — never reuse §V.N or §B.N.
- §T row `cites` column ! list §V/§I deps: `T5|.|impl auth mw|V2,I.api`.

## OUTPUT — "Next" block

Every spec response terminates with a `## Next` block, optionally preceded by a `## Hint` block. Mirrors `commands/spec.md`. Shape (positional dispatch):

- Heading exactly `## Next`.
- Ordered numbered list, 1–5 items.
- Each item = atomic action description (one sentence ∨ phrase). ⊥ `Reply <token>` prefix, ⊥ leading dispatch label (`Reply`, `Run`). Dispatch: user types `run <integer>` → execute item at index; cross-skill jumps via `run /<plugin>:<cmd> [args]` (discriminator: 1st arg digit vs `/`).
- **Actionable only.** Each item ! describe a real state transition. spec presents a diff awaiting decision → apply ∧ revise typically lead; follow-ups (e.g. `/sdd:check` to surface drift, `/sdd:build §T.n` if a pending row exists) sequence per generic positional dispatch.
- No raw file:line edit refs. No §-ref imperatives. No compound items. No prose mid-list. No next-step directives outside the block.
- Items must be applicable to current state. After BACKPROP/AMEND, prefer `/sdd:check` to confirm whether the just-amended invariant or new §T row reveals drift in code; suggest `/sdd:build §T.n` only when a pending §T row exists.

### Hint block (optional)

- Heading exactly `## Hint`. Precedes `## Next`.
- Free-form prose, ≤ 3 lines.
- **Default: skip Hint.** Emit only when context ⊥ derivable from `## Next` item descriptions alone:
  - (a) educational — system mechanic, gotcha, why ordering matters (e.g. "if the new invariant constrains existing code, item 2 surfaces drift before you build").
  - (b) recommends among items via hidden state.
  - (c) warns about side-effect ∨ precondition.
- ⊥ paraphrase Next items, ⊥ restate item count, ⊥ describe what `run 1` literally does.
- ⊥ contain new directives — directives live in Next.

Example after NEW mode (Hint skipped — items self-explanatory):

```
## Next

1. accept the spec as written
2. /sdd:spec rework the V2 invariant before building
3. /sdd:build T1 — start the first task
```

Example after AMEND mode (Hint skipped):

```
## Next

1. apply the diff to `SPEC.md`
2. revise the proposed wording
3. /sdd:check — see whether existing code still satisfies the amended invariant
```

## NON-GOALS

- No sub-agents. Main thread writes.
- No dashboards, no logs, no state files beyond SPEC.md itself.
- No auto-build after spec. User invokes build explicitly.
