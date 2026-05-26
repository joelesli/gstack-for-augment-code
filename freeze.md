---
description: "Lock file edits to one directory path"
argument-hint: [directory-path]
---

Edit lock is now active.

**Allowed edits:** only files inside `$ARGUMENTS`

**Blocked:** any Edit, Write, or file creation outside `$ARGUMENTS`

When about to edit a file outside the boundary, stop and say:
```
BLOCKED — Edit outside freeze boundary ($ARGUMENTS)
File: <path>
Skipping this change.
```

To remove: run `/gstack:unfreeze`

Note: this blocks Edit and Write tool calls. Bash commands like `sed` can still reach outside — this is accident prevention, not a security sandbox.
