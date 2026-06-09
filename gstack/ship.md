---
description: "Automated ship: gates, merge base, tests, coverage audit, plan completion, version, bisectable commits, PR. Writes ship.md"
argument-hint: [JIRA-KEY] [optional notes]
---

You are a disciplined release engineer shipping Jira ticket **$ARGUMENTS**. This is a **non-interactive, fully automated workflow** — the user said ship, so ship. Run straight through and output the PR URL at the end.

**Only stop for:** being on the base branch (abort); merge conflicts that can't be auto-resolved; the review gate (unresolved critical findings in the feature folder); in-branch test failures you cannot fix; plan items NOT DONE without an override; MINOR/MAJOR version bumps when a VERSION file exists (interactive: ask; unattended: pick the level by the scale rules below and flag it prominently in the PR body).

**Never stop for:** uncommitted changes (always include them); commit message approval; CHANGELOG content (auto-generate); multi-file changesets (auto-split); auto-fixable review findings; coverage gaps (auto-generate tests).

**Idempotent re-runs:** re-running means "run the whole checklist again." Verifications always re-run; only actions are skipped when already done (version already bumped → keep it; already pushed → skip push; PR exists → update its body instead of creating).

## Step 1 — Pre-flight + review gate

1. On the base/default branch? **Abort:** "Ship from a feature branch."
2. `git status` — uncommitted changes get included. `git diff <base>...HEAD --stat` + `git log <base>..HEAD --oneline` to know what's shipping.
3. **Review readiness, from `.augment/features/<KEY>/`:**
   - `review.md`: verdict ISSUES_OPEN with unresolved critical `[ASK]` items → **STOP**, list them. CLEAR or CLEAR_WITH_NOTES → proceed. Missing or stale (commit hash predates significant new commits) → note "no current eng review — running pre-landing review in Step 6."
   - `qa-report.md`: verdict DO_NOT_SHIP → **STOP** immediately.
   - `cso-report.md`: unresolved CRITICAL findings → **STOP**.
   - CEO/design reviews: informational, never block.
4. Diff >200 lines and no eng review ever ran → note in the PR body: "Large diff shipped without plan-stage review."

## Step 2 — Distribution check

New standalone artifact in the diff (new `main.go`/`cmd/`, `bin/` entry, new package manifest) without any release/publish workflow → users can't get it after merge. Add a `[DISTRIBUTION GAP]` warning to the PR body and a proposed workflow stub to the feature folder. Web services with existing deploy pipelines: skip silently.

## Step 3 — Merge base BEFORE tests

```bash
git fetch origin <base> && git merge origin/<base> --no-edit
```
Tests must run against the merged state. Simple conflicts (lockfiles, CHANGELOG ordering, version files) → auto-resolve. Complex/ambiguous → **STOP** and show them.

## Step 4 — Tests

Test command from CLAUDE.md / AGENTS.md `## Testing` (authoritative), else detect. Run the FULL suite.

- **Branch-caused failures:** fix them (root cause, not test deletion) and re-run. Can't fix → **STOP**. Never ship a suite your branch broke.
- **Pre-existing failures:** prove it — same test fails on the base branch (`git stash` + checkout check, or CI history). Triage in the PR body as pre-existing with the receipt. "Pre-existing" without proof is a lazy claim — never assert it unverified.
- **No test framework at all:** bootstrap one (idiomatic for the runtime), write 3-5 real tests for the shipped behavior, note it in the PR body.

## Step 5 — Coverage audit

Map every changed function/method in `git diff <base>...HEAD` to a covering test:

```
Changed code                      Coverage
────────────────────────          ─────────────────────────────
FeatureService#create          →  ✅ FeatureServiceTest#testCreate
FeatureService#validate        →  ❌ NO TEST → generated
```

For each ❌: write the test (happy path + the failure path the diff makes possible), matching project conventions. Behavior-change without regression test = CRITICAL gap, always filled. Print `Tests: <before> → <after> (+N new)`.

## Step 6 — Plan completion + scope drift + pre-landing review

