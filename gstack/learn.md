---
description: "Project learnings manager: capture, show, search, prune non-obvious insights that compound across tickets"
argument-hint: [add <insight> | show | search <term> | prune | export]
---

You manage the project's learnings file: `.augment/learnings.md` — durable, non-obvious insights that make every later session smarter. This file is project-level (not per-ticket) because learnings compound across tickets. Other gstack commands read it for context; this command maintains it.

Parse `$ARGUMENTS` for a subcommand. Default (no args) = `show`.

## Entry format

```markdown
## <short-key> — <type>
**Date:** <date>  **Confidence:** <1-10>  **Source:** observed | user-stated | inferred
**Files:** <paths this references>
<the insight, 1-3 sentences, concrete>
```

Types: **pattern** (reusable approach), **pitfall** (what NOT to do), **preference** (user-stated), **architecture** (structural decision), **tool** (library/framework insight), **operational** (env/CLI/workflow knowledge), **investigation** (root cause from a debug session).

Confidence honesty: verified in code = 8-9; user explicitly stated = 10; inference = 4-5.

## Subcommands

**add <insight>** — Capture it. First check it clears the bar: would this save 5+ minutes in a future session? Don't log the obvious, the transient, or what the user already knows. Then check for an existing entry covering the same thing — update it (and its confidence) rather than duplicating. Write the entry; confirm in one line.

**show** — Print the 10 most recent entries, newest first, one line each: `<key> (<type>, <confidence>/10, <date>) — <first sentence>`.

**search <term>** — Match against keys, types, file paths, and body text. Print full matching entries.

**prune** — For each entry: do its referenced files still exist (`ls` them)? Is the insight still true of the current code (spot-check)? Stale or contradicted → show it and remove (interactive: confirm; unattended: remove only entries whose referenced files are gone, flag the rest as `STALE?`). Report kept/removed counts.

**export** — Write `.augment/learnings-export.md` grouped by type, suitable for pasting into CLAUDE.md / AGENTS.md or sharing with the team.

## Standing instruction for other commands

When any gstack command discovers a durable, non-obvious project quirk, gotcha, or command fix mid-run, it should append it here (same bar: saves 5+ minutes next time). When a learning influences a finding, say so: "Prior learning applied: <key> (confidence N/10, from <date>)" — the compounding should be visible.
