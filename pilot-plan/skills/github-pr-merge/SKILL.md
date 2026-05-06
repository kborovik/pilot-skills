---
name: github-pr-merge
description: Merge GitHub PR into main w/ release-ready commit message. Triggers when user mentions merging PR, landing branch, shipping GitHub PR. Phrasings: "merge this", "land the PR", "merge PR #N".
argument-hint: [PR number]
model: sonnet
allowed-tools: Bash(git *), Bash(gh *)
---

Merge current branch's PR → main w/ release-note-ready commit message.

## Process

1. **Identify PR:**
   - $ARGUMENTS provided → use as PR number
   - Else current branch's PR: `gh pr view --json number,title,body,url`
   - ⊥ PR → inform user ∧ exit

2. **Verify ready to merge:**
   - Remote status:
     - `gh pr checks` — CI status
     - `gh pr view --json mergeable` — merge conflicts
     - `gh pr view --json reviewDecision` — review approval
     - `gh pr view --json isDraft` — draft status
   - ⊥ ready (failing checks, conflicts, missing reviews) → inform user of blockers ∧ exit

3. **Analyze changes:**
   - All commits: `gh pr view --json commits`
   - Files changed: `gh pr diff --name-only` or `git diff main..HEAD --stat`
   - Actual diff: `gh pr diff`
   - Identify scope: bug fix, feature, enhancement, refactor, etc.

4. **Sync PR description → reflect completed work:**
   - Diff current body (§1) vs actual diff ∧ commit history
   - PR description often outdated → reflects original plan, ≠ impl
   - Sync → accurately reflect final state:
     ```bash
     gh pr edit <number> --body "$(cat <<'EOF'
     <updated description>
     EOF
     )"
     ```
   - Keep general structure but ensure:
     - Completed items accurate (≠ aspirational)
     - Removed / abandoned changes ⊥ listed
     - Additional work beyond original scope included
   - Skip if description already matches actual changes

5. **Render release-note-ready commit message:**
   - Title: Conventional Commits from PR title w/ PR number → `type(area): concise imperative description (#42)`
     - **type**: `fix`, `feat`, `refactor`, `chore`, `docs`, `test`
     - **area**: affected module (`gmail`, `missions`, `cli`, `e2e`, `server`, `contacts`, `calendar`, `schema`, `config`, `llm`)
     - Derive from PR title; reuse if already in format
   - Body sections:
     - **Summary**: 2-3 sentence description of what changed
     - **Changes**: bulleted list of key changes
     - **Breaking Changes**: if applicable
   - Format for squash merge

6. **Post insights as PR comment:**
   - Before merge → comment w/ notable insights
   - `gh pr comment <number> --body "$(cat <<'EOF'...EOF)"`
   - Include any of:
     - Code quality / pattern observations in diff
     - Tech debt ∨ risks noticed
     - Follow-up suggestions
     - Edge cases ∨ release considerations
   - Concise ∧ actionable. Skip if ⊥ meaningful insights.

7. **Merge:**
   - Squash + `--delete-branch`:
     ```bash
     gh pr merge <number> --squash --delete-branch --subject "<title>" --body "$(cat <<'EOF'
     <body>
     EOF
     )"
     ```
   - Alternatives:
     - Merge commit: `gh pr merge <number> --merge --delete-branch`
     - Rebase: `gh pr merge <number> --rebase --delete-branch`
   - Repos w/ merge queues → `--auto` queues when checks pass: `gh pr merge <number> --squash --auto --delete-branch`
   - Always pass `--delete-branch` explicitly. ⊥ short form `-d`. ⊥ omit.

8. **Clean up branches:**
   - `--delete-branch` deletes local ∧ remote branches for merged PR
   - Switch to main: `git checkout main`
   - Pull latest: `git pull origin main`
   - Prune stale refs: `git fetch --prune`

9. **Confirm completion:**
   - Show merge commit: `git log -1`
   - Output merged PR URL
   - Display commit message for release notes

## Commit message format

```
feat(server): add user authentication system (#42)

## Summary
Implements JWT-based authentication replacing the session-based system.
Users can now log in and receive tokens that expire after 24 hours.

## Changes
- Add JWT token generation and validation
- Create login and registration endpoints
- Add middleware for protected routes
- Add token refresh endpoint

## Breaking Changes
- Session cookies are no longer supported
- API clients must include Authorization header
```

## Style

Apply `core:steno` skill to:

- Refreshed PR description (§4)
- Squash commit body — Summary / Changes / Breaking sections (§5)
- Insights comment (§6)

Drop articles ∧ filler, use fragments ∧ bullets, preserve identifiers / paths / `#refs` / SHAs verbatim. Conventional Commits prefix (`type(area):`) ∧ `(#number)` suffix fixed → ⊥ compress.

## Requirements

- Squash merge for clean history (unless repo prefers merge commits)
- Conventional Commits w/ PR number: `type(area): description (#number)`
- Commit message ! release-note suitable
- Steno style for body sections (→ `core:steno` skill)
- `--delete-branch` for branch cleanup
- Audit all checks pass before merge
- ⊥ force merge if checks failing
- Draft PRs → confirm w/ user before marking ready
