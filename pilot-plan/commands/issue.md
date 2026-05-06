---
description: Create a GitHub issue. Always-gated by `core:socratic` skill — concrete intent passes ≤ 1 turn, vague intent triggers dialogue until convergence.
argument-hint: <issue description | vague intent>
---

# /gh:issue — gated GitHub issue filing

Two-phase invocation. Phase 1 enforces a validity gate; Phase 2 files the issue.

## Phase 1 — socratic gate (always engaged)

Invoke the `core:socratic` skill with `$ARGUMENTS`.

If `$ARGUMENTS` is empty, ask the user for the issue intent (problem, idea, bug, feature) before proceeding.

The gate evaluates input against three **convergence criteria** before passing downstream:

1. **explicit goal** — observable symptom (bug) ∨ concrete behavior delta (feature/refactor)
2. **acceptance criterion ∨ repro step** — at least one verifiable outcome ∨ reproducible failure
3. **§V/§T citation ∨ ⊥-spec flag** — cite a §V invariant ∨ §T row this issue concerns, ∨ explicitly flag the issue as out-of-spec scope

**Fast-path** — concrete first-turn input meeting all three criteria passes the gate in ≤ 1 turn (zero-friction for prepared authors). The gate is convergence-detection, ⊥ interrogation-for-its-own-sake.

**Dialogue path** — vague ∨ under-specified input triggers the `core:socratic` loop (single-question, just-in-time teaching) until criteria are met. Per skill spec: ≥ 3 turns w/o convergence → escape protocol offered.

⊥ skip flag, ⊥ `--quick` bypass, ⊥ author opt-out. Rationale: GitHub issues = team-shared artifacts; validity bar > author keystroke savings.

## Phase 2 — file the issue

After gate passes, invoke the `github-issue-create` skill with the gated, sharpened input.

The skill handles: codebase investigation, Conventional Commits title, label selection, `gh issue create`, and posting an insights comment. Body in `steno` style (readable shorthand for GitHub-facing text).

## Escape-hatch boundary

The `core:socratic` escape protocol ("just file it" ∨ "skip the questions" ∨ "I know what I want") stops dialogue but does **⊥ bypass the gate**. On escape:

1. evaluate current input against convergence criteria
2. if criteria still unmet → either ask once for the missing piece, ∨ proceed to Phase 2 with explicit `## Unresolved` callouts in the body documenting each unmet criterion

This preserves the validity bar (gaps become visible to the team in the issue body) while honoring the user's "I'm done thinking" signal. The escape is artifact-side (the gate still records gaps in the body) — it does not bypass enforcement.

## OUTPUT

Phase 1 output: socratic dialogue turns ∨ fast-path acknowledgment ("intent meets convergence criteria — proceeding to file").

Phase 2 output: filed issue URL + number, plus insights comment URL.

## NON-GOALS

- ⊥ direct file path that bypasses Phase 1.
- ⊥ separate `--think` ∨ `--quick` flags — gate is unconditional.
- ⊥ commenting on existing issues (use `gh issue comment` directly ∨ a future cmd).
