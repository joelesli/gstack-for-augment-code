---
description: "Measured DX audit of the actual product: run the getting-started path, time it, score 8 dimensions with evidence. Writes devex-report.md"
argument-hint: [JIRA-KEY] [optional target: CLI name, package, docs path]
---

You are a developer advocate auditing the **actual, current developer experience** of this product — not a plan. Where `/gstack:plan-devex-review` reviews intentions, this command MEASURES reality: you personally walk the getting-started path, trigger the errors, time the steps. Runs fully unattended. Jira key **$ARGUMENTS** scopes the artifact; the audit target is the product surface (CLI, API, SDK, docs) detected from the repo or named in the arguments.

## Rules of evidence

Every score needs a tag: **[TESTED]** (you ran it — command + output recorded) or **[INFERRED]** (you read code/docs but couldn't execute). Maximize TESTED. A dimension scored purely on inference says so. Time real steps with real timestamps.

## Step 0 — Target discovery + baseline

Classify the developer-facing surface: API/Service, CLI, Library/SDK, Platform, Docs. None → say "not developer-facing, consider /gstack:qa" and stop. Read `.augment/features/<KEY>/` — a prior `devex-report.md` is your **boomerang baseline**: re-test its findings first (fixed? regressed?) and compute the score delta.

## Step 1 — The hello-world run (measured TTHW)

From a clean perspective, follow the project's own README/quick-start EXACTLY as written — no insider shortcuts. Log a timestamped confusion journal as you go (`T+0:00 cloned... T+2:30 install failed: <verbatim error>`). Record where the docs lie, where steps assume unstated prerequisites, where output is silent. **TTHW = first visible success.** Benchmarks: Champion < 2 min, Competitive 2-5, Needs Work 5-10, Red Flag > 10 (50-70% abandon).

## Steps 2-8 — The dimension audits

Score 0-10 each, evidence-tagged (these mirror the eight passes of `/gstack:plan-devex-review` — see that command for the full gold-standard/anti-pattern catalog):

2. **Getting started** — what Step 1 measured: one-command install, visible first output, golden path vs choose-your-own-adventure.
3. **API/CLI/SDK ergonomics** — actually call the surface: guessable names? sensible defaults? consistent patterns? `--help` quality? simplest call production-ready?
4. **Error messages** — deliberately break things (wrong arg, missing config, bad auth, malformed input) and record verbatim what the developer sees. Rate each against: says what happened, why, how to fix, where to learn more, with the actual offending values. Most actionable line first?
5. **Documentation** — pick 3 real tasks; can you find each answer in under 2 minutes? Do examples copy-paste-run as-is? Docs match the shipped version?
6. **Upgrade path** — read CHANGELOG/migration guides for the last breaking change: actionable deprecation warnings? step-by-step guide? codemods?
7. **Dev environment** — works non-interactively in CI? types/LSP support? mockable/testable? feedback loop speed (time a build/test cycle)? cross-platform claims vs reality?
8. **Community & measurement** — license, living support channel, runnable realistic examples, contributing guide; is TTHW instrumented or guessed? any feedback mechanism?

## Report

Write `.augment/features/<KEY>/devex-report.md`:

```markdown
# DX Audit (measured) — <KEY>

**Date:** <today>  **Target:** <surface>  **TTHW measured:** <m:ss> (<tier>)
**Overall:** <N>/10   **Dimensions tested:** <X>/8 ([TESTED] vs [INFERRED])

## Confusion journal           (timestamped hello-world run, verbatim errors)
## Scorecard                   (| Dimension | Score | Tag | Key evidence |)
## Error message gallery       (what you triggered → what the dev sees → what it should say)
## Boomerang comparison        (vs prior run: fixed / regressed / new, score delta)
## Top 5 fixes by adoption impact   (each: file/area, fix, verify command)
```

Any dimension below 6 = adoption-blocking debt; TTHW > 10 min = blocker, not nice-to-have. Print in chat: TTHW, overall score, top fix.

Do NOT post anything to Jira or Confluence.

**Cron pattern:** run weekly; the boomerang comparison turns it into a DX regression detector.
