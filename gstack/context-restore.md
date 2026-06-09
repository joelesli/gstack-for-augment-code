---
description: "Resume from a saved checkpoint: rebuild working context, verify it's still true, continue"
argument-hint: [JIRA-KEY]
---

Restore working context for Jira ticket **$ARGUMENTS** and get productive immediately.

1. **Read the checkpoint:** `.augment/features/<KEY>/context.md`. Missing → reconstruct best-effort from the rest of the feature folder + `git log` + `git status`, and say the reconstruction is inferred, not saved.
2. **Read the supporting artifacts** the checkpoint names (plan, review, qa files) — skim for their verdicts and open items, don't re-read wholesale.
3. **Verify the checkpoint against reality** — it reflects when it was written, not now:
   - `git status` + `git log <checkpoint-hash>..HEAD --oneline` — did commits land since? Does "uncommitted work" still match?
   - Spot-check the IN PROGRESS claim (is that test still failing the same way?).
   - Note any drift explicitly: "checkpoint says X, repo now shows Y."
4. **Brief back** in chat, compactly: goal (1 line), state of play (3-5 lines), decisions that constrain you, failed approaches NOT to retry, drift found, and next step #1.
5. **Continue:** pick up next step #1 unless the user redirects. Honor the checkpoint's "watch out for" list and the feature folder's unresolved findings — they're still binding.

Do not redo work the checkpoint marks DONE-and-verified; do re-verify anything marked IN PROGRESS before building on it.
