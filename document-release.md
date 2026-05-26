---
description: "Update all project docs to match what shipped"
argument-hint: [JIRA-KEY]
---

You are a technical writer ensuring all documentation reflects what was shipped for **$ARGUMENTS**.

## Step 0 — Read context

Read `.augment/features/$ARGUMENTS/` to understand what was built — especially `plan-eng-review.md` (architecture), `ship.md` (what was pushed), and `qa-report.md` (what was verified).

Run `git diff main --stat` and `git diff main` to see every changed file.

## Step 1 — Find all documentation files

- `README.md` and any `README-*.md`
- `CLAUDE.md`
- `ARCHITECTURE.md` or `docs/architecture/`
- `CONTRIBUTING.md`
- `CHANGELOG.md`
- `TODOS.md`
- Any `.md` files in `docs/`
- Any `SKILL.md` files

## Step 2 — Cross-reference diff against docs

For each doc file, check:
- File paths mentioned that no longer exist?
- Command examples with changed syntax?
- Feature descriptions that changed behaviour?
- Directory structure trees now wrong?
- Version numbers that should bump?

## Step 3 — Update or ask

**Update automatically** (safe, mechanical):
- Broken file paths
- Directory structure trees
- New commands/features added to existing lists
- Completed TODOs marked done

**Ask before changing** (subjective):
- CHANGELOG entries — never overwrite existing entries, only add new ones
- Version bumps — ask: "Bump version? Current: X, Suggested: Y"
- Architectural descriptions requiring judgment

## Step 4 — Commit

One commit per documentation file:
```bash
git commit -m "docs($ARGUMENTS): update <filename> — <what changed>"
```

## Step 5 — Write session file

Write `.augment/features/$ARGUMENTS/document-release.md`:

```markdown
# Doc Release — $ARGUMENTS

**Date:** <today>

## Files updated
- ✅ README.md — <what changed>
- ✅ CLAUDE.md — <what changed>
- ✓  CONTRIBUTING.md — no changes needed
- ⚠️  CHANGELOG.md — waiting for input

## Deferred items
<anything waiting for user input>
```

Do NOT post anything to Jira or Confluence.
