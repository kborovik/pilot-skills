# SPEC

## §G GOAL

multi-plugin Claude Code marketplace `pilot-skills` → ships `sdd` (spec-driven dev) ∧ `gh` (GitHub workflow) ∧ `core` (shared protocol skills). compressed math-glyph SPEC.md ∧ steno-encoded GitHub artifacts. single-thread Claude, ⊥ orchestration.

## §C CONSTRAINTS

- ⊥ application code, ⊥ tests, ⊥ build pipeline — only plugin manifests, slash cmds, skills, prose docs.
- ∀ published plugin ! project-agnostic; repo-specific recipes ∈ `.claude/` only.
- ∀ plugin layout ≡ `<plugin>/.claude-plugin/plugin.json` + `<plugin>/commands/*.md` + `<plugin>/skills/<n>/SKILL.md`.
- one SPEC.md @ repo root ≡ sole spec file.

## §I INTERFACES

### Published cmds

- `/sdd:spec <free-form intent>` → SPEC.md mutator (sole writer); gate routes to {NEW, DISTILL, BACKPROP, AMEND}.
- `/sdd:build [§T.n|--next|--all]` → plan → edit → verify loop.
- `/sdd:check [§V|§I|§T|--all]` → drift report, read-only.
- `/sdd:explain <§-cite|--next>` → math-glyph → prose for humans.
- `/gh:design <topic>` → propose-then-critique → file issue w/ `design` label.
- `/gh:issue <free-form>` → socratic-gated issue file.
- `/gh:pr <issue#|objective>` → branch off main, push, open PR.
- `/gh:commit [hint]` → commit staged w/ Conventional Commits.
- `/gh:merge [PR#]` → squash-merge w/ release-note body.
- `/gh:release [bump|x.y.z]` → single-package SemVer bump + annotated tag; bails on monorepo.

### Published skills

- `sdd`: `spec`, `build`, `check`, `glyph`, `backprop`.
- `gh`: `design`, `github-issue-create`, `github-pr-create`, `github-pr-merge`, `github-commit-staged`, `github-release`.
- `core`: `socratic`, `steno`. ⊥ commands — skills only.
- cross-plugin ref ≡ `core:<skill>` (namespaced).

## §V INVARIANTS

V0: SPEC.md ≡ LLM-operated artifact (first principle). humans observe → /sdd:explain, mutate → /sdd:spec; ⊥ hand-edit. ∀ other §V derive from V0; affordances solely for human ergonomics (e.g. mid-task wait-state status) ⊥ added.
V1: SPEC.md @ repo root ≡ sole spec file ∧ sole source of truth; `/sdd:spec` ≡ sole mutator — ⊥ hand-edit, ⊥ docs/ tree, ⊥ JSON sidecars.
V2: ∀ published plugin (sdd, gh, core) ! project-agnostic; repo-specific recipes ∈ `.claude/` — ⊥ leak repo-local paths ∨ rules into `<plugin>/`.
V3: published skill (`<plugin>/skills/**`) ∧ cmd (`<plugin>/commands/**`) ∧ plugin README bodies ⊥ cite SPEC §-numbers (§V.N, §T.N, §B.N) — shared artifacts travel between repos w/o pinned numerics. `.claude/**` ∧ `CLAUDE.md` ∧ `SPEC.md` MAY cite (project-local).
V4: SPEC.md ∧ spec-adjacent writes ! glyph encoding — math operators (∀ ∃ ∴ ≡ ⊥ ¬ ≤ ≥ ≠ ∈ ∉ ∧ ∨ → §), fragments, pipe tables.
V5: GitHub-facing artifacts (issue title∧body, PR title∧body, PR squash/merge commit body, release commit body, gh-skill comments) ! steno encoding — readable symbols (→ & | §-cites), ⊥ heavy math glyphs (∀ ∃ ∴ ⊥ ∈ ∉). steno ⊥ apply to Conventional Commits title prefix `type(area):`, code, error strings.
V6: plugin name ≠ dir name — resolve plugin → dir via `.claude-plugin/marketplace.json` `plugins[].source`; ⊥ hardcode dir paths in cmd ∨ skill bodies.
V7: monorepo release via repo-local `/release [--all|<plugin>] [bump|x.y.z]`; published `/gh:release` ≡ single-package only ∧ bails on monorepo by design.
V8: ∀ release mutation ! confirm-before-mutation. single-plugin path: per-plugin confirm gate. `--all` path: single bulk-confirm covers full loop — ⊥ further prompts mid-loop.
V9: bump auto-detect — `<type>(<scope>)?!:` ∨ `BREAKING CHANGE` ∈ body → major; else `feat(<scope>)?:` → minor; else → patch; 0 commits ∈ range → bump = `—`, plugin skipped from loop.
V10: tag format `<plugin>-v<x.y.z>` (e.g. `gh-v1.1.0`, `sdd-v1.5.1`) ∀ monorepo plugin release.
V11: per-plugin commit collection ≡ dir-scope ∪ subject-scope, deduped by SHA, commit order preserved.

- dir-scope: `git log <range> --pretty=format:"%H%x09%s" -- <plugin-dir>/`.
- subject-scope: `git log <range> -E --grep "^[a-z]+\(<plugin>\)!?:"`.
- self-release exclusion pattern: `^chore\(<plugin>\): release <plugin>-v[0-9]+\.[0-9]+\.[0-9]+`.

