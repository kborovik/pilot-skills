# pilot-skills — Claude Code plugin marketplace

Multi-plugin marketplace. ⊥ code, ⊥ tests, ⊥ build — only plugin manifests, skills, slash cmds. SPEC.md @ repo root ≡ source of truth: §G (goal), §C (constraints), §I (cmd ∧ skill catalog), §V (invariants).

## Layout

```
.claude-plugin/marketplace.json   marketplace manifest (∀ plugins)
pilot-spec/                       plugin "sdd"
pilot-plan/                       plugin "gh"
```

∀ plugin contains:

```
<plugin>/.claude-plugin/plugin.json   manifest Claude Code reads (single source of truth)
<plugin>/commands/<n>.md              slash cmd files
<plugin>/skills/<n>/SKILL.md          skills (one dir per skill)
```

Dir name ≠ plugin name (see §V.6).

## Developing this repo

Use `sdd` plugin (this repo's own `pilot-spec/`) → drive changes. SPEC.md ≡ source of truth (§V.1); `/sdd:*` cmd surface ∈ §I, behavior governed by §V.

Spec writes use `glyph` skill (math-glyph); GitHub-facing text uses `steno` skill — see §V.4 ∧ §V.5. Bug → §V invariant via `backprop` skill (auto-fired by build ∧ check on test fail; user trigger `/sdd:spec <free-form intent>` — gate routes to BACKPROP on bug-class intent).

⊥ bypass `/sdd:build` ∀ non-trivial changes — dogfooding ≡ the point.

Monorepo release: `/release [--all|<plugin>] [bump]` (repo-local, `.claude/`) per §V.7.

## Adding things

- **New plugin** → new top-level dir + `.claude-plugin/plugin.json` + entry ∈ root `marketplace.json` `plugins[]`.
- **New cmd** → `<plugin>/commands/<n>.md` w/ frontmatter (`description`, `argument-hint`). Surfaces as `/<plugin-name>:<n>`. Sync slash-ref ∈ README.md ∧ SPEC.md §I per §V.21.
- **New skill** → `<plugin>/skills/<n>/SKILL.md` w/ frontmatter (`name`, `description`). Body ∧ description ∈ math-glyph per §V.4; `description` ≡ what Claude reads to decide when to fire — be specific re triggers.
- **New invariant ∨ §T row** → `/sdd:spec` (sole SPEC.md mutator).

## Don't

- ⊥ add code/build/test sections — ⊥ code (§C).
- ⊥ restate §G/§I/§V content — defer via §-cite per §V.1.
