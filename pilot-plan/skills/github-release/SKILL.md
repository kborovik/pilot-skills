---
name: github-release
description: Cut SemVer release for single-package repo — bump `version` in detected manifest, commit, create annotated tag `v<x.y.z>`. Local only, ⊥ push, ⊥ GitHub Actions. Triggers when user says "release", "tag", "ship", "cut a version", "bump version".
argument-hint: [patch|minor|major|x.y.z|retag-baseline]
model: sonnet
allowed-tools: Bash(git *), Read, Edit
---

Cut SemVer release for single-package repo. Local workflow — bumps version in repo's package manifest, commits, inits annotated tag `v<x.y.z>`. ⊥ push.

## Scope

Single-package repos only. ⊥ monorepo orchestration — ≥ 2 manifests at distinct paths → bail with note: "monorepo detected, use a repo-local release procedure".

## Manifest auto-detect

Grep at repo root in priority order. First hit wins:

```
1. package.json            → .version
2. pyproject.toml          → [project].version  | [tool.poetry].version
3. Cargo.toml              → [package].version
4. .claude-plugin/plugin.json → .version       (single-plugin repo)
5. VERSION                 → raw SemVer text
```

⊥ found → ask user which file holds version ∧ how to update.

Tag format: `v<x.y.z>` (e.g. `v1.0.0`, `v2.3.1`).

## Process

1. **Parse $ARGUMENTS:**
   - ⊥ arg → auto bump (∨ baseline-mode if first release).
   - `patch` ∨ `minor` ∨ `major` → bump direction override.
   - `x.y.z` (literal SemVer) → pin exactly. ! match `^[0-9]+\.[0-9]+\.[0-9]+$`. Permits downgrade.
   - `retag-baseline` → recovery mode. Drops every prior tag matching §3 pattern ∧ retags HEAD at current manifest version. Requires ≥ 1 prior tag (else error: "no prior tag — use baseline mode by omitting arg"). ⊥ touches manifest. If manifest version is also wrong, fix it manually first ∨ use `x.y.z` form instead.

2. **Detect manifest** (per table above). ≥ 2 candidates → bail (monorepo).

3. **Find last tag:**
   - `git tag --list "v*" --sort=-v:refname | head -n 1`
   - ⊥ tag → first release (baseline mode applies in §6).

4. **Read current version** from manifest. Invalid SemVer → error.

5. **Collect commits in range:**
   - Range: `<last-tag>..HEAD` if tag exists, else full history.
   - **Retag-baseline mode** → range = full history regardless of prior tags (redoing baseline; prior tags about to be dropped).
   - `git log <range> --pretty=format:"%H%x09%s"`
   - **Exclude** commits whose subject matches `^chore(\([^)]+\))?: release v[0-9]+\.[0-9]+\.[0-9]+` — skill's own release commits ⊥ count toward bump.
   - ⊥ commits remaining → exit cleanly w/ this exact shape (bypassed in retag-baseline mode):
     ```
     Nothing to release.
       Range:                   <last-tag>..HEAD   (or "full history" if no prior tag)
       Last tag:                <last-tag-or-NONE>
       Manifest @ version:      <manifest-path> @ <current-version>
       Commits in range:        <total>
       Filtered self-release:   <n>   (matches §5 regex)
       Commits remaining:       0

     Next steps:
       - Add commits, then re-run.
       - To force a release with no new commits, pass an explicit `<x.y.z>` arg.
       - To recover a wrong prior tag, pass `retag-baseline`.
     ```
   - ⊥ side effects beyond stdout. Same fail-loud spirit as the confirm-step (auto-detect breakdown ∧ user-confirm-before-mutate) — ⊥ silent exits.

