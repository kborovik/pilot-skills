---
name: design
description: 'Propose-then-critique structural design loop → file new GitHub issue w/ `design` label. Distinct from socratic — socratic sharpens vague intent (converges on "enough"); design exhausts open questions before filing (converges on "Open Questions ∅"). Use when user wants to design a structural change, weigh tradeoffs between named alternatives, propose an architecture, or shape a subsystem before implementation. Triggers: "/gh:design", "design the X", "shape the X subsystem", "tradeoffs between A and B", "how should we structure", "propose an architecture for". Skip when intent already concrete ∧ no open structural Qs → route to /gh:issue.'
allowed-tools: AskUserQuestion, Read, Grep, Bash
---

# Design — propose-then-critique → fileable design issue

## Loop

1. read `SPEC.md` ∈ root → degrade gracefully if absent
2. topic vague ∨ empty → ≤ 2 questions to localize, then propose
3. propose shape (named structures, types, key decisions) ∈ 1 pass
4. surface `## Open Questions` list at bottom
5. wait → user critique / answers
6. update Proposal in place; resolved Qs → `## Design decisions` w/ rationale
7. repeat 5–6 until `## Open Questions` ∅
8. on confirm → file new issue via `gh issue create` w/ `design` label

∀ turn: ⊥ self-resolve Open Questions. resolution ⊢ user input.

## Distinction from socratic

| skill    | converges on | mechanism                       |
| -------- | ------------ | ------------------------------- |
| socratic | "enough"     | 1 question/turn, sharpen intent |
| design   | "exhausted"  | propose shape, exhaust open Qs  |

⊥ merge. socratic = bug ∨ small-feature framing. design = structural choice.

## Output template (issue body)

body ∈ steno (readable shorthand for GitHub-facing text) — `→ & | §` allowed; ⊥ heavy math glyphs (`∀ ∃ ∴ ⊥ ∈ ∉`). § citations OK if `SPEC.md` present.

```
## Problem

[symptoms + §B/§V citations if SPEC.md present, else "designing without SPEC anchor"]

## Proposal

[named structures, types, shape — propose-then-critique starting point]

## [topic-specific sections, e.g. "Tool ownership", "Naming", "Layering"]

## Effect on in-flight SPEC items

[§T/§V deltas — what gets superseded, narrowed, unchanged. omit section if SPEC.md absent.]

## Design decisions

[each resolved Open Q + rationale, in `**Decision:** ... **Why:** ...` shape]

## Success criterion

[observable invariants — "X cannot recur", "Y returns Z", measurable]

## Out of scope

[deferred → §T row ∨ future issue]

## Unresolved

[only if ≥3-turns/Q escape used — parked Qs for follow-up]
```

## Code reads

reactive only. ⊥ preemptive scans.

- ⊥ allowed: grep repo before first proposal "to find context". propose from user's framing + `SPEC.md`.
- ✓ allowed: user cites `file:line` ∨ symbol ∨ path → read that target. user claims behavior in code → spot-check before next proposal turn.

cap: ≤ 2 reads/turn. broader sweep needed → stop, hand to `/gh:issue` (broad investigation by design).

## SPEC.md degradation

`SPEC.md` ∈ root absent → flag once: "designing without SPEC anchor; §V/§B/§T citations omitted". continue. omit `## Effect on in-flight SPEC items` from output.

## Long-session escape

single Open Q ≥ 3 turns w/o resolution → offer: "park this Q under `## Unresolved` in body ∧ converge on the rest, or keep going?"

park ⇒ filed issue carries explicit unresolved list ∈ `## Unresolved` section. ⊥ pretend resolved.

## Mode

file-new-issue only. ⊥ comment-on-existing.

## Title

conventional-commits prefix: `feat(<scope>): ...`, `refactor(<scope>): ...`, etc. `design` label carries design-ness. ⊥ double-encode "design:" ∈ title.

## Filing

1. resolve target repo: `gh repo view --json nameWithOwner`
2. ensure `design` label: `gh label list --json name | jq -r '.[].name' | grep -q '^design$'` → absent? `gh label create design --description "Structural design issue — propose-then-critique" --color "BFD4F2"`
3. write body to tmpfile (steno-formatted per template)
4. `gh issue create --title "<conventional-commits title>" --label design --body-file <tmpfile>`
5. show URL on success

## Convergence gate

ready ⇔ `## Open Questions` ∅ ∧ user confirms.

⊥ file w/o confirmation. ⊥ self-resolve Qs. ⊥ collapse multiple Qs into one to fake convergence.

## Boundary

⊥ mutate `SPEC.md`. design produces issue. SPEC amendment ⊢ caller runs `/sdd:spec` after issue filed. impl ⊢ `/sdd:build` after spec amended.

⊥ root-cause debugging — that's `/sdd:backprop`. design = structural shape, ⊥ "why is this broken".

## Escape hatch

"just file it" ∨ "skip the design" ∨ "I already know what I want" → stop. hand verbatim intent to `/gh:issue`.
