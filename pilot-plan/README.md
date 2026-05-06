<h1 align="center">PilotPlan</h1>

<p align="center">
  <strong>GitHub workflow for Claude Code</strong><br/>
  <sub>issues · PRs · commits · merges · releases</sub><br/>
  <sub><em>installed as the <code>gh</code> plugin · slash commands <code>/gh:*</code></em></sub>
</p>

---

## Table of contents

1. [What this is](#what-this-is)
2. [Install](#install)
3. [Commands](#commands)
   - [`/gh:design`](#ghdesign--propose-then-critique-design-loop)
   - [`/gh:issue`](#ghissue--file-an-issue-gated)
   - [`/gh:pr`](#ghpr--open-a-pull-request)
   - [`/gh:commit`](#ghcommit--commit-staged-changes)
   - [`/gh:merge`](#ghmerge--squash-merge-a-pr)
   - [`/gh:release`](#ghrelease--cut-a-semver-release)
4. [Skills](#skills)
5. [`steno` — GitHub-facing shorthand](#steno--github-facing-shorthand)
6. [The socratic gate](#the-socratic-gate)
7. [Design vs issue](#design-vs-issue)
8. [Non-goals](#non-goals)
9. [Files](#files)
10. [License](#license)

---

## What this is

`pilot-plan` ships the `gh` plugin — six slash commands that drive day-to-day GitHub work from inside Claude Code. The brand name **PilotPlan** signals where this plugin is heading: today the verbs are GitHub-shipping (issue, PR, commit, merge, release); the planning surface (triage, milestones, roadmap) lands later.

Two design choices set PilotPlan apart from a thin `gh` CLI wrapper:

- **Always-gated issues.** `/gh:issue` runs an internal **socratic** loop before `gh issue create`. Concrete intent passes the gate in ≤ 1 turn; vague intent triggers single-question dialogue until validity criteria converge. Issues are team-shared artifacts — the cognitive bar is enforced at filing time, not optional.
- **Steno output.** Every issue body, PR description, commit body, and release note is written in `steno` style — readable shorthand (`→ & | §`), no math glyphs. Reviewers don't slow down on `∀ ∃ ∴ ⊥`.

Workflow-only. PilotPlan does **not** write or refactor code — that boundary is deliberate. Use [`pilot-spec`](../pilot-spec) (the `sdd` plugin) for the implementation loop.

## Install

```bash
/plugin marketplace add kborovik/pilot-skills
/plugin install gh@pilot-skills
```

Then in any repo:

```bash
/gh:issue   # file your first issue (socratic gate engages on vague intent)
```

## Commands

### `/gh:design` — propose-then-critique design loop

Use when there's a structural choice to weigh — tradeoffs, named alternatives, subsystem shape. The model proposes a shape, you critique, the loop converges only when an explicit `## Open Questions` list is empty. Files a new issue with the `design` label (creating the label if missing).

```bash
/gh:design how should the release pipeline split monorepo plugins?
```

Distinct from `/gh:issue`: socratic converges on **enough** (sharpen vague intent for any issue); design converges on **exhausted** (every open structural question has a recorded decision).

### `/gh:issue` — file an issue (gated)

Two-phase. Phase 1 runs the **socratic** gate (always-on, no bypass). Phase 2 calls `gh issue create`.

| convergence criterion          | what counts                                                          |
| ------------------------------ | -------------------------------------------------------------------- |
| explicit goal                  | observable symptom (bug) ∨ concrete behavior delta (feature)         |
| acceptance criterion ∨ repro   | one verifiable outcome ∨ reproducible failure                        |
| §V/§T citation ∨ ⊥-spec flag   | cite the invariant/task this concerns, ∨ flag explicitly out-of-spec |

Concrete first-turn input passes the gate in one turn. Vague input loops until criteria converge. No `--quick` bypass, no opt-out.

```bash
/gh:issue webhook handler retried POSTs after 5xx → double-charged 11 customers; out-of-spec
/gh:issue "something feels slow"   # triggers dialogue
```

### `/gh:pr` — open a pull request

Branches off `main`, pushes, opens a PR. Accepts an issue number (`/gh:pr 42`) or a free-form objective.

```bash
/gh:pr 42                              # work the issue → branch → push → open PR
/gh:pr add retry-after parsing to webhook client
```

PR title uses Conventional Commits. PR body is steno.

### `/gh:commit` — commit staged changes

Commits whatever is currently staged with a Conventional Commits message that matches repo history. Optional hint sharpens the subject.

```bash
/gh:commit                              # infer message from diff + repo style
/gh:commit fix off-by-one in retry loop
```

### `/gh:merge` — squash-merge a PR

Squash-merges a PR with a release-note-ready message. Defaults to the current branch's PR.

```bash
/gh:merge          # current branch
/gh:merge 87       # specific PR
```

### `/gh:release` — cut a SemVer release

Single-package only. Auto-detects the version manifest (`package.json` / `pyproject.toml` / `Cargo.toml` / `.claude-plugin/plugin.json` / `VERSION`), bumps, commits, creates an annotated tag `v<x.y.z>`. **Local only** — no `git push`, no `gh release create`, no `CHANGELOG.md` generation.

```bash
/gh:release             # auto-detect bump from Conventional Commits
/gh:release patch       # explicit
/gh:release 2.4.0       # pinned (downgrade allowed for redo)
/gh:release retag-baseline   # recovery for a bad first tag
```

Bails on monorepos by design. For multi-plugin repos, write a repo-local orchestrator (this repo's `.claude/commands/release.md` is the reference pattern).

## Skills

Slash commands are entry points. Skills under `skills/<name>/SKILL.md` carry the actual behavior so they can be reused outside the slash-command context.

| skill                  | location     | role                                                                          |
| ---------------------- | ------------ | ----------------------------------------------------------------------------- |
| `core:socratic`        | core         | internal gate of `/gh:issue` — single-question loop + just-in-time teaching   |
| `core:steno`           | core         | human-facing shorthand encoder for issue/PR/commit bodies                     |
| `design`               | gh           | propose-then-critique structural design loop, files issue w/ `design` label   |
| `github-issue-create`  | gh           | files the gated issue (`/gh:issue` Phase 2)                                   |
| `github-pr-create`     | gh           | branches off main, pushes, opens PR                                           |
| `github-pr-merge`      | gh           | squash-merges with release-note-ready message                                 |
| `github-commit-staged` | gh           | commits staged changes with Conventional Commits message                      |
| `github-release`       | gh           | single-package SemVer bump + annotated tag                                    |

`core:socratic` and `core:steno` are shared encoder/protocol skills — they live in the `core` plugin (directory `pilot-core/`, auto-installed via `dependencies` field) and are invoked by `gh` cmds via namespaced ref. Same skills also available to `sdd` and any future plugin that depends on `core`. Socratic is internal — it runs inside `/gh:issue`, not as a user-invoked surface.

## `steno` — GitHub-facing shorthand

Every artifact PilotPlan creates on GitHub uses **steno** encoding: drop articles and filler, fragments where they read cleanly, readable symbols only (`→ & | §`). No heavy math glyphs (`∀ ∃ ∴ ⊥ ∈ ∉`) — those slow reviewers down.

**Bad** (prose):

> The release script was force-pushing tags without verifying the manifest version had been bumped, which caused two production tags to point at the same commit.

**Good** (steno):

> Release script force-pushed tags w/o checking manifest bump → 2 prod tags @ same commit. Fix: gate push on `git diff --name-only HEAD~1 -- <manifest>` non-empty.

Steno lives in the [`core` plugin](../pilot-core/skills/steno) and is invoked from `gh` cmds via `core:steno`. Distinct from the `glyph` skill in the [`sdd` plugin](../pilot-spec/skills/glyph). `glyph` writes math-glyph encoding for `SPEC.md` — the LLM is the audience. `steno` writes shorthand for human reviewers on GitHub. The two are deliberately not merged.

## The socratic gate

`/gh:issue` is **always-gated**. The rationale:

- **GitHub issues are team-shared artifacts.** Validity bar > author keystroke savings.
- **Concrete authors pay nothing.** Fast-path passes prepared input in one turn.
- **Vague authors get help.** Single-question loop with a teaching overlay surfaces distinctions the framing missed (e.g. "is this a §V violation, a missing §T, or out-of-spec?").
- **No bypass flag.** A `--quick` bypass would erode the bar exactly when authors are tired or distracted — which is when the bar matters most.

Escape hatch: if you say "just file it", the gate stops asking but does **not** disappear. Unmet criteria become explicit `## Unresolved` callouts in the issue body — gaps stay visible to the team.

## Design vs issue

| skill      | converges on  | use when                                                          |
| ---------- | ------------- | ----------------------------------------------------------------- |
| `socratic` | **enough**    | intent is vague — sharpen it into a fileable issue                |
| `design`   | **exhausted** | structural choice with tradeoffs — exhaust open questions first   |

Use `/gh:issue` for bugs, feature requests, and concrete tasks. Use `/gh:design` when you'd otherwise file an issue titled "how should we structure X?" — the design loop forces the tradeoffs into `## Design decisions` and `## Open Questions` sections before the issue exists.

## Non-goals

- **No code development.** `/gh:pr` opens a PR; it does not refactor, simplify, or review the diff. Use `/sdd:build` for code work.
- **No `git push` in release.** Local tag only — you push manually after inspection.
- **No `CHANGELOG.md` generation.** Conventional Commits + annotated tag bodies are the changelog.
- **No GitHub Actions wiring.** Plugin runs from your machine; CI is your repo's concern.
- **No `--quick` flag on `/gh:issue`.** The gate is unconditional by design.
- **No comment-on-existing for `/gh:design`.** Design always files a new issue.

## Files

```
.claude-plugin/plugin.json     plugin manifest (name: gh, version, author, dependencies: ["core"])
commands/issue.md              /gh:issue entry point
commands/design.md             /gh:design entry point
commands/pr.md                 /gh:pr entry point
commands/commit.md             /gh:commit entry point
commands/merge.md              /gh:merge entry point
commands/release.md            /gh:release entry point
skills/design/                 propose-then-critique loop
skills/github-issue-create/    files the gated issue
skills/github-pr-create/       opens a PR
skills/github-pr-merge/        squash-merges a PR
skills/github-commit-staged/   commits staged changes
skills/github-release/         single-package SemVer release

# socratic ∧ steno live in ../pilot-core/skills/ — invoked here via core:socratic / core:steno
```

## License

MIT. See the repo-root [`LICENSE`](../LICENSE).
