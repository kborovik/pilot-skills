# pilot-skills

Claude Code plugin marketplace. Three plugins, one repo.

## The bet

**Review throughput is the bottleneck, not generation throughput.**

Modern LLMs already write code faster than any human can read it. The scarce resource isn't the agent — it's the reviewer who has to sign off, and every additional agent, file, and diff multiplies what they have to hold in their head. Multi-agent orchestration optimizes for the wrong end of the pipeline: sub-agents fan out in parallel, race on shared files, step on each other's edits, and produce overlapping diffs that no single person can hold in working memory long enough to audit. The merge looks coherent only because the tooling glued it together — the _reasoning_ behind it is scattered across N agent transcripts nobody reads. You ship more code; you audit less of it; bugs land in the gaps between agents.

PilotSpec inverts that. Single-thread Claude, one `SPEC.md` as the durable artifact, math-glyph compression so the spec stays small enough to read end-to-end. The thing a human reviews is the spec — invariants, tasks, bugs — not the agent's stream of consciousness. Generation is cheap. The spec is what costs, and what's worth keeping.

## Position in the SDD landscape

```
cavekit ── PilotSpec (this repo) ── spec-coding-mcp ── spec-kit ── Tessl ── Kiro ── Superpowers ── BMAD ── MUSUBI
[lightweight ────────────────────────────────────────────────────────────────────────────── full orchestration]
```

**vs. [Superpowers](https://github.com/obra/superpowers)** (the 166k-star leader): Superpowers fans out to sub-agents and runs wave-based execution to ship faster. PilotSpec does the opposite — single-thread Claude, one `SPEC.md`, no orchestration. PilotSpec also writes its spec in math-glyph (∀ ∃ ∴ ⊥ §) at ~75% token reduction and persists it as the durable artifact — Superpowers, spec-kit, Kiro, and BMAD all use prose markdown across multiple files. Different design centers, not just different sizes.

## Plugins

| brand         | plugin                 | commands                                                                      | description                                                        |
| ------------- | ---------------------- | ----------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| **PilotSpec** | [`sdd`](./pilot-spec)  | `/sdd:spec`, `/sdd:build`, `/sdd:check`, `/sdd:explain`                       | Compressed spec-driven dev over one `SPEC.md`.                     |
| **PilotPlan** | [`gh`](./pilot-plan)   | `/gh:commit`, `/gh:issue`, `/gh:design`, `/gh:pr`, `/gh:merge`, `/gh:release` | GitHub workflow today; planning surface tomorrow.                  |
| **PilotCore** | [`core`](./pilot-core) | —                                                                             | Shared encoder/protocol skills (`socratic`, `steno`). No commands. |

`core` is a foundation plugin (directory `pilot-core/`) — it ships shared skills (a Socratic intent-gate and the `steno` GitHub-text encoder) that both `sdd` and `gh` invoke via namespaced refs (`core:socratic`, `core:steno`). Both plugins declare it in `dependencies`, so Claude Code auto-installs `core` when you install either consumer plugin.

## Install

```bash
/plugin marketplace add kborovik/pilot-skills
/plugin install sdd@pilot-skills          # core auto-installs
/plugin install gh@pilot-skills           # core auto-installs
```

## PilotSpec — spec-driven dev (`sdd` plugin)

Single `SPEC.md` at repo root. Math-glyph-encoded sections (math operators ∀ ∃ ∴ ⊥ ∈ ∉ ≤ ≥ ≠ ∧ ∨ → § + pipe tables, ~75% token cut vs prose). Plan → execute → backprop loop.

```bash
/sdd:spec      # write or amend the spec — invariants (§V), tasks (§T), bugs (§B)
/sdd:build     # plan-then-execute next task; auto-backprops on test/build failure
/sdd:check     # read-only drift report — code vs §V / §I / §T
/sdd:explain   # decompress any §X.n citation back into plain English
```

### The mental model

```
   ┌────────────┐         ┌──────────────┐         ┌────────────┐
   │  /sdd:spec │ ──────► │   SPEC.md    │ ◄────── │ /sdd:check │
   │  (mutator) │         │ (truth file) │         │ (read-only)│
   └────────────┘         └──────┬───────┘         └────────────┘
                                 │
                                 ▼
                          ┌──────────────┐
                          │  /sdd:build  │
                          │ (plan→exec)  │
                          └──────┬───────┘
                                 │ on failure
                                 ▼
                          ┌──────────────┐
                          │   backprop   │  appends §B + usually §V
                          └──────────────┘
```

- **`SPEC.md` is the only spec file.** No `docs/` tree, no JSON sidecars.
- **`/sdd:spec` is the only writer.** `/sdd:build` may flip a task status cell (`.` → `x`); everything else routes through `/sdd:spec`.
- **`/sdd:check` writes nothing.** It reports drift and suggests remedies.
- **`/sdd:explain` writes nothing.** It expands a math-glyph citation into prose for humans.

**Backprop** is a skill (not a command) that turns bugs into spec edits. It fires when:

- a test fails during `/sdd:build` verification — `/sdd:build` auto-invokes it before retrying
- `/sdd:check` reports a `VIOLATE` with root cause found
- you tell `/sdd:spec` what happened — post-mortem, prod incident, user report (free-form intent; gate routes to BACKPROP)

Each run appends a `§B` row, usually adds a `§V` invariant + failing test that would have caught the class of bug, then ships the fix in one commit. Next `/sdd:check` enforces the new invariant.

See [`pilot-spec/README.md`](./pilot-spec/README.md) for the full manual.

## PilotPlan — GitHub workflow (`gh` plugin)

GitHub workflow today; planning surface tomorrow. Issues, PRs, commits, merges, releases. PR / issue bodies are written in `steno` style (readable shorthand — no math glyphs). Workflow only — no code development.

```bash
/gh:design <topic>           # propose-then-critique structural design → file issue w/ `design` label
/gh:issue <description>      # file an issue — always-gated by socratic skill (concrete intent: ≤ 1 turn; vague: dialogue until convergence)
/gh:pr <issue# | objective>  # branch off main, push, open PR
/gh:commit [hint]            # commit currently staged changes (Conventional Commits)
/gh:merge [PR#]              # squash-merge with release-note-ready message
/gh:release [bump]           # bump SemVer, tag v<x.y.z> (single-package repos only)
```

`/gh:release` auto-detects `package.json` / `pyproject.toml` / `Cargo.toml` / `.claude-plugin/plugin.json` / `VERSION`. Bails on monorepos by design — local only, no auto-push, no `CHANGELOG.md`.

## Attribution

The `sdd` plugin (directory `pilot-spec/`) is adapted from [**JuliusBrussee/cavekit**](https://github.com/JuliusBrussee/cavekit), MIT-licensed. See [`pilot-spec/README.md`](./pilot-spec/README.md) for full attribution.

## License

MIT. See [`LICENSE`](./LICENSE). Upstream cavekit attribution: [`pilot-spec/README.md`](./pilot-spec/README.md#attribution--license).
