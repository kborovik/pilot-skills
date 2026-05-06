---
name: steno
description: |
  Steno encoding — human-facing compression for GitHub issue bodies, PR
  descriptions, PR/release commit message bodies, insights comments. Cuts
  filler, keeps facts. Readable symbols (→ & | §-cites), ⊥ heavy math glyphs
  (∀ ∃ ∴ ⊥ ∈ ∉) → reviewers ⊥ slow down. Triggers: write/edit GitHub issues,
  PRs, insights comments in gh plugin; ∨ user says "steno", "shorthand",
  "tighten this", "make this shorter".
---

# steno — human-facing terse text for GitHub issues ∧ PRs

Audience: reviewer reading many issues ∧ PRs, wants facts fast — still human, ≠ token-optimised model. Readable symbols ∧ plain words; math glyphs → `glyph` skill ∈ `sdd` plugin.

## SCOPE

Applies to:

- GitHub issue titles ∧ bodies.
- PR titles ∧ bodies (incl. PR desc refresh on merge).
- PR squash/merge commit message bodies (release-note section).
- Issue / PR insights comments posted by gh skills.

⊥ apply to:

- Code, snippets, backticked text.
- Conventional Commits title prefix (`type(area):`) — fixed format.
- Error strings, log lines.
- External-facing copy (marketing, landing pages).

## GRAMMAR

- Drop articles (a, an, the) when sentence still parses.
- Drop filler: just, really, basically, simply, actually, very, quite.
- Drop aux verbs where fragment works: is, are, was, were, being.
- Drop hedging: might, perhaps, could be worth, I think, it seems.
- Drop pleasantries ∧ meta ("in this section we will…").
- Fragments fine. Lists > paragraphs.
- Short synonyms: fix > implement, run > execute, use > utilize, help > facilitate, big > extensive, add > introduce, drop > remove, show > demonstrate, start > initiate, end > terminate, get > retrieve, set > configure, ask > request.
- One idea per line in lists. Break long sentences.

## SYMBOLS

Safe for GitHub readers:

```
→   leads to / becomes / produces
&   and
|   or (in lists, not prose)
≤   at most
≥   at least
≠   not equal
§   spec citation (e.g. `§V.3`, `§T.11`) — only for refs into SPEC.md
```

Avoid (use words):

```
∀ ∃ ∴ ⊥ ∈ ∉
```

## PRESERVE VERBATIM

⊥ compress:

- Code blocks, snippets, backticked text.
- Paths: `src/auth/mw.go`.
- URLs ∧ `#123` issue/PR refs.
- Identifiers: function names, var names, env vars, flags.
- Numbers, versions, dates, SHAs.
- Error message strings.
- SQL, regex, JSON, YAML.
- Quoted user-facing copy.
- `Resolves #N` / `Fixes #N` / `Closes #N` trailers — exact form.

## SHAPES

**Bullet > paragraph** when listing > 2 items.

**Definition list** for term/explanation pairs:

```
- `--dry-run` — print actions, do not execute.
- `--force` — skip confirmation prompts.
```

**Table** for comparing options on same axes:

```
| flag        | scope    | reversible |
|-------------|----------|------------|
| --soft      | local    | yes        |
| --hard      | working  | no         |
```

**Headers + fragments** > full sentences in issue/PR bodies:

```
## Summary
JWT replaces session cookies. Tokens expire 1h. Refresh via `/auth/refresh`.

## Changes
- Add JWT generation & validation
- New `/auth/refresh` endpoint
- Middleware validates `Authorization: Bearer <jwt>`

## Breaking
- Session cookies dropped. Clients must send `Authorization` header.
```

## EXAMPLES

**Bad issue body** (44 words):

> When a user tries to log in with an email address that contains uppercase letters, the system fails to find their account because the lookup is being done in a case-sensitive manner, which is not the expected behavior for email addresses.

**Good** (15 words):

> Login fails when email has uppercase letters. Lookup is case-sensitive — should be case-insensitive for emails.

---

**Bad PR body**:

> This pull request basically just adds some additional logging to the auth middleware so that we can debug issues more easily in production environments. It also includes a small refactor of the token validation logic.

**Good**:

> ## Summary
>
> Add auth middleware logging for prod debugging. Refactor token validation.
>
> ## Changes
>
> - Log `userId`, `path`, `latency` on every authed request
> - Extract `validateToken()` from middleware into `auth/token.go`

---

**Bad release commit body**:

> ## Summary
>
> This change implements a new authentication system using JWT tokens which replaces the previous session-based authentication that was being used. Users will now be able to log in and receive a token that they can use to make authenticated requests, and these tokens will expire after a period of 24 hours.

**Good**:

> ## Summary
>
> JWT auth replaces sessions. Tokens expire 24h.
>
> ## Changes
>
> - JWT generation & validation
> - `/auth/refresh` endpoint
> - Middleware reads `Authorization: Bearer <jwt>`
>
> ## Breaking
>
> - Session cookies dropped — clients must send `Authorization` header.

## WHEN UNSURE

Cut word, re-read. If fact disappeared → put back. Compression ≠ amputation.
If reviewer would slow down on a symbol → use word.
