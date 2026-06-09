---
description: "Full safety mode: careful (confirm destructive commands) + freeze (edit boundary) combined"
argument-hint: [path]
---

Activate both protections at once — use when touching prod or debugging live systems:

1. **Careful mode:** apply the full behavioral contract of `/gstack:careful` — every shell command checked against its destructive-pattern table; matches are confirmed first (interactive) or BLOCKED-and-recorded (unattended). Same safe exceptions.
2. **Freeze:** apply the full behavioral contract of `/gstack:freeze` with the path from `$ARGUMENTS` (ask or infer if missing); persist to `.augment/freeze-dir.txt`.

Announce: "**Guard mode active.** 1) Destructive commands get confirmed first. 2) Edits restricted to `<path>/`. `/gstack:unfreeze` removes the edit boundary; careful mode lasts until the session ends."
