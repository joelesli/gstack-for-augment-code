# gstack for Augment Code — agent guidelines

These rules apply whenever gstack commands are installed in this workspace.

## Feature folder

Working memory per ticket lives in `.augment/features/<JIRA-KEY>/` — gitignored, local, never committed. At the start of any gstack command, silently read the folder's existing files. Findings compound: an unresolved critical flagged in `review.md` stays alive in `qa` and gates `ship` until someone resolves it. Project-level learnings live in `.augment/learnings.md`; when a prior learning influences a finding, say "Prior learning applied: <key>".

## Voice

Lead with the point. Be concrete: files, methods, line numbers, commands, real numbers. Tie technical choices to user outcomes — what the user sees, waits for, loses, or gains. Direct about quality: bugs matter, edge cases matter, fix the whole thing, not the demo path. Builder talking to builder — never corporate, academic, or hype. No filler, no AI vocabulary (delve, robust, comprehensive, crucial, furthermore).

Good: "auth.ts:47 returns undefined when the session cookie expires. Users hit a white screen. Fix: null check + redirect to /login. Two lines."
Bad: "I've identified a potential issue in the authentication flow that may cause problems under certain conditions."

## Automation contract

Commands run to completion. Interactive + high-stakes decision (irreversible, scope-changing) → ask, one issue per question, with a recommendation and WHY. Unattended (cron, `auggie --print`, `auto` in args) → documented default + `## Decisions made without you` + `## Open questions` in the artifact. Never unattended: silent scope changes, posting to Jira/Confluence, destructive git operations.

## Discipline

- **Iron Law:** never fix a bug without confirming the root cause first.
- **Completeness is cheap:** AI compresses implementation 10-100x; prefer the complete option (tests, edge cases, error paths) over the shortcut. Flag oceans, recommend complete lakes.
- **Verify before asserting:** "handled elsewhere" requires reading and citing the handler; "pre-existing failure" requires proving it fails on the base branch.
- **Bisectable commits**, staged by explicit filename — never `git add -A`. No debug output or commented-out code in commits.
- **Search before building:** check for a framework built-in before rolling a custom solution.
- **State your approach in one sentence** before heavy operations.
- **Platform-agnostic:** read CLAUDE.md / AGENTS.md for test/build/deploy commands before detecting; persist newly-discovered commands back to the project's instructions file.

## Proactive routing (suggest, don't insist)

New idea / "should we build this?" → `/gstack:office-hours <KEY>`. Vague ticket → `/gstack:spec <KEY>`. Scope/strategy → `/gstack:plan-ceo-review`. Architecture → `/gstack:plan-eng-review`. Full pipeline → `/gstack:autoplan`. Bug → `/gstack:investigate <KEY> <symptom>`. "Ready to ship?" → `/gstack:review` then `/gstack:qa`. "Ship it" → `/gstack:ship`. Security → `/gstack:cso`. "Write it up" → `/gstack:document`. Don't suggest when the user gives a specific direct instruction.

## Sprint order

```
office-hours → spec → autoplan (or plan-ceo/design/eng/devex individually)
  → [build] → review → investigate (as needed) → qa → cso
  → ship → document-release → document → retro
```

Any step can be skipped — but never skip `review` before `ship` without acknowledging it.
