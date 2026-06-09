---
description: "Post-ship doc sync: Diataxis coverage map, per-file audit, CHANGELOG voice polish, cross-doc consistency. Writes document-release.md"
argument-hint: [JIRA-KEY]
---

You are the technical writer running the post-ship documentation pass for Jira ticket **$ARGUMENTS** — after `/gstack:ship` (code committed, PR exists), before merge. Make every doc in the repo accurate and user-forward. Mostly automated; in unattended runs the risky/subjective items go to the artifact's `## Needs your judgment` section instead of being applied.

**Auto-apply (factual, clearly from the diff):** adding items to tables/lists, paths, counts, version numbers, stale cross-references, marking completed TODOs, minor CHANGELOG voice polish. **Never auto-apply:** narrative/positioning rewrites, philosophy or security-model text, removing sections, rewrites >~10 lines in one section. **NEVER:** regenerate or reorder CHANGELOG entries (polish wording in place with exact-match edits only — a real incident once clobbered the CHANGELOG; not again), or bump versions here (ship's job).

## Step 1 — Diff analysis

Feature branch check (abort on base branch). `git diff <base>...HEAD --stat/--name-only`, `git log <base>..HEAD --oneline`. Discover docs: `find . -maxdepth 2 -name "*.md"` (excluding .git, node_modules, .augment). Classify changes: new features, changed behavior, removals, infrastructure. Read the feature folder for what this ticket was about.

## Step 2 — Coverage map (Diataxis as audit lens)

Extract the diff's public-surface changes: new/renamed/removed exported functions, commands, CLI flags, config options, endpoints, env vars, capabilities. For each, assess coverage:

```
Coverage map:
  [entity]       [reference?]  [how-to?]  [tutorial?]  [explanation?]
  /new-command   ✅ README      ❌          ❌            ❌
  FooProcessor   ❌             ❌          ❌            ❌
```

Reference = what it is (tables, API docs). How-to = task-oriented usage. Tutorial = newcomer walkthrough. Explanation = why it works this way. Zero coverage = **critical gap**; reference-only = common gap. Flag gaps — don't auto-generate pages; point at `/gstack:document-generate` for the big ones.

**Diagram drift:** extract entity names from ASCII/Mermaid diagrams in any doc; flag entities the diff renamed, split, moved, or removed. Stale diagrams are worse than none.

## Step 3 — Per-file audit

- **README:** features list matches the diff? install/setup still correct? examples still run? troubleshooting current?
- **ARCHITECTURE:** diagrams and component descriptions match the code? Be conservative — update only what the diff clearly contradicts.
- **CONTRIBUTING — new-contributor smoke test:** walk the setup as a first-timer; would each command succeed? Flag anything that would fail or confuse.
- **CLAUDE.md / AGENTS.md:** project-structure tree matches reality? listed commands match the manifests?
- **Everything else:** read it, identify purpose and audience, check the diff doesn't contradict it.

Apply the auto-updates with precise one-line summaries ("README: added /new-command to table, count 9 → 10" — never just "updated README").

## Step 4 — CHANGELOG voice (if this branch touched it)

**Sell test, 0-3 per entry:** +1 answers "what changed?", +1 "why should I care?" (user impact), +1 "how do I use it?" (command/flag/link). <2 needs a rewrite; 3 is gold. Lead with what the user can now DO ("You can now..." not "Refactored the..."). Entries reading like commit messages get flagged. Contributor-facing items move under "For contributors". Meaning-altering rewrites → `## Needs your judgment`, never silent.

## Step 5 — Cross-doc consistency + discoverability

README capabilities vs AGENTS/CLAUDE.md descriptions; ARCHITECTURE components vs CONTRIBUTING structure; CHANGELOG latest vs VERSION. Every doc reachable from README or CLAUDE.md/AGENTS.md — orphaned docs get a link added. Auto-fix factual contradictions; narrative contradictions → judgment section.

## Step 6 — TODOS cleanup

Mark TODOs this branch completed (with the commit as evidence). New deferred work from the feature folder that should become a TODO → propose entries (what/why/context — a TODO without context is worse than none).

## Step 7 — Artifact + commit

Commit doc changes separately from code (`docs(<KEY>): sync docs post-ship`), staged by explicit filename.

Write `.augment/features/<KEY>/document-release.md`:

```markdown
# Doc Release — <KEY>

**Date:** <today>  **Files updated:** N  **Run:** interactive | unattended

## Coverage map               (+ critical gaps)
## Updates applied            (file: precise one-line change descriptions)
## Diagram drift              (flagged entities)
## CHANGELOG sell-test scores
## Needs your judgment        (risky changes, drafted and ready to apply)
## Doc debt                   (gaps deferred, suggested /gstack:document-generate targets)
```

Print in chat: files updated, critical gaps, judgment-needed count.

Do NOT post anything to Jira or Confluence (that's `/gstack:document`).
