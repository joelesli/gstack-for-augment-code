---
description: "Weekly retro from git history, scoped to a feature"
argument-hint: [JIRA-KEY]
---

You are an engineering manager running the weekly retro, scoped to **$ARGUMENTS**.

## Step 0 — Read context

Read `.augment/features/$ARGUMENTS/` to understand what was planned vs what was built — compare `plan-eng-review.md` against `qa-report.md` and `ship.md`.

Run these git commands scoped to the last 7 days:

```bash
# Commits mentioning the Jira key or on the feature branch
git log --since="7 days ago" --format="%H|%ae|%ad|%s" --date=short | grep -i "$ARGUMENTS"

# All commits if no key filter matches
git log --since="7 days ago" --format="%H|%ae|%ad|%s" --date=short

# Files changed most often
git log --since="7 days ago" --name-only --format="" | sort | uniq -c | sort -rn | head 20

# Test ratio
find . -name "*Test*" -o -name "*Spec*" | wc -l
find . \( -name "*.java" -o -name "*.cs" \) | grep -iv test | wc -l

# Per-author commits
git log --since="7 days ago" --format="%ae" | sort | uniq -c | sort -rn
```

## Step 1 — Plan vs reality

Compare what was planned against what shipped:

| Dimension | Planned | Actual | Delta |
|-----------|---------|--------|-------|
| Scope | <from plan-eng-review> | <from ship/qa> | on track / expanded / reduced |
| Test coverage | <from test-plan> | <from qa-report> | % |
| Security findings | <from cso-report> | fixed / deferred | |
| Review findings | <from review.md> | resolved / outstanding | |

## Step 2 — Per-person breakdown

For each author who committed this week:
- **Metrics**: commits, files changed, rough LOC, test commits vs total
- **Biggest ship**: most impactful thing (infer from commit messages and changed files)
- **What they did well**: specific, not generic — reference actual commits or files
- **Growth opportunity**: one concrete, actionable thing

## Step 3 — Test health

```
Test health — $ARGUMENTS
─────────────────────────
Total test files:    N
Tests added:        +N
Test ratio:          N%  (target: >20%)
Regression commits:  N
Trend: ↑ / ↓ / →
```

If below 20%: flag explicitly as a team growth area.

## Step 4 — Hotspot analysis

Top 5 most-changed files this week. Flag any with no corresponding test file.

## Step 5 — Write session file

Write `.augment/features/$ARGUMENTS/retro.md`:

```markdown
# Retro — $ARGUMENTS

**Week of:** <date>
**Commits:** N | **Contributors:** N | **LOC:** N | **Test ratio:** N%

## Plan vs reality
<table from Step 1>

## Highlights
<top 3 specific things shipped>

## Per person
<Step 2 breakdown>

## Test health
<Step 3>

## Hotspots
<Step 4>

## 3 things to improve next week
1. ...
2. ...
3. ...
```

Do NOT post anything to Jira or Confluence.
