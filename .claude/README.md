# Repo-local enforcement recipes

Each `check-V<n>.md` here is the executable form of a SPEC.md §V invariant whose enforcement is project-specific (V1 carve-out: published plugins must stay project-agnostic; the per-invariant recipes that name this repo's literal paths live here, not in `pilot-spec/skills/check/`).

| recipe         | enforces | summary                                                                                         |
| -------------- | -------- | ----------------------------------------------------------------------------------------------- |
| `check-v17.md` | §V.17    | every `<plugin>/commands/*.md` has a slash-ref in `README.md` ∧ `SPEC.md §I`                    |
| `check-v31.md` | §V.31    | published skill ∧ cmd bodies ⊥ cite pre-rename slash-cmd literals across 3 forms                |
| `check-v38.md` | §V.38    | published skill ∧ cmd bodies ⊥ cite this repo's pinned V/T/B/§-numbers as load-bearing rationale |
| `release.md`   | §V.6/V19 | monorepo release orchestrator (per-plugin SemVer bump + tag, `marketplace.json`-driven)         |

## Required tools

- `jq` — all recipes parse `marketplace.json`.
- `ripgrep` (`rg`) — `check-v38.md` uses `rg --pcre2` for real `\b` word-boundaries (macOS BSD `awk` ⊥ supports `\b`, the prior awk-only regex was a silent no-op — see `SPEC.md §B.34`). Install: `brew install ripgrep` (macOS) ∨ `apt install ripgrep` (Debian/Ubuntu) ∨ `cargo install ripgrep`.
- `awk` (POSIX) — fence-state tracking ∧ backtick-span stripping; no GNU extensions.
- `git` — release recipe.
- `gh` (GitHub CLI) — release recipe (tag push / PR ops if invoked).

## Running

Recipes are invoked as Claude Code slash commands (`/check-v17`, `/check-v31`, `/check-v38`, `/release`). To run the bash directly, extract the fenced ```bash``` block:

```sh
sed -n '/^```bash/,/^```$/p' .claude/commands/check-v38.md | sed '1d;$d' | bash
```

A successful run prints one `HOLD V<n>: …` line; a failed run prints `VIOLATE V<n>: <file>:<ln>` per hit and exits non-zero.
