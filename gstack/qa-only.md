---
description: "Report-only QA: test against test-plan.md, document with evidence, fix NOTHING. Writes qa-report.md"
argument-hint: [JIRA-KEY] [--quick | --regression]
---

You are a QA lead testing the work behind Jira ticket **$ARGUMENTS** in **report-only mode**: find and document, change nothing. Use this when someone else owns the fixes, or when you want a clean severity picture before deciding what to do. Runs fully unattended.

**HARD RULE: make NO code changes.** No fixes, no refactors, no "harmless" cleanups. The single exception: you MAY write new test files that expose bugs (clearly named, e.g. `*.qa-repro.*`) — they document the bug executably and become the regression test when someone fixes it. Never modify existing source or existing tests.

## Method

Identical to `/gstack:qa` phases 1-3 and 5-6, with the fix loop removed:

1. **Inputs:** read `.augment/features/<KEY>/` — `test-plan.md` is the primary input; `review.md` findings get re-verified; prior `qa-report.md`/`qa-baseline.json` is the regression baseline. No test-plan → derive one from the branch diff + Jira acceptance criteria (fetch via the Jira MCP tool) and say so.
2. **Modes:** diff-aware by default (scope to `git diff <base>...HEAD`); `--quick` = smoke only; `--regression` = full + baseline diff.
3. **Test infrastructure:** find the test command (CLAUDE.md / AGENTS.md first, then detect). NO framework exists → that is a Critical finding; do NOT bootstrap one in report-only mode — describe exactly what bootstrap is needed.
4. **Execute:** run the suite (record pre-existing vs branch-caused failures); for each plan item without a covering test, write a repro test and run it; live-check affected routes if the app boots locally (status codes, error text, realistic + edge-case payloads). Test shadow paths: empty, invalid, double-submit, error path.
5. **Document:** each issue immediately, with severity (Critical/High/Medium/Low), category, found-by, exact repro, verbatim evidence, expected behavior. Verify twice before documenting. `[REDACTED]` for credentials.
6. **Health score:** same rubric and weights as `/gstack:qa` (Functional 25, Error handling 20, Plan coverage 20, Data safety 15, Performance 10, Polish 10; deductions Critical -25 / High -15 / Medium -8 / Low -3).

## Report

Write `.augment/features/<KEY>/qa-report.md` (same format as `/gstack:qa`, every issue status `OPEN`, plus a `## Recommended fix order` section: severity-sorted, each with the suspected root cause and suggested fix — ready for `/gstack:investigate` or a teammate). Write `qa-baseline.json`.

Verdict: any Critical → DO_NOT_SHIP; High → SHIP_WITH_NOTES at most. Print verdict, score, top-3 in chat.

Do NOT post anything to Jira or Confluence.

**Next step:** `/gstack:investigate <KEY> <top issue>` to fix with root-cause discipline, or `/gstack:qa <KEY>` for the fixing variant.
