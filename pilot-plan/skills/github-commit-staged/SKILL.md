---
name: github-commit-staged
description: Commit staged git changes w/ descriptive Conventional Commits message. Triggers when user asks to commit, save changes, ∨ mentions GitHub commits. Phrasings: "commit", "save my changes", "commit this".
argument-hint: [commit message hint]
model: sonnet
allowed-tools: Bash(git *), Bash(git status)
---

Commit staged changes w/ well-crafted message.

## Process

1. **Check staged changes:**
   - `git diff --cached --stat` → see what staged
   - ⊥ staged → inform user ∧ exit
   - $ARGUMENTS provided → use as hint

2. **Analyze changes:**
   - `git diff --staged` → review
   - Identify added / modified / removed
   - Understand purpose ∧ scope

3. **Check repo style:**
   - `git log --oneline --max-count=10` → recent patterns
   - Match existing style (prefixes, capitalization, format)

4. **Generate message:**
   - Title: Conventional Commits → `type(area): concise imperative description`
     - **type**: `fix`, `feat`, `refactor`, `chore`, `docs`, `test`
     - **area**: affected module (`gmail`, `missions`, `cli`, `e2e`, `server`, `contacts`, `calendar`, `schema`, `config`, `llm`)
   - Title ≤ 72 chars
   - Complex changes → body after blank line

5. **Create commit:**
   - Heredoc → preserves formatting:

     ```
     git commit --message "$(cat <<'EOF'
     <title>

     <optional body>
     EOF
     )"
     ```

6. **Verify:**
   - `git status` → confirm clean state
   - `git log --max-count=1` → show created commit

## Commit message format

Simple:

```
fix(server): add validation for email input
```

With body:

```
refactor(server): replace session-based auth with JWT tokens

- Replace session-based auth with JWT
- Add token refresh endpoint
- Update middleware to validate tokens
- Add tests for token expiration
```

## Requirements

- Conventional Commits: `type(area): imperative description`
- Title completes: "This commit will..."
- Only commit what's staged (⊥ stage more)
- ⊥ push unless explicitly requested
- Match repo style if possible
- Cite issue numbers if relevant (e.g. `fix login bug (#42)`)
