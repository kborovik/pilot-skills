---
name: github-issue-create
description: Create, file, open, log, ∨ track GitHub issue. Phase 2 of /gh:issue — expects pre-gated input from `core:socratic` skill (convergence criteria already met). Triggers when user mentions GitHub issues, bug reports, feature requests, ∨ wants to track work on GitHub. Phrasings: "file an issue", "open a bug", "track this on GitHub".
argument-hint: <issue description>
model: opus
allowed-tools: Bash(gh *), Read, Grep, Glob
---

GitHub issue. Investigate codebase first for context. Expects pre-gated input from /gh:issue Phase 1 (`core:socratic` skill) — ⊥ re-interrogate convergence criteria already validated upstream.

## Process

1. **Parse $ARGUMENTS**
   - ⊥ description → ask user

2. **Investigate codebase:**
   - Search relevant files via Grep / Glob
   - Read related code → understand current implementation
   - Check git log for recent changes in affected areas
   - Identify affected components / modules

3. **Gather details:**
   - Type: bug, feature, enhancement, refactor
   - Affected files ∧ code paths
   - Bugs → grep error patterns, failing conditions
   - Features → identify where changes needed

4. **Ask clarifying questions if needed:**
   - AskUserQuestion for ambiguous requirements
   - Confirm scope if investigation reveals multiple approaches
   - Bug → ask for repro steps

5. **Generate content:**
   - Title: Conventional Commits → `type(area): concise imperative description`
     - **type**: `fix`, `feat`, `refactor`, `chore`, `docs`, `test`
     - **area**: affected module (`gmail`, `missions`, `cli`, `e2e`, `server`, `contacts`, `calendar`, `schema`, `config`, `llm`)
     - Examples:
       - `fix(gmail): Thread ID divergence breaking outbound assignment routing`
       - `feat(schema): Data deletion policy across schema and delete commands`
       - `refactor(e2e): Redesign fixtures as capability specs`
       - `chore(cli): Remove deprecated debug_modules setting`
   - Body sections:
     - **Summary**: 2-3 sentence description
     - **Context**: Relevant code paths ∧ files from investigation
     - **Proposed Solution** (?): based on codebase analysis
     - **Acceptance Criteria**: clear, testable
     - **Affected Files**: list of files needing changes

6. **Determine labels:**
   - `gh label list` → read existing labels. ⊥ invent new ones
   - Pick every label that genuinely applies. ≥ 1 type label (e.g. `bug`, `enhancement`, `refactor`, `documentation`)
   - Apply area / scope labels if defined (e.g. `gmail`, `cli`, `schema`)
   - ⊥ matching type label → ask user before `gh label create`

7. **Create issue:**
   - `gh issue create --title "..." --label "label1" --label "label2" --body "$(cat <<'EOF'...EOF)"`
   - Each label as separate `--label` flag (or comma-separated in single value)
   - Output URL ∧ number

8. **Post insights as comment:**
   - After create → add comment w/ notable insights
   - `gh issue comment <number> --body "$(cat <<'EOF'...EOF)"`
   - Include any of:
     - Architectural context discovered
     - Related code patterns / dependencies ⊥ obvious
     - Risks ∨ complexity affecting impl
     - Alternative approaches to note
   - Concise ∧ actionable. Skip if ⊥ meaningful insights.

## Style

Apply `core:steno` skill to issue title ∧ body. Drop articles ∧ filler, use fragments ∧ bullets, preserve identifiers / paths / `#refs` verbatim. Conventional Commits prefix (`type(area):`) fixed → ⊥ compress.

## Requirements

- Investigate before asking — gather context first
- Keep investigation focused on description
- Steno style for body (→ `core:steno` skill)
- Markdown formatting in body
- Cite specific files / line numbers when relevant
