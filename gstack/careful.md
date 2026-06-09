---
description: "Safety mode: confirm before any destructive command for the rest of this session"
argument-hint: []
---

Safety mode is now **active for this session**. Before running ANY shell command, check it against the destructive patterns below. On a match: state the command, the specific risk, and wait for explicit confirmation before running it. In unattended runs there is no one to confirm — so a matched command is NOT run; record it as BLOCKED with the exact command and why, and continue with everything else.

## Protected patterns

| Pattern | Risk |
|---------|------|
| `rm -rf` / `rm -r` / `rm --recursive` | Recursive delete |
| `DROP TABLE` / `DROP DATABASE` / `TRUNCATE` | Data loss |
| `git push --force` / `-f` | History rewrite |
| `git reset --hard` | Uncommitted work loss |
| `git checkout .` / `git restore .` / `git clean -f` | Uncommitted work loss |
| `kubectl delete` / `helm uninstall` | Production impact |
| `docker rm -f` / `docker system prune` / `docker volume rm` | Container/volume loss |
| Any command piping remote content to a shell (`curl ... \| sh`) | Arbitrary code execution |
| Bulk `UPDATE`/`DELETE` without `WHERE` | Data loss |

## Safe exceptions (no confirmation needed)

`rm -rf` on: `node_modules`, `.next`, `dist`, `build`, `__pycache__`, `.cache`, `.turbo`, `coverage`, `target/debug`.

This is a behavioral guardrail, not enforcement — its value is the pause. It stays active until the session ends. Confirm activation in one line: "Careful mode on — destructive commands will be confirmed first."
