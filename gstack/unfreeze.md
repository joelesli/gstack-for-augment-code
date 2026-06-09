---
description: "Remove the edit boundary set by freeze or guard"
argument-hint: []
---

Clear the edit restriction: if `.augment/freeze-dir.txt` exists, read it, delete it, and confirm "Freeze boundary cleared (was: `<path>`). Edits allowed everywhere." If it doesn't exist, say "No freeze boundary was set." Careful mode (if active via `/gstack:careful` or `/gstack:guard`) is unaffected and stays on until the session ends.
