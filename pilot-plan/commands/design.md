---
description: Run a propose-then-critique structural design session — model proposes shape, user critiques, converge on Open Questions empty → file new GitHub issue with the `design` label.
argument-hint: <topic or empty>
---

# /gh:design — structural design session

Invoke the `design` skill with `$ARGUMENTS`.

If `$ARGUMENTS` is empty or vague, the skill asks 1–2 short questions to localize the topic before proposing.

The skill handles: SPEC.md read (degrades gracefully if missing), reactive code reads, propose-shape pass, explicit `## Open Questions` section, iterative critique loop, convergence on empty Open Questions, then on user confirmation files a new GitHub issue via `gh issue create` with the `design` label (creating the label if missing).

If the user signals they already know what they want ("just file it", "skip the design"), stop the dialogue and route directly to `/gh:issue` with their verbatim intent.
