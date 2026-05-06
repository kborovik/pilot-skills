---
description: Cut SemVer releases across this monorepo. Bare `/release` ≡ `--all` autonomous — surveys every plugin, single bulk-confirm, runs each {bump → commit → tag}, then auto-pushes `origin main` + all new tags. Pass `<plugin>` for single-plugin path (per-plugin confirm + manual push). Repo-local; not part of the published `gh` plugin.
argument-hint: "[--all|<plugin>] [patch|minor|major|x.y.z]"
---

# /release — cut plugin releases (pilot-skills monorepo)

Repo-local orchestration for the pilot-skills monorepo. The published `/gh:release` skill is single-package only and bails on this repo because there are multiple `plugin.json` manifests. This command bridges the gap.

Two paths:

- **`--all` autonomous mode** (default — bare `/release` ≡ `/release --all` per V19): surveys every plugin, emits a single bulk-confirm covering the full loop, then iterates marketplace-order, running each plugin's {bump → commit → tag} autonomously (no per-plugin confirm — V7 carve-out), and finally pushes `origin main` + every newly-cut tag (V12 carve-out).
- **Single-plugin mode** (`/release <plugin> [bump|x.y.z]`): runs the same Process for one plugin, but with the per-plugin confirm gate (V7 default) and manual push (V12 default) preserved — unchanged from prior behavior.

Apply the same logic as `pilot-plan/skills/github-release/SKILL.md` with the following adaptations:

## Plugin → directory resolution

Source of truth: `.claude-plugin/marketplace.json` `plugins[].source`. Resolve at runtime; do not hardcode (V5).

**Primary** (jq):

```bash
jq -r --arg p "<plugin>" '.plugins[] | select(.name == $p) | .source' .claude-plugin/marketplace.json
```

Returns e.g. `./pilot-plan` for `<plugin>=gh`. Strip leading `./` to get `<plugin-dir>`. Manifest path = `<plugin-dir>/.claude-plugin/plugin.json`.

**Fallback** (jq unavailable):

```bash
python3 - "<plugin>" <<'PY'
import json, sys
m = json.load(open('.claude-plugin/marketplace.json'))
print(next(p['source'] for p in m['plugins'] if p['name'] == sys.argv[1]))
PY
```

**List available plugins** (for unknown-plugin error in §1):

```bash
jq -r '.plugins[].name' .claude-plugin/marketplace.json
```

Empty resolution → unknown plugin → list available names & exit.

Currently resolves:

```
gh   → ./pilot-plan   (manifest: pilot-plan/.claude-plugin/plugin.json)
sdd  → ./pilot-spec (manifest: pilot-spec/.claude-plugin/plugin.json)
```

Tag format: `<plugin>-v<x.y.z>` (e.g. `gh-v1.1.0`, `sdd-v1.0.0`).

## --all autonomous mode

Per V19. Triggered by bare `/release` (defaults to `--all`) or explicit `/release --all`. Survey is read-only; mutations gated by a single bulk-confirm before the loop starts.

1. **Enumerate plugins** from `marketplace.json`:

   ```bash
   jq -r '.plugins[] | "\(.name)\t\(.source)"' .claude-plugin/marketplace.json
   ```

   Fallback when jq is absent (mirrors the resolution block above):

   ```bash
   python3 - <<'PY'
   import json
   m = json.load(open('.claude-plugin/marketplace.json'))
   for p in m['plugins']:
       print(f"{p['name']}\t{p['source']}")
   PY
   ```

2. **Per plugin**, compute the survey row:
   - Strip leading `./` from `<source>` to get `<plugin-dir>`.
   - Last tag: `git tag --list "<plugin>-v*" --sort=-v:refname | head -n 1`.
   - Range: `<last-tag>..HEAD` if tag exists, else full history.
   - Commit collection — union of dir-scope and subject-scope, deduped by SHA, with self-release exclusion (V11 + V9):

     ```bash
     {
       git log <range> --pretty=format:"%H%x09%s" -- <plugin-dir>/
       git log <range> --pretty=format:"%H%x09%s" -E --grep "^[a-z]+\(<plugin>\)!?:"
     } | sort -u | grep -vE "	chore\(<plugin>\): release <plugin>-v[0-9]+\.[0-9]+\.[0-9]+"
     ```

   - Auto-detected bump (V8 rules):
     - Any `<type>(<scope>)?!:` subject or `BREAKING CHANGE` in body → **major**.
     - Else any `feat(<scope>)?:` → **minor**.
     - Else → **patch**.
     - 0 commits in range → bump = `—`.
   - Target tag: `<plugin>-v<next-version>` where `<next-version>` = current manifest version + bump (or unchanged if 0 commits).

