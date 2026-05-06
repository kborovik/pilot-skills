---
name: github-pr-create
description: Create GitHub PR from issue number ∨ objective. Triggers when user mentions GitHub PRs, pull requests, opening PR. Phrasings: "open a PR", "create a pull request", "PR for issue #N".
argument-hint: [issue number or PR objective]
model: opus
allowed-tools: Bash(git *), Bash(gh *)
---

PR from issue number ∨ free-form objective. GitHub workflow only — ⊥ implement, refine, review, verify code.

## Process

1. **Determine input from $ARGUMENTS:**
   - ⊥ argument → AskUserQuestion for issue number
   - Number → issue number (§2a)
   - Text → free-form objective (§2b)

2a. **From issue number — fetch:**

- `gh issue view <number>`
- Extract title, body, labels
- PR title: Conventional Commits matching issue → `type(area): concise imperative description`
  - **type**: `fix`, `feat`, `refactor`, `chore`, `docs`, `test`
  - **area**: affected module
  - Derive from issue title; reuse directly if already in format
- Body: ! include `Resolves #<number>` → auto-close issue
- Branch: `<issue-number>-<slugified-title>` (e.g. `42-add-rate-limiting`)
- Slugify: lowercase, spaces → hyphens, strip special chars, ≤ 50 chars
- Create branch ∧ PR:
  - From main: `git checkout main && git pull origin main`
  - Branch: `git switch --create <issue-number>-<slugified-title>`
  - Empty commit: `git commit --allow-empty --message "wip: <issue-title> (#<issue-number>)"`
  - Push: `git push --set-upstream origin <branch-name>`
  - Regular PR: `gh pr create --title "..." --body "$(cat <<'EOF'...EOF)"`
  - Output URL

2b. **From free-form objective:**

- Generate Conventional Commits title → `type(area): concise imperative description`
  - **type**: `fix`, `feat`, `refactor`, `chore`, `docs`, `test`
  - **area**: affected module
- Slugify: lowercase, spaces → hyphens, strip special chars, ≤ 50 chars
- Create branch ∧ PR:
  - From main: `git checkout main && git pull origin main`
  - Branch (temp name): `git switch --create <slugified-title>`
  - Empty commit: `git commit --allow-empty --message "wip: <PR title>"`
  - Push: `git push --set-upstream origin <slugified-title>`
  - Regular PR: `gh pr create --title "..." --body "$(cat <<'EOF'...EOF)"`
  - Extract PR number from output URL

3. **Post insights as PR comment (?):**
   - `gh pr comment <number> --body "$(cat <<'EOF'...EOF)"`
   - Include any of:
     - Architectural observations ∨ design decisions to flag
     - Risks ∨ future-attention areas
     - Alternatives considered ∧ why rejected
   - Concise ∧ actionable. Skip if ⊥ meaningful insights.

## Style

Apply `core:steno` skill to PR title ∧ body. Drop articles ∧ filler, use fragments ∧ bullets, preserve identifiers / paths / `#refs` / `Resolves #N` verbatim. Conventional Commits prefix (`type(area):`) fixed → ⊥ compress.

## Requirements

- Always regular PR (⊥ `--draft`)
- Branch names concise but descriptive
- From issue → ! include `Resolves #<number>` in body
- From objective → body ! contain enough context for reviewers
- Steno style for body (→ `core:steno` skill)
- Skill stops at PR creation. Implementation, simplification, review, CI verification ∉ scope.
