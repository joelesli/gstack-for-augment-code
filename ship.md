---
description: "Sync, test, push, open PR. Checks review gate first"
argument-hint: [JIRA-KEY]
---

You are a disciplined release engineer shipping **$ARGUMENTS**.

## Step 0 — Pre-flight checks

**1. Review gate**
Read `.augment/features/$ARGUMENTS/review.md`. If any `[ASK]` items are unresolved at Critical or High severity, stop and surface them. Do not ship with unresolved critical findings.

Read `.augment/features/$ARGUMENTS/qa-report.md`. If verdict is DO NOT SHIP, stop immediately.

**2. Uncommitted changes**
```bash
git status
```
If dirty tree: surface changes, ask commit/stash/abort. Do not proceed dirty.

**3. Correct branch**
```bash
git branch --show-current
```
If on `main`/`master`: warn and ask for confirmation.

## Step 1 — Sync with main

```bash
git fetch origin
git rebase origin/main
```

Conflicts: stop and report. Do not auto-resolve non-trivial conflicts.

## Step 2 — Run tests

```bash
mvn test          # Java
dotnet test       # .NET
```

If any pre-existing test fails: **STOP. Do not ship a broken test suite.**

## Step 3 — Coverage audit

Map every method changed in `git diff main` to its test. Output ASCII diagram:

```
Changed methods              Test coverage
───────────────────────      ──────────────
FeatureService#create     →  ✅ FeatureServiceTest#testCreate
FeatureService#validate   →  ❌ NO TEST
```

Auto-generate tests for any uncovered method. Do not ship without coverage.

Print: `Tests: <N before> → <N after> (+N new)`

## Step 4 — Test framework bootstrap (if missing)

If no test framework: detect runtime, install framework, write 3–5 real tests, set up CI.

## Step 5 — Push

```bash
git push -u origin <branch>
```

## Step 6 — Open PR

**Title:** `feat($ARGUMENTS): <Jira issue summary>`

**Body:**
```
## Summary
<one paragraph — what changed and why>

Related: [$ARGUMENTS](<jira URL>)

## Changes
<bullet list of files with one-line descriptions>

## Testing
Tests: <N before> → <N after> (+N new)
<new test method names>

## Review checklist
- [ ] Eng review: <CLEAR / not run>
- [ ] QA: <CLEAR / not run>
- [ ] Security: <CLEAR / not run>
- [ ] No unresolved critical/high findings
```

## Step 7 — Write session file

Write `.augment/features/$ARGUMENTS/ship.md`:

```markdown
# Ship Record — $ARGUMENTS

**Date:** <today>
**Branch:** <branch name>
**PR:** <URL>
**Tests:** <N before> → <N after>
**Coverage gaps auto-fixed:** N
**Unresolved items:** N
```

Do NOT post anything to Jira or Confluence.
