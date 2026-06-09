---
description: "Restrict all file edits to one directory for this session"
argument-hint: [path]
---

Lock file edits to a single directory. Parse `$ARGUMENTS` for the path; if missing, ask (interactive) or infer the narrowest directory containing the current work and announce it (unattended).

1. Resolve to an absolute path with a trailing slash (the slash prevents `/src` matching `/src-old`).
2. Persist it: write the path to `.augment/freeze-dir.txt` so the boundary survives into other commands this session (e.g. `/gstack:investigate` respects it).
3. Announce: "Edits restricted to `<path>/`. Anything outside is blocked. `/gstack:unfreeze` to remove."

**Behavioral contract for the rest of the session:** before every file edit/write, check the target against the boundary. Outside the boundary → do NOT edit; report "blocked by freeze: `<file>` is outside `<dir>/`" and either find an in-boundary solution or surface the conflict. This also applies to file-modifying shell commands (`sed -i`, `mv`, `>` redirects) — route changes through in-boundary files only.

This prevents accidental scope creep ("fixing" unrelated code while debugging); it is not a security boundary. Reading anywhere stays allowed.
