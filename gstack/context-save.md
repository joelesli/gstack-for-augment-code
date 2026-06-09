---
description: "Checkpoint the session: decisions, state, failed approaches, next steps — so a fresh session resumes losslessly"
argument-hint: [JIRA-KEY] [optional note]
---

Save a checkpoint of this working session for Jira ticket **$ARGUMENTS** so a future session (or a teammate, or you after a crash) resumes without re-deriving anything. Run this before ending a long session, before risky operations, or when context is getting heavy.

Write `.augment/features/<KEY>/context.md` (overwrite — it's a snapshot, not a log; prior snapshots' "Decisions" entries get carried forward into the new one):

```markdown
# Context Checkpoint — <KEY>

**Saved:** <timestamp>  **Branch:** <branch> @ <short-hash>  **Note:** <user note if any>

## Goal                       (what this session is trying to accomplish, one paragraph)
## State of play              (what's DONE and verified, what's IN PROGRESS and exactly where
                               it stands — "function written, test failing on the empty-input
                               case" not "working on tests")
## Decisions made             (each with the one-line WHY — carried forward across checkpoints)
## Failed approaches          (what was tried and didn't work, and why — the most valuable
                               section; prevents the next session from re-walking dead ends)
## Uncommitted work           (git status summary; files modified and what's in them)
## Next steps                 (ordered; first item specific enough to start cold)
## Watch out for              (gotchas discovered: flaky test, env quirk, load-bearing hack)
```

Ground every line in evidence — re-run `git status` and `git log -5` now rather than trusting memory. Mention which gstack artifacts exist in the folder so the resuming session reads them.

If there is uncommitted work and the repo uses WIP commits comfortably, offer (interactive) or note (unattended) a `WIP(<KEY>): checkpoint` commit as a harder save point — never push it.

Print: checkpoint path, next-step #1.

Resume with `/gstack:context-restore <KEY>`.