V12: push policy — single-plugin path: emit manual push instructions (`git push origin main && git push origin <plugin>-v<x.y.z>`). `--all` path: auto-push `origin main` + every newly-cut tag after loop (covered by §V.8 bulk-confirm).
V13: tooling preference — `jq` ∀ `marketplace.json` parse w/ `python3` fallback when jq absent; `rg --pcre2` ∀ regex requiring `\b` word-boundary. ⊥ macOS BSD `awk` for `\b` patterns — silent no-op (rule has bitten this repo before).
V14: `/sdd:build` ⊥ silent retry — verify fail → classify {(a) code bug → fix; (b) spec wrong ∨ (c) unspec edge → invoke backprop skill, append §B + usually new §V, resume against updated spec}.
V15: `/sdd:build` may flip §T status cell only (`.` → `x`); ∀ other SPEC.md edits → `/sdd:spec`.
V16: backprop fires automatically on `/sdd:build` verify fail ∧ `/sdd:check` VIOLATE w/ root cause; user trigger via `/sdd:spec <bug intent>` (gate routes to BACKPROP). ∀ bug → §B row appended; §V invariant optional but preferred when bug class has recurrence risk.
V17: `/sdd:check` ∧ `/sdd:explain` ⊥ writes — read-only diagnostics; suggest remedies via `/sdd:spec` ∨ `/sdd:build`, ⊥ invoke them.
V18: `/gh:issue` ∧ `/sdd:spec` always-gated by `core:socratic` — concrete first-turn input passes gate ≤ 1 turn; vague → single-Q dialogue until convergence triple satisfied; ⊥ `--quick`, ⊥ bypass flag.
V19: `/gh:design` converges on `## Open Questions` ∅ before filing — distinct from socratic (converges on "enough" vs design's "exhausted").
V20: ∀ `/sdd:*` cmd response terminates with `## Next` block — 1–5 numbered atomic actions, positional dispatch (`run <int>` ∨ `run /<plugin>:<cmd> [args]`); optional `## Hint` block precedes Next.
V21: ∀ `<plugin>/commands/*.md` ! have slash-cmd reference ∈ root `README.md` ∧ SPEC.md §I — both surfaces stay in sync w/ published cmd set.
V22: `core` plugin ⊥ commands — ships skills only (`socratic`, `steno`); consumers cite via namespaced ref `core:<skill>`.
V23: §V ∧ §T ∧ §B numbering monotonic — ⊥ reuse N across history.
V24: SPEC.md > 500 lines → compact §B (drop oldest bugs) before splitting — one-file rule preserved.
V25: glyph encoding preserves verbatim — code blocks, paths, URLs, identifiers, numbers, versions, error strings, SQL, regex, JSON, YAML, quoted strings.
V26: `/sdd:spec` post-apply ! auto-fire `/sdd:check --all`; surface drift → user before further `/sdd:*` invocations. ⊥ silent commit-then-done.
V27: published-body examples ! placeholder cite form (`§V.<n>`, `§T.<n>`, `§B.<n>`); pinned numerics (`§V.1`, `§T.3`, `§B.1`, …) ⊥ allowed even in pedagogical examples. convention already established in spec ∧ backprop skills.

## §T TASKS

| id  | status | task                                                                                              | cites  |
| --- | ------ | ------------------------------------------------------------------------------------------------- | ------ |
| T1  | x      | distill initial SPEC.md from current repo state                                                   | -      |
| T2  | x      | strip repo-coupled SPEC §-cites from `<plugin>/**` bodies (pilot-plan/README.md V23 cleared)      | V3     |
| T4  | x      | refresh `CLAUDE.md` §V cites to point at fresh numbers                                            | V1,V23 |
| T5  | x      | run `/sdd:check --all` after T2..T4 → catch residual drift                                        | V17    |
| T6  | x      | drop `~` wait-state refs ∈ pilot-spec/{skills/{build,check,glyph},commands/explain,README}        | V0,V15 |
| T7  | x      | wire post-apply `/sdd:check --all` auto-fire into pilot-spec/skills/spec/SKILL.md                 | V26    |
| T8  | x      | sweep pinned numerics → placeholder form ∀ V3 violations in `<plugin>/**`                         | V3,V27 |
| T9  | x      | drop `~` wait-state ref @ root README.md:72 (root README excluded from §T.6 scope)                | V0,V15 |
| T10 | x      | drop "defined ∈ §I" claim @ CLAUDE.md:31 — `/release` repo-local, ⊥ published                     | V2     |

## §B BUGS

| id  | date       | cause                                                                                                                 | fix |
| --- | ---------- | --------------------------------------------------------------------------------------------------------------------- | --- |
| B1  | 2026-05-06 | V0+V15 amend left `~` refs ∈ pilot-spec/{skills/{build,check,glyph},commands/explain,README}; derivative ⊥ propagated | V26 |
| B2  | 2026-05-06 | V3 violations: pinned numerics @ pilot-spec/{commands/explain,skills/{check,glyph},README}, pilot-core/skills/steno    | V27 |
| B3  | 2026-05-06 | §B.1 derivative-leak class: §T.6 scoped to pilot-spec/{...,README}, root README.md:72 excluded ∴ `~` ref persisted     | V26 |
| B4  | 2026-05-06 | derivative free-text drift sub-class: CLAUDE.md:31 claimed `/release ∈ §I`, but only `/gh:release` published ∴ V2 leak | V2  |
