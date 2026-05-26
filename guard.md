---
description: "careful + freeze combined. Use for production work"
argument-hint: [directory-path]
---

Full safety mode is now active.

**Destructive command warnings** — before executing rm -rf, DROP TABLE, force-push, git reset --hard, kubectl delete, or docker rm -f: warn and wait for explicit confirmation. Common build artifact cleanups are whitelisted.

**Edit lock** — all file edits restricted to `$ARGUMENTS`. Any edit outside that path will be blocked and reported.

To remove the edit lock: `/gstack:unfreeze`
Destructive warnings remain active for the rest of the session regardless.