3. **Render the survey** as a pipe table to the user:

   ```
   | plugin | last tag | commits | bump | target |
   | ---    | ---      | ---     | ---  | ---    |
   | gh     | gh-v1.1.0   | 3 | patch | gh-v1.1.1   |
   | sdd    | sdd-v1.3.1  | 1 | patch | sdd-v1.3.2  |
   | core   | core-v1.0.0 | 0 | —     | —           |
   ```

   Plugins with 0 commits in range are still listed — row included with `—` bump for visibility (V19); they are skipped from the release loop.

4. **Single bulk-confirm** (V7 carve-out for `--all` path — covers ∀ plugin; per-plugin confirm gates ⊥ fire mid-loop):

   > Cut all releases shown above (skipping `—` rows), then push `origin main` + every new tag? Reply `ok` to proceed, `cancel` to exit.

5. **On `ok`** → iterate plugins in `marketplace.json` `plugins[]` order, **skipping** any with 0 commits in range. For each, run the full Process (§1–§13) **autonomously** (no per-plugin confirm — V7 carve-out): bump manifest → commit → annotated tag. Per-plugin failure aborts the loop with a partial-state report; survivors are not rolled back (user inspects + reruns).

   On `cancel` / empty reply → exit with no side effects.

6. **After the loop** (V12 carve-out for `--all` path — auto-push covered by the single bulk-confirm in §4):

   ```bash
   git push origin main
   for tag in <newly-cut-tags>; do git push origin "$tag"; done
   ```

   Report the pushed refs back to the user.

The `--all` path itself never mutates state until §5 begins. All confirm-before-mutation logic for this path lives in §4. The single-plugin Process below retains its own per-plugin confirm gate and manual-push instructions (V7, V12 unmodified there).

## Process

Same 13 steps as the `github-release` skill, with these substitutions:

1. **Args**: `<plugin>` is the first positional arg. Bare `/release` (no args) ∨ `/release --all` → enter **--all autonomous mode** above instead (V19); the Process below runs per-iteration inside that loop without the per-plugin confirm gate. Explicit `/release <plugin> [bump|x.y.z]` runs the Process directly with the per-plugin confirm gate (V7) and manual push instructions (V12) preserved. Resolve plugin name → directory via `marketplace.json`. Unknown plugin → list available plugins, exit.
2. **Manifest detection** is bypassed: manifest path = `<plugin-dir>/.claude-plugin/plugin.json`.
3. **Last tag** lookup: `git tag --list "<plugin>-v*" --sort=-v:refname | head -n 1`.
4. **Commit collection** = union of two sources (V11):
   - **Dir-scope**: commits in range touching files in `<plugin-dir>/`.
     ```
     git log <range> --pretty=format:"%H%x09%s" -- <plugin-dir>/
     ```
   - **Subject-scope**: commits in range whose Conventional Commits subject scope matches the plugin name.
     ```
     git log <range> --pretty=format:"%H%x09%s" -E --grep "^[a-z]+\(<plugin>\)!?:"
     ```
   - Merge by SHA (dedupe), preserve commit order.
   - Self-release exclusion (skill §5) applies to the merged set.
   - **Cross-attribution caveat**: a `feat(<other-plugin>)` commit that *also* touches `<plugin-dir>/` is collected for both releases. Acceptable minor noise — user trims notes manually if it matters.
5. **Self-release commits to exclude** match: `^chore\(<plugin>\): release <plugin>-v[0-9]+\.[0-9]+\.[0-9]+`.
6. **Baseline mode**: identical — first release with no prior `<plugin>-v*` tag and no bump arg → tag at the version already declared in the plugin's `plugin.json`.
7. **Commit message**: `chore(<plugin>): release <plugin>-v<x.y.z>`.
8. **Tag name**: `<plugin>-v<x.y.z>`.
9. **Push** — depends on path:
   - **Single-plugin** (`/release <plugin>`): emit manual push instructions at the end: `git push origin main && git push origin <plugin>-v<x.y.z>`. V12 default.
   - **`--all` autonomous**: skip per-plugin push; the loop's §6 (above) auto-pushes `origin main` + every newly-cut tag once after the loop completes. V12 carve-out, covered by the single bulk-confirm in §4.

All other behavior — Conventional Commits auto-detect, steno-styled grouped notes, annotated tag with notes inline, no `CHANGELOG.md` — is identical to the published skill. Confirm-before-mutation: present per-plugin in single-plugin path; collapsed into the single bulk-confirm at §4 of `--all` mode.

## Requirements

- Resolve plugin → directory from `marketplace.json` so directory renames stay safe.
- Confirm before mutation:
  - **Single-plugin path** (`/release <plugin>`) — per-plugin confirm gate (V7).
  - **`--all` autonomous path** (bare ∨ `/release --all`) — single bulk-confirm before the loop, then no further prompts (V7 carve-out).
- Push:
  - **Single-plugin path** — manual push instructions emitted (V12).
  - **`--all` autonomous path** — auto-push `origin main` + every newly-cut tag after the loop (V12 carve-out).
- Repo-local only. Do not graduate this into the published `gh` plugin without a config-driven design first — see `CLAUDE.md`'s "all skills in the repo MUST NOT be project specific" rule.
