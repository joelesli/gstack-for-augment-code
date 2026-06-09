---
description: "Engineering retrospective from git history: metrics, sessions, hotspots, plan-vs-reality, per-person feedback. Writes retro.md"
argument-hint: [JIRA-KEY] [--window 7d|14d|30d]
---

You are an engineering manager running a retrospective for Jira ticket **$ARGUMENTS**. Data over vibes: every claim anchors to a commit, a file, or a number. Runs fully unattended — a good weekly cron job with `--window 7d`.

**Scope:** default = the feature (commits on this branch / referencing `<KEY>`, plus the feature folder history). `--window <Nd>` = everything on the default branch in the window, all contributors (team retro).

## Step 0 — Guard rails

`git fetch origin <default>` first. If the newest origin commit predates the window entirely, the local clone is stale — warn and report what you can. No remote → local-only retro, say so.

## Step 1 — Gather raw data

```bash
git log <scope> --format="%h|%an|%ad|%s" --date=iso
git log <scope> --numstat --format="COMMIT:%h|%an"     # LOC per commit; split test vs prod files (test/|spec/|__tests__/)
git log <scope> --format="" --name-only | sort | uniq -c | sort -rn | head -10   # hotspots
```
Plus: CHANGELOG entries in window, PR/MR numbers from commit messages, TODOS state, and the feature folder (plan artifacts = the "plan" side of plan-vs-reality).

## Step 2 — Metrics table

| Metric | Value |
|--------|-------|
| Features shipped (CHANGELOG + merged PRs) | N |
| Commits / contributors / PRs merged | N / N / N |
| Logical SLOC added (non-blank, non-comment — the primary code-volume metric) | N |
| Raw LOC +/− (context only — AI inflates it; ten lines of a good fix outships 10K of scaffold) | +N/−N |
| Test LOC ratio | N% |
| Active days / detected sessions | N / N |
| Backlog health (open TODOs, completed this period) | ... |

Team scope: per-author leaderboard below the table (commits, +/-, top area), current user first as "You".

## Step 3 — Patterns

- **Commit time histogram** (hourly, local time): peak hours, dead zones, late-night clusters worth flagging.
- **Session detection:** 45-minute gap threshold. Deep (50+ min) / medium (20-50) / micro (<20). Total active time, avg session, LOC per active hour.
- **Commit type mix** (feat/fix/refactor/test/chore/docs as % bars). Fix ratio >50% → flag: "ship fast, fix fast" pattern, possible review gap.
- **Hotspots:** files changed 5+ times = churn hotspots; recurring churn in one file is an architectural smell, cross-check `investigate.md` history.
- **Focus score:** % of commits touching the single most-changed top-level directory.
- **Ship of the period:** the highest-impact PR/commit — what and why it matters.

## Step 4 — Plan vs reality (feature scope)

From the feature folder: what did `plan-eng-review.md`/`spec.md` say would be built, in what order, with what effort guess? Compare against what the log shows actually happened. Surface: estimate misses (with the actual), unplanned work (what triggered it), planned-but-dropped (deliberate or forgotten?), review findings that later materialized as fixes anyway (the expensive kind of "told you so" — name them so next time the finding gets fixed at review).

## Step 5 — Per-person analysis (team scope)

For each contributor: commits + LOC, top 3 areas, personal type mix, session pattern, test ratio, biggest ship. The current user gets the deepest treatment, first person ("Your peak hours..."). Each teammate gets 2-3 sentences plus:
- **Praise, specific:** anchored to commits — "shipped the auth middleware rewrite in 3 focused sessions at 45% test coverage", never "great work".
- **One growth opportunity:** a leveling-up suggestion anchored in data — "5 fix commits on the same file suggest the original PR needed a review pass" — not criticism.

Parse `Co-Authored-By:` trailers; credit co-authors. AI co-authors aren't team members — count "AI-assisted commits" as its own metric. Solo repo → skip the team section.

## Step 6 — Lessons + artifact

Three lists, each item anchored to evidence: **Keep doing** (what worked, provably), **Stop doing** (what cost time, with the cost), **Try next** (one concrete process change, not three). Append durable insights to `.augment/learnings.md` if they clear the 5-minute bar.

Write `.augment/features/<KEY>/retro.md` (window mode without a real ticket: `.augment/features/RETRO-<date>/retro.md`):

```markdown
# Retro — <KEY | window>

**Date:** <today>  **Scope:** feature | <N>d window  **Range:** <first>..<last commit>

## Metrics
## Patterns            (histogram, sessions, type mix, hotspots, focus score)
## Ship of the period
## Plan vs reality     (feature scope)
## Team                (window scope)
## Keep / Stop / Try
```

Print in chat: the metrics table, ship of the period, and the single Try-next item.

Do NOT post anything to Jira or Confluence.
