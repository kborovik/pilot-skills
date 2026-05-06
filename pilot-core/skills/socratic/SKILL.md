---
name: socratic
description: 'Parameterized single-Q Socratic loop w/ teaching overlay — caller passes (mode-decision space, per-mode convergence triple, intent string); skill returns (converged-mode, facts). Sharpens vague intent until criteria met; concrete first-turn input passes gate ≤ 1 turn, vague triggers dialogue. ⊥ user-invoked surface — invoked from consumer cmd''s pre-create/pre-apply gate. Mechanism: 1-Q-per-turn loop, no-checklist tone, code-reads reactive only, ≥3-turn escape hatch. Caller owns artifact draft ∧ apply; skill owns question selection ∧ convergence detection.'
allowed-tools: AskUserQuestion, Read, Grep
---

# Socratic — parameterized intent gate

## CALLER CONTRACT

caller passes 3 inputs at invocation:

1. **mode-decision space** — set of mode names skill returns. e.g. `{file}` (single-mode gate); `{NEW, DISTILL, BACKPROP, AMEND}` (multi-mode dispatch).
2. **per-mode convergence triple** — facts marking each mode "converged" ∧ ready to draft. shape: `mode → required facts`. e.g. `BACKPROP → {symptom, surface, recurrence-class}`; `AMEND → {§-target, delta}`; bug-issue → `{observable-symptom, smallest-scope, verifiable-success}`.
3. **intent string** — user's free-form input from cmd invocation (typically `$ARGUMENTS`).

skill returns `(converged-mode, facts-collected)` to caller. caller drafts artifact (issue body, §V/§T row, etc.) from facts.

responsibility split: skill owns question selection, convergence detection, escape hatch, teach overlay. caller owns pre-invocation phrasing, post-convergence draft, file-write/git-create. skill ⊥ know caller's artifact shape.

## Loop

1. ask 1 question
2. wait for answer
3. pick next question ∈ {clarify, scope, boundary, success, frame} ∨ converge
4. converge → return `(mode, facts)` to caller

∀ turn: 1 question. ¬ batched. ¬ checklist tone.

## Question pool

| category | fires when                      | shape                                                         |
| -------- | ------------------------------- | ------------------------------------------------------------- |
| clarify  | symptom is vague                | "what specifically — input → observed vs expected?"           |
| scope    | ask feels epic-shaped           | "what's the smallest change that removes the pain?"           |
| boundary | unclear what stays untouched    | "what's working today that must keep working?"                |
| success  | no acceptance criterion         | "how do we know it's fixed without re-asking you?"            |
| frame    | user names a fix, not a problem | "is that the problem, or your current guess at the solution?" |

pick by what's most missing. ¬ re-ask what user already supplied.

## Tone

interrogate problem, ¬ user. questions assume user is right ∧ probe _statement_, ¬ judgment. ¬ "are you sure?" → "what would falsify this?"

## Code reads

read code reactively, ⊥ preemptively.

- ⊥ allowed: grep repo before any questions to "find the bug". undermines dialogue ∧ duplicates caller's broad investigation.
- ✓ allowed: user cites specific `file:line` ∨ symbol ∨ path → read that target. user claims behavior is broken w/o data → spot-check to verify claim before next question.

shape: "looking at `<file>:<line>`, [observable fact] — given that, [next question]". ¬ "I think the bug is X". model verifies, user diagnoses.

scope cap: ≤ 2 reads per turn. broader sweep needed → stop dialogue, return control to caller for codebase investigation.

## Teach

education ≡ overlay, ¬ separate phase. when answer reveals gap, surface distinction ∈ 1–2 sentences, then ask next question. ¬ lecture, ¬ tangent.

teach when:

| trigger                                    | distinction to surface                                   |
| ------------------------------------------ | -------------------------------------------------------- |
| user names a fix as the problem            | symptom vs cause vs solution                             |
| user offers an unfalsifiable success crit  | what makes a criterion verifiable (observable + bounded) |
| user conflates scope w/ ambition           | smallest-change seam vs total redesign                   |
| user assumes a behavior is broken w/o data | observed vs expected vs assumed                          |

shape: "[concept in 1 line] — given that, [next question]". user learns by using distinction on their own problem, ¬ by being told.

## Convergence

ready ⇔ caller-passed convergence triple satisfied for ∃ mode ∈ mode-decision space.

skill ⊥ know what facts mean — only checks presence per caller spec. e.g. caller passes `BACKPROP → {symptom, surface, recurrence-class}`; skill marks BACKPROP converged when all 3 facts present in dialogue history.

precision in caller-passed triples ⇒ less downstream rework — caller's artifact draft consumes facts directly.

≥3 turns w/o convergence → offer escape: "I have enough for a rough draft — return now ∧ refine downstream, or keep going?"

## Escape hatch

"just file it" ∨ "skip the questions" ∨ "I know what I want" → stop dialogue. ⊥ bypass the convergence gate — audit current state vs caller-passed criteria; gaps unmet → either ask once for missing piece ∨ return `(mode, partial-facts)` w/ explicit `unmet-criteria` list so caller can surface gaps in artifact (e.g. `## Unresolved` callout). preserves the validity bar (gaps visible) while honoring user's "done thinking" signal.

## Handoff

return `(converged-mode, facts-collected)` to caller. caller handles draft + apply. skill ⊥ file, ⊥ commit, ⊥ write artifact directly — handoff is data-only.

## Boundary

skill stops at framing-the-question / collecting-the-facts. downstream root-cause analysis, draft formatting, ∧ artifact apply belong to caller-side per-mode handler.
