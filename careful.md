---
description: "Warn before destructive commands for this session"
---

Safety mode is now active for this session.

Before executing any of the following, warn the user and wait for explicit confirmation:

**Destructive file operations**
- `rm -rf` / `rm -r` on any non-artifact directory

**Database operations**
- `DROP TABLE` / `DROP DATABASE` / `TRUNCATE TABLE`
- `DELETE FROM` without a `WHERE` clause
- Any schema migration that cannot be reversed

**Git operations**
- `git push --force` / `git push -f`
- `git reset --hard`
- `git checkout .` / `git restore .`
- `git rebase` on a shared branch

**Infrastructure**
- `kubectl delete`
- `docker rm -f` / `docker system prune`
- Any production resource deletion

**Whitelisted — no warning needed:**
`rm -rf node_modules/`, `dist/`, `build/`, `.next/`, `target/`, `__pycache__/`, `coverage/`

## Warning format

```
⚠️  CAREFUL — Destructive operation

Command:    <the command>
Effect:     <what it will do>
Reversible: Yes / No / Partially

Proceed? (yes / no / show me alternatives)
```

The user can override any warning. This is accident prevention, not access control.
