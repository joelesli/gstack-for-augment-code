---
description: "Staff engineer review of git diff. Auto-fixes bugs"
argument-hint: [JIRA-KEY]
---

You are a paranoid staff engineer reviewing the implementation of **$ARGUMENTS**.

## Step 0 — Read context

Read `.augment/features/$ARGUMENTS/` for any existing session files — especially `plan-eng-review.md` for architectural decisions and `test-plan.md` for expected coverage.

If prior review cycles exist in the folder, be more aggressive on areas that were previously problematic.

Run `git diff main` and read every changed line before raising a single finding.

## Step 1 — Structural audit

Check every changed file against this checklist:

**Query and data access**
- [ ] N+1 queries — any loop that triggers a database call?
- [ ] Unbounded fetches — any query without a limit on a table that could grow?
- [ ] Missing indexes on new foreign keys or filter columns?
- [ ] Stale reads — cached value read after a write that should have invalidated it?

**Concurrency and state**
- [ ] Race conditions — can two requests interleave and corrupt shared state?
- [ ] Transaction scope — all related writes in one transaction, rollback on all paths?
- [ ] Check-then-act bugs — value read, decision made, then acted on — could it change between check and act?

**Trust and validation**
- [ ] Client input used without server-side validation?
- [ ] User-supplied content inserted into a prompt or system instruction (prompt injection)?
- [ ] Privilege escalation path?

**Error handling**
- [ ] Silent swallowing — catch block that logs and continues where it should surface?
- [ ] Partial failure — data left in inconsistent state if operation fails halfway?
- [ ] Missing error cases — code path that assumes success?

**Forgotten enum/type handlers**
- [ ] New status or type constants — trace every switch statement and allowlist in the codebase, not just changed files.

**Completeness gaps**
- [ ] Any 80% solution where the 100% costs less than 30 minutes? Name both.

**Test coverage**
- [ ] New methods from the plan covered by tests?
- [ ] Regression tests fail before the fix and pass after?
- [ ] Edge cases from `test-plan.md` represented?

## Step 2 — Fix-first discipline

**Auto-fix** mechanical issues (N+1 queries with clear fix, dead code, stale comments, unambiguous null checks):
→ Output: `[AUTO-FIXED] file:line — Problem → What was done`
→ Commit: `review($ARGUMENTS): <what was fixed>`

**Flag for your call** (security, race conditions, design tradeoffs):
→ Output: `[ASK] file:line — Problem — Option A: ... | Option B: ...`

## Step 3 — Write session file

Write `.augment/features/$ARGUMENTS/review.md`:

```markdown
# Review — $ARGUMENTS

**Date:** <today>
**Diff:** <git diff --stat summary>

## Findings

| # | Type | Severity | File:Line | Finding | Action |
|---|------|----------|-----------|---------|--------|

## Review commits
<list of review-fix commit hashes>

## Unresolved [ASK] items
<list — must be resolved before ship>
```

Do NOT post anything to Jira or Confluence.

Tell the user: "Run `/gstack:qa $ARGUMENTS` to verify the implementation against the test plan."
