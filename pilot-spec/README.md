<h1 align="center">PilotSpec</h1>

<p align="center">
  <strong>compressed spec-driven development for Claude Code</strong><br/>
  <sub>one file · four commands · zero sub-agents</sub><br/>
  <sub><em>installed as the <code>sdd</code> plugin · slash commands <code>/sdd:*</code></em></sub>
</p>

---

## Table of contents

1. [What this is](#what-this-is)
2. [Why a spec at all](#why-a-spec-at-all)
3. [Install](#install)
4. [The mental model](#the-mental-model)
5. [SPEC.md format](#specmd-format)
6. [Commands](#commands)
   - [`/sdd:spec`](#sddspec--mutate-the-spec)
   - [`/sdd:build`](#sddbuild--plan-then-execute)
   - [`/sdd:check`](#sddcheck--drift-report)
   - [`/sdd:explain`](#sddexplain--math-glyph--prose)
7. [Skills](#skills)
8. [Workflows](#workflows)
9. [Ambiguity policy](#ambiguity-policy)
10. [Math-glyph encoding](#math-glyph-encoding)
11. [Backprop in detail](#backprop-in-detail)
12. [Position in the SDD landscape](#position-in-the-sdd-landscape)
13. [FAQ](#faq)
14. [Non-goals](#non-goals)
15. [Files](#files)
16. [Attribution & license](#attribution--license)

---

## What this is

Plan-then-execute forgets. SDD remembers — but most spec-driven frameworks bury that
value under agent swarms, dashboards, and ceremony that costs more tokens than they save.

`pilot-spec` keeps only what earns its place:

- **durable spec** — `SPEC.md` at repo root survives context resets.
- **math-glyph encoding** — ~75% fewer tokens than prose. Math operators (∀ ∃ ∴ ⊥ ≤ ≥ ≠ ∈ ∉ ∧ ∨ → §), fragments, pipe tables for repeating records. Distinct from `steno` (the `core` plugin's human-facing shorthand, which deliberately avoids math glyphs).
- **backprop reflex** — every test failure becomes a `§B` entry; classes of bug become `§V` invariants the spec never forgets.
- **native loop** — main Claude does the work. No sub-agents, no orchestrator, no parallel workers.

> The spec is the only artifact that earns its tokens. Everything else that costs tokens must either save more tokens later, save the user's attention, or it gets cut.

### Who reads SPEC.md

`SPEC.md` is an **LLM-facing artifact**. You operate it through Claude — `/sdd:spec` writes, `/sdd:build` and `/sdd:check` read, `/sdd:explain` decodes a citation back into prose when you want to read along. The expected loop is `human → /sdd:* → Claude → SPEC.md`, not hand-editing the file in your editor.

That framing is load-bearing. Every design choice that looks user-hostile on first read — math glyphs over English, fragments over sentences, pipe tables over bulleted lists, `:line` citations dropped because they rot — is optimized for the model that re-parses the spec on every command, not for the human who only sees `/sdd:explain` output. If you find yourself reaching for the spec to skim it as a human, that's what `/sdd:explain` is for.

## Why a spec at all

Long sessions drift. Plans get forgotten between context resets. Decisions made in
turn 3 get re-litigated in turn 30. Tests pass but the system was never specified, so
the next agent breaks the same thing again.

A spec is the cheapest way to make a long-running project agent-coherent:

- **Persistent context.** Re-read at the start of each command. Survives `/clear`.
- **Addressable.** `§V.3` is a citation. Code, tests, and commit messages can point at it. Reviewers don't have to guess what changed.
- **Testable invariants.** `§V` rows are written so a failing test maps 1-to-1 to a violated invariant.
- **Bug memory.** `§B` table makes it impossible to forget that a class of bug already happened. Backprop usually adds the invariant and test that would have caught it.

The trade-off is one extra file in the repo and a small ceremony before each task.
For anything beyond a one-shot script, that ceremony pays for itself by the third
context reset.

## Install

```bash
/plugin marketplace add kborovik/pilot-skills
/plugin install sdd@pilot-skills
```

Then in any repo:

```bash
/sdd:spec   # creates SPEC.md if missing
```

## The mental model

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
- **`/sdd:spec` is the only writer.** `/sdd:build` may flip a task status cell (`.` → `~` → `x`); everything else routes through `/sdd:spec`.
- **`/sdd:check` writes nothing.** It reports drift and suggests remedies.
- **`/sdd:explain` writes nothing.** It expands a math-glyph citation into prose for humans.

## SPEC.md format

Six fixed sections, fixed order. Each row is addressable as `§<S>.<n>`.

```markdown
# SPEC

## §G GOAL

one line. what code must do.

## §C CONSTRAINTS

- non-negotiable boundary
- tech / language / library locked in

## §I INTERFACES

external surface — what the world sees.

- cmd: `foo bar` → stdout JSON
- api: POST /x → 200 {id}
- file: `config.yaml` schema …
- env: `FOO_KEY` required

## §V INVARIANTS

numbered. testable. each ! MUST hold.
V1: ∀ req → auth check before handler
V2: token expiry ≤ ⊥ allowed
V3: DB write ! in transaction

## §T TASKS

id|status|task|cites
T1|.|scaffold repo|-
T2|.|impl §I.api POST /x|V2
T3|x|add §V.1 middleware|V1,I.api

## §B BUGS

id|date|cause|fix
B1|2026-04-20|token `<` not `≤`|V2
B2|2026-04-21|race on write|V3
```

**Status glyphs in §T:** `.` todo · `~` wip · `x` done.
**Cell rules:** literal `|` → escape as `\|`. Empty cell = `-`. Backticks OK.

## Commands

### `/sdd:spec` — mutate the spec

The sole mutator. The argument is **free-form intent** — a socratic gate (the `core:socratic` skill) reads what you wrote and emits a mode post-convergence. You don't pick the mode by typing a prefix.

| project state    | possible modes                                                       | gate behavior                                                                                                              |
| ---------------- | -------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| no `SPEC.md`     | **NEW** ∨ **DISTILL**                                                | concrete intent passes ≤ 1 turn; vague intent triggers single-question dialogue to convergence                             |
| `SPEC.md` exists | **BACKPROP** ∨ **AMEND** ∨ **NEW** (rare, requires explicit re-init) | mode emerges from convergence triple — symptom + surface + recurrence-class for BACKPROP, §-target + delta for AMEND       |

Examples (all free-form intent — gate classifies):

```bash
/sdd:spec a CLI that ingests JSON over stdin and emits Parquet
/sdd:spec build the spec from this codebase
/sdd:spec V2's `≤` should be `<` for unsigned tokens
/sdd:spec rate-limiter dropped requests under 100rps
```

### `/sdd:build` — plan then execute

Native single-thread plan → execute → verify loop. No sub-agents.

| arg form | scope                                      |
| -------- | ------------------------------------------ |
| `§T.n`   | implement that one task                    |
| `--next` | lowest-numbered row with status `.` or `~` |
| `--all`  | every `.` row in §T order                  |
| (empty)  | same as `--all`                            |

Loop per task:

1. **PLAN** — cite every §V / §I the task touches. If a gap is found ("no §V applies", "§I silent on shape X"), pause and ask the user to route it back to `/sdd:spec`. Plan never invents rules.
2. **EDIT** — flip status `.` → `~`, make the change, run tests / build.
3. **VERIFY** — on failure, classify:
   - **(a) code bug** → fix and re-run.
   - **(b) spec wrong** or **(c) unspecified edge case** → invoke `backprop` (via `/sdd:spec <cause>` — gate classifies as BACKPROP), let it append `§B` and usually a new `§V`, resume against the updated spec.
4. **CLOSE** — flip `~` → `x` only when verification is green.

`/sdd:build` never silently retries. Every failure is either fixed or specced.

### `/sdd:check` — drift report

Read-only diagnostic. Diffs `SPEC.md` against the working tree.

| arg     | what it audits                                    |
| ------- | ------------------------------------------------- |
| `§V`    | invariants — does the code still uphold each one? |
| `§I`    | interfaces — do public surfaces match §I shape?   |
| `§T`    | task status — `x` rows that look unfinished, etc. |
| `--all` | all three (default)                               |

Output groups violations by severity (`VIOLATE` / `RISK` / `STALE`) and suggests a
remedy: usually `/sdd:spec <free-form intent>` (gate routes to AMEND ∨ BACKPROP) or `/sdd:build`. It never
runs them itself.

### `/sdd:explain` — math-glyph → prose

The inverse of the `glyph` skill (which writes math-glyph encoding). Given any citation, returns plain English with cited context.

```bash
/sdd:explain §V.3      # expand invariant 3
/sdd:explain §T.7      # expand task 7 + every §V/§I it cites
/sdd:explain §B.2      # expand bug 2 + the invariant that catches recurrence
/sdd:explain --next    # expand the next unfinished task
```

Useful for code review, onboarding, or when you'd otherwise have to translate
`∀ req → auth check before handler` in your head.

## Skills

`commands/*.md` are the entry points. Skills under `skills/<name>/SKILL.md` carry
the actual behavior so they can be reused outside the slash-command context.

| skill      | role                                                                 |
| ---------- | -------------------------------------------------------------------- |
| `spec`     | mirrors `/sdd:spec` — sole mutator                                   |
| `build`    | mirrors `/sdd:build` — plan → execute loop                           |
| `check`    | mirrors `/sdd:check` — drift report                                  |
| `glyph`    | LLM-facing math-glyph encoder for spec writes (~75% token reduction) |
| `backprop` | bug → spec protocol; six steps, invoked on every failed verification |

You don't usually invoke skills directly — Claude picks them up from the command
flow. The exception is `backprop`, which fires automatically on test/build failure
inside `/sdd:build`.

## Workflows

### Greenfield — new project

```bash
/sdd:spec build a static-site generator that converts a Markdown directory into a single-page HTML bundle
# review §G/§C/§I/§V in SPEC.md, amend if needed
/sdd:build --next   # plan, implement, verify T1 (scaffold)
/sdd:build --next   # T2 (renderer)
/sdd:check          # before opening a PR
```

### Brownfield — existing repo

```bash
/sdd:spec build the spec from this codebase   # gate routes to DISTILL
/sdd:check                     # see what already drifts from the distilled spec
/sdd:spec V2's bound is too loose for the rate-limiter   # gate routes to AMEND
/sdd:build §T.3                # tackle a specific task
```

### A bug just hit production

```bash
/sdd:spec webhook handler retried POSTs after 5xx, double-charged 11 customers
# backprop appends §B, adds §V invariant "POST handler ! idempotent on retry",
# adds a failing test, then commits the fix
/sdd:check                     # confirm new §V is now upheld
```

### Pre-merge sanity

```bash
/sdd:check --all
/sdd:explain §V.4              # if a violation is unclear, decompress it
```

## Ambiguity policy

`/sdd:build` treats ambiguity as a **spec defect**, not a coding judgement call. It
surfaces in three places:

1. **Pre-execute plan gate.** PLAN cites every §V / §I the task touches before any
   edit. Gaps surface as "no §V applies" or "§I silent on shape X". User reviews;
   unresolved gaps route back to `/sdd:spec` to add the missing rule.
2. **Mid-execute failure triage.** On verification fail, build classifies:
   (a) code bug — fix, no spec change; (b) spec wrong; (c) unspecified edge case →
   invoke `backprop`, let it append §B and usually a new §V, resume against updated
   spec. No silent retries.
3. **Write-policy fence.** `/sdd:build` only flips §T status (`.` → `~` → `x`). All
   other `SPEC.md` edits go through `/sdd:spec`. Build cannot resolve ambiguity by
   quietly redefining the rules.

Net: ambiguity is a routing decision, not a creative one — bounce the question back
to the spec layer, either pre-execute via the plan gate or post-failure via backprop.

## Math-glyph encoding

The `glyph` skill writes **math-glyph encoding** — a compression scheme built around math operators (∀ ∃ ∴ ⊥ ∈ ∉ ≤ ≥ ≠ ∧ ∨ → §) plus fragments and pipe tables. "Math-glyph" is the precise term: generic glyphs are any printable symbols (the §T status markers `.` `~` `x`, bullets, dingbats); the encoding here specifically leans on math-glyphs because they carry truth-conditional meaning in one character.

Result: every spec write lands at roughly a quarter of its prose length while staying machine- and human-readable. The companion `steno` skill (`core` plugin, invoked from `gh` cmds) handles GitHub-facing text and deliberately omits math-glyphs so reviewers don't slow down.

Rules:

- Drop articles (a/an/the). Drop filler. Drop aux verbs where a fragment works.
- Short synonyms (`fix` over `implement`).
- **Preserve verbatim:** code, paths, identifiers, URLs, numbers, error strings, SQL, regex.

Symbol set:

```
→   leads to / becomes / triggers
∴   therefore / fix
∀   for all / every
∃   exists / some
!   must
?   may / optional
⊥   never / impossible / forbidden
≠   not equal / differs from
∈   in / member of
∉   not in
≤   at most
≥   at least
∧   and
∨   or
```

**Bad** (prose):

> The authentication middleware must verify the token expiry on every request before allowing the handler to execute.

**Good** (math-glyph):

> V1: ∀ req → auth check before handler

**Bad** (prose bug note):

> Fixed a bug where token expiry comparison used strict less-than instead of less-than-or-equal, causing tokens to be rejected exactly at their expiry timestamp.

**Good** (math-glyph):

> B1: token `<` not `≤` ∴ tokens rejected @ expiry. §V.2 now ! `≤`.

If math-glyphs slow you down on review, `/sdd:explain §V.1` decompresses on demand.

## Backprop in detail

Backprop is the one non-obvious thing SDD does that vanilla plan-then-execute
doesn't. It runs in six steps:

1. **Capture** — record the failing case verbatim (test name, error, stack, repro).
2. **Trace** — find the cause: code bug, spec wrong, or unspecified edge case.
3. **Append §B** — add a row: `id|date|cause|fix`. Math-glyph-encoded.
4. **Decide on §V** — would an invariant have caught the _class_ of this bug? If
   yes, add or tighten one. Cite it from the new §B row.
5. **Failing test first** — write a test that would have caught the class. Watch it
   fail, then ship the fix. The test stays in the repo as a permanent guard.
6. **Commit once** — single commit: spec edit + test + fix. PR description points at
   the new §B and §V.

Triggers:

- Test/build fails inside `/sdd:build` verification.
- `/sdd:check` reports a `VIOLATE` whose root cause has been identified.
- User: `/sdd:spec <description>` — post-mortems, prod incidents, user reports; gate routes to BACKPROP on bug-class intent.

## Position in the SDD landscape

The SDD ecosystem has 15+ active frameworks as of 2026 — from [Superpowers](https://github.com/obra/superpowers) (166k stars) to MUSUBI (28). They split along two axes: **rigor** (how heavyweight is the spec?) and **orchestration** (how many agents do the work?).

```
cavekit ── PilotSpec (this repo) ── spec-coding-mcp ── spec-kit ── Tessl ── Kiro ── Superpowers ── BMAD ── MUSUBI
[lightweight ────────────────────────────────────────────────────────────────────────────────── full orchestration]
```

PilotSpec sits at the lightweight extreme: fewer commands than spec-kit (4 vs ~8), one file vs 3–8+, no IDE, no sub-agents, no orchestration server.

### vs. Superpowers

[Superpowers](https://github.com/obra/superpowers) is the most-starred SDD project on GitHub. The design contrast is sharp:

| axis                | Superpowers                               | PilotSpec                                                 |
| ------------------- | ----------------------------------------- | --------------------------------------------------------- |
| **execution model** | sub-agents, wave-based parallel execution | single-thread main Claude, one diff at a time             |
| **spec artifact**   | prose skill files + HARD-GATE prompts     | one `SPEC.md` in math-glyph (∀ ∃ ∴ ⊥ § + pipe tables)     |
| **token economy**   | full prose                                | ~75% reduction vs prose                                   |
| **bug feedback**    | TDD discipline enforced via gates         | `backprop` reflex: failure → §B row → §V invariant + test |
| **optimizes for**   | generation throughput (ship faster)       | review throughput (audit faster)                          |

Superpowers' bet is that more agents in parallel = more leverage. PilotSpec's bet is that **the bottleneck is human reasoning, not LLM throughput** — sub-agents race on shared files, fan out context, and produce diffs that are hard to predict or audit. A single execution stream is reproducible (same spec + same task → same plan), cheaper (one context, not N), and reviewable. Different design centers, not just different sizes.

### vs. spec-kit / Kiro / Tessl

Per [Fowler's SDD survey](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html), most frameworks are **spec-first** (spec is throwaway after the task). PilotSpec is **spec-anchored** by design — `SPEC.md` is durable, evolves task by task, and `/sdd:check` reconciles drift. Tessl is the only mainstream framework with the same anchored stance; spec-kit and Kiro both create per-task artifacts.

PilotSpec also writes its spec in math-glyph encoding rather than prose markdown. Spec-kit, Kiro, BMAD, MUSUBI all spread the spec across multiple prose files; cavekit and PilotSpec are the only ones combining **one file + heavy compression**, and PilotSpec is the only one using _math glyphs_ (∀ ∃ ∴ ⊥) over English-fragment shorthand.

## FAQ

**Why not YAML / JSON for the spec?**
Markdown + pipe tables grep cleanly, diff cleanly, render in every PR review tool,
and don't trip on quoting. JSON specs invite tooling that defeats the point — the
spec is meant to be read by humans and one LLM, not parsed by a build system.

**Why one file?**
Sub-1000-line specs fit in context cheaply. Multi-file specs invite cross-file
inconsistency and force `grep` ceremony. If `SPEC.md` exceeds 500 lines, compact §B
(drop oldest bugs) before splitting.

**What if I want sub-agents / parallel workers / a dashboard?**
Deliberately omitted. Sub-agents fan out context, race on the same files, and produce diffs that are hard to predict or review. A single execution stream is faster end-to-end (no orchestration overhead, no merge reconciliation), cheaper (one context window, not N), and reproducible — the same spec and the same task yield the same plan. Predictability and speed beat parallelism when **the bottleneck is human reasoning**, not LLM throughput.

**Does `/sdd:build` always backprop on failure?**
On verification failure that is _not_ a clear code bug, yes. Pure code bugs (typo,
wrong loop bound) get fixed without a spec change. Anything that smells like
under-specification or "we never said what should happen here" routes through
`backprop`.

**Can I skip math-glyph encoding and write prose specs?**
Yes, but the next time the spec is loaded into context you'll pay 4× the tokens for
the same content. The encoding is optional in syntax but expensive to skip in practice.

## Non-goals

- no sub-agents — main Claude does the work
- no dashboards — `cat SPEC.md` is the dashboard
- no parallel workers — one thread, one spec, one diff
- no JSON / YAML spec bodies — Markdown + pipe tables
- no hooks, no orchestration binaries, no TypeScript helpers

## Files

```
commands/spec.md           /sdd:spec entry point
commands/build.md          /sdd:build entry point
commands/check.md          /sdd:check entry point
commands/explain.md        /sdd:explain entry point
skills/spec/               spec mutator (mirrors commands/spec.md)
skills/build/              plan-execute skill (mirrors commands/build.md)
skills/check/              drift report skill (mirrors commands/check.md)
skills/glyph/              LLM-facing math-glyph encoder
skills/backprop/           bug → spec protocol (six steps)
```

## Attribution & license

`pilot-spec` is adapted from [**JuliusBrussee/cavekit**](https://github.com/JuliusBrussee/cavekit)
(v4.0.0, MIT-licensed).

For the original project, history, and `v3.1.0` (the full Hunt lifecycle with
sub-agents, parallel workers, and design-system enforcement), see the upstream repo.

MIT. See [`LICENSE`](./LICENSE).