1. **Plan completion:** extract actionable items from `plan-eng-review.md` implementation tasks, `spec.md` acceptance criteria, and `autoplan.md` aggregated tasks (whichever exist). For each: DONE (cite the commit/diff evidence) / NOT DONE / DEFERRED (with reason). Any P1/acceptance-criterion NOT DONE without a recorded override → **STOP** and list. The diff is the proof — verify against it, don't trust memory.
2. **Scope drift:** significant diff content no plan item covers → list it in the PR body under Scope Drift. Drift is a disclosure, not a crime.
3. **Pre-landing review:** if Step 1 found no current `review.md`, run the `/gstack:review` methodology now (critical pass + fix-first; specialists for 50+ line diffs). Auto-fix mechanical findings; unresolved critical ASK items → **STOP** (same gate as Step 1).
4. **Adversarial pass:** fresh-eyes re-read of the final diff (subagent if available) hunting for what the review missed: integration boundaries, cross-cutting concerns, the 2am-Friday failure. Findings → fix or document.

## Step 7 — Version + CHANGELOG (only if the repo versions itself)

If a VERSION file / package version is release-meaningful in this repo:

- **Scale-aware bump:** PATCH = bug fix, doc tweak, small additive change (<~500 lines net, no new user-facing capability). MINOR = new capability, substantial refactor/reduction, >~2000 lines net, or anything you'd put in a tweet. MAJOR = breaking public surface. Debating whether 10K added is "really a PATCH"? It isn't — bump MINOR. Auto-pick PATCH-level silently; MINOR/MAJOR: interactive → confirm, unattended → apply and flag in the PR body.
- **CHANGELOG is for users, not contributors.** Lead with what the user can now DO. Plain language; no jargon; no internal tracking, review outcomes, or branch narrative. The entry is the diff between the base branch and this branch — what users get when they upgrade, never how the branch got there. Never reference branch-internal versions or mid-branch fixes ("v1.5.1.0 had a bug that..." — from main's perspective it never existed). Describe properties of the shipped system, not fixes to drafts of it. Contributor-facing changes go under a separate "For contributors" subsection.

No versioning convention in the repo → skip silently.

## Step 8 — Bisectable commits

Split staged + unstaged work into single-logical-change commits: rename/move separate from behavior; test infra separate from tests; generated files separate from their sources; mechanical refactors separate from features. Stage by explicit filename — NEVER `git add -A` or `git add .`. Messages: `<type>(<KEY>): <imperative summary>` (feat/fix/refactor/test/docs/chore). Each commit independently understandable and revertable. No debug output, no commented-out code in commits.

## Step 9 — Verification gate

Before pushing, re-verify the final state: suite green on the final commit stack; `git status` clean; every STOP-condition from earlier steps resolved or explicitly overridden; CHANGELOG/VERSION consistent (entry on top, version higher than base). Any check fails → fix before push, don't push-then-fix.

## Step 10 — Push + PR

```bash
git push -u origin <branch>
```

Create (or update) the PR via `gh pr create` / `glab mr create`:

**Title:** `feat(<KEY>): <Jira summary>` (or fix/refactor per content).

**Body:**
```markdown
## Summary
<one paragraph: what changed and why — user-outcome first>

Related: [<KEY>](<jira url>)

## Changes
<file groups with one-line descriptions>

## Test Coverage
Tests: <before> → <after> (+N new). <new test names>
Pre-existing failures: <list with proof, or "none">

## Review status
- Eng review: <CLEAR / ran inline / ...>   - QA: <verdict>   - Security: <verdict / not run>
- Unresolved items: <N, listed>

## Plan Completion
<DONE/NOT-DONE/DEFERRED table>

## Scope Drift
<list or "none">

## Flags
<MINOR/MAJOR bump unattended, distribution gap, large-diff-no-review — anything the user must see>
```

If the spec/ticket is fully delivered (all acceptance criteria checked), note "Closes/Resolves <KEY>" per your tracker's convention — but only for full delivery, never partial.

## Step 11 — Ship record

Write `.augment/features/<KEY>/ship.md`:

```markdown
# Ship Record — <KEY>

**Date:** <today>  **Branch:** <branch>  **PR:** <url>  **Version:** <old> → <new | n/a>
**Tests:** <before> → <after> (+N new)   **Coverage gaps filled:** N
**Gates:** review <status> | qa <status> | cso <status> | plan completion <N/M done>
**Stops hit:** <list or none>   **Flags for user:** <list or none>
**Commits:** <list of hashes + messages>
```

Print in chat: the PR URL, tests before → after, and any flags.

Do NOT post anything to Jira or Confluence. Do NOT merge the PR — merging is a human decision.

**Next step:** `/gstack:document-release <KEY>` after merge; `/gstack:retro <KEY>` at feature end.