6. **Determine target version:**
   - Arg = `retag-baseline` → target = current manifest version. Manifest unchanged. Proceeds to drop prior tags in §11.5.
   - Arg = `x.y.z` → target = `x.y.z`. Skip auto-detect.
   - Arg = `patch`/`minor`/`major` → bump from current. Skip auto-detect.
   - ⊥ arg ∧ ⊥ prior tag → **baseline mode**: target = current version (use what's in manifest, ⊥ bump). Sets the starting tag at the version already declared.
   - ⊥ arg ∧ prior tag exists → auto-detect from commit subjects:
     - Any `<type>(<scope>)?!:` ∨ body containing `BREAKING CHANGE` → **major**
     - Else any `feat(<scope>)?:` → **minor**
     - Else → **patch**

7. **Compute next version** (skip if explicit `x.y.z`):
   - patch: `x.y.z` → `x.y.(z+1)`
   - minor: `x.y.z` → `x.(y+1).0`
   - major: `x.y.z` → `(x+1).0.0`
   - baseline: target = current

8. **Render release notes** (steno-styled, grouped):
   - Group commits by Conventional Commits type:
     - **Features** ← `feat`
     - **Fixes** ← `fix`
     - **Other** ← `refactor` / `docs` / `chore` (non-release) / `test` / `perf` / `build` / `ci` / `style`
     - **Uncategorized** ← anything not matching `<type>(<scope>)?:` prefix
   - Each entry: bullet with subject minus `<type>(<scope>):` prefix. Keep scope only if it disambiguates.
   - Preserve `#refs`, paths, identifiers, SHAs verbatim.
   - Apply `core:steno` skill — fragments, ⊥ filler.
   - Baseline mode + only-init-commits → release notes may be empty / sparse. Acceptable.

9. **Confirm before mutation** — render breakdown so bump derivation auditable. Required fields:
   - **Manifest**: path ∧ current version (e.g. `<plugin-dir>/.claude-plugin/plugin.json @ <x.y.z>`).
   - **Range**: `<last-tag>..HEAD` or `<full history>` if baseline.
   - **Commit counts by Conventional Commits type**: e.g. `feat:0  fix:0  refactor:1  docs:0  chore:0  breaking:0  uncategorized:0`. Cover all types observed in range.
   - **Filtered self-release commits**: count of commits matching §5 self-release regex. Show count even if zero (proves the self-release filter ran).
   - **Bump derivation**: explicit one-liner showing rule fired — e.g. `0 breaking + 0 feat → patch (default)`, `1 breaking → major`, `arg "x.y.z" → pinned`, `no prior tag & ⊥ arg → baseline (no bump)`, `arg "retag-baseline" → recovery (no bump, manifest unchanged)`.
   - **Target tag**: `v<x.y.z>` (∨ `<plugin>-v<x.y.z>` for monorepo orchestrator).
   - **Tags scheduled for deletion** (retag-baseline only): list every prior tag matching §3 pattern. Note: if any pushed, user must drop them remotely too — instructions in §13.
   - **Rendered release notes** (steno-styled, grouped per §8).
   - Wait for user confirmation. ⊥ confirm → exit, ⊥ side effects.

10. **Bump version in manifest** (skip if baseline mode ∨ retag-baseline mode — manifest already at target):
    - Edit manifest → set version field.
    - Stage: `git add <manifest-path>`.

11. **Commit** (skip if baseline mode ∨ retag-baseline mode — manifest unchanged):
    - Heredoc + `--cleanup=verbatim` (keeps any future `#`-prefix lines from being stripped as git comments):
      ```
      git commit --cleanup=verbatim --message "$(cat <<'EOF'
      chore: release v<x.y.z>
      EOF
      )"
      ```

11.5 **Delete prior tags** (retag-baseline mode only):
    - For every tag listed in §9 "Tags scheduled for deletion":
      ```
      git tag --delete <tag>
      ```
    - Local deletion only at this step. Remote deletion → §13.

12. **Init annotated tag w/ notes inline** — `--cleanup=verbatim` is mandatory (release notes use `## Features` / `## Other` headers that git's default cleanup would strip as comments):
    ```
    git tag --annotate --cleanup=verbatim v<x.y.z> --message "$(cat <<'EOF'
    v<x.y.z>

    <release notes>
    EOF
    )"
    ```

13. **Echo result to user:**
    - Tag name, target SHA, computed bump.
    - Re-print rendered annotation (else hidden in `git show <tag>`).
    - Push instructions: `git push origin main && git push origin v<x.y.z>`. ⊥ push automatically.
    - Retag-baseline mode → also instruct remote-tag deletion for each dropped tag: `git push origin :refs/tags/<old-tag>`. Skip for tags ⊥ pushed.

## Release notes shape

```
## Features
- new release skill (`/gh:release`)
- add `--auto` example for merge queues

## Fixes
- token expiry boundary (`<` → `≤`)

## Other
- rewrite skills in math-glyph encoding
- drop unused gh CLI flag reference blocks
```

## Style

Apply `core:steno` skill to release notes. Drop articles ∧ filler, fragments ∧ bullets, preserve identifiers / paths / `#refs` / SHAs verbatim. Tag name ∧ version string fixed → ⊥ compress.

## Requirements

- Single-package only. Monorepos → bail; user runs repo-local release procedure instead.
- SemVer only — `x.y.z`, ⊥ pre-release suffixes (skill scope = simple).
- Annotated tag (`-a` / `--annotate`), ⊥ lightweight.
- Local only — ⊥ `git push`, ⊥ `gh release create`. User pushes manually.
- ⊥ render `CHANGELOG.md` — release notes live in tag annotation.
- Baseline mode for first release: ⊥ bump if no prior tag ∧ ⊥ explicit arg. Tags version already declared in manifest.
- Retag-baseline recovery: explicit `retag-baseline` arg drops every prior tag matching pattern ∧ retags HEAD at current manifest version. Requires ≥ 1 prior tag. Manifest unchanged. If manifest is also wrong, fix it manually first ∨ use `x.y.z` arg (permits downgrade).
- Self-release commits (`chore: release v*`) excluded from auto-detect scan → empty range exits cleanly.
- Confirm before mutation. ⊥ silently rewrite version files. Confirm step ! list tags scheduled for deletion in retag-baseline mode.
- All `git tag --annotate` ∧ `git commit` calls in this skill ! pass `--cleanup=verbatim`. git's default `commit.cleanup=strip` would silently drop `#`-prefix lines (release-note section headers) from messages.
