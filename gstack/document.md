---
description: "Synthesize the feature folder into one Confluence page (upsert). The only command that writes externally"
argument-hint: [JIRA-KEY] [optional Confluence parent page URL]
---

You are a technical writer synthesizing everything done so far on Jira ticket **$ARGUMENTS** into a single readable Confluence page. This is the ONLY gstack command that writes outside the repo — and only here, only when run.

## Step 0 — Read everything

Read ALL files present in `.augment/features/<KEY>/` (skip what doesn't exist yet):

| File | Contains |
|------|----------|
| `office-hours.md` | Problem framing, demand evidence, chosen approach |
| `spec.md` | Backlog-ready spec, acceptance criteria |
| `plan-ceo-review.md` | Scope mode, premise challenge, registries, vision |
| `plan-design-review.md` / `plan-devex-review.md` | Design/DX scores and decisions |
| `autoplan.md` | Full pipeline gate package |
| `plan-eng-review.md` / `test-plan.md` | Architecture decisions, coverage diagram, test plan |
| `review.md` | Code review findings, fixes, open items |
| `investigate.md` | Bug investigations, root causes |
| `qa-report.md` / `devex-report.md` / `cso-report.md` | QA health score and verdict, DX audit, security findings |
| `ship.md` | Branch, PR, tests, version |
| `document-release.md` | Doc updates made |
| `retro.md` | Plan vs reality, lessons |

Also fetch the Jira issue via the MCP tool for the canonical summary and URL.

## Step 1 — Stage

Only `office-hours.md`/`spec.md` → **Ideation**. `plan-*.md` → **Planning**. `review.md`/`qa-report.md`/`investigate.md` → **In Development**. `ship.md` → **Shipped**. `retro.md` → **Complete**.

## Step 2 — Synthesize the narrative

Write clean narrative, NOT a dump of the raw files. A future engineer or PM reading this page should understand what was built, why, and how it went without opening anything else. Resolve contradictions in favor of the most recent file, and say when something changed ("originally scoped as X; reduced to Y after the eng review found Z"). No placeholder sections — only what has real content.

```
# <KEY> — <Jira summary>

**Status:** Ideation | Planning | In Development | Shipped | Complete
**Jira:** <link>   **PR:** <link if shipped>   **Last updated:** <today>

## What we're building and why     (problem, user, why now — 1-2 paragraphs)
## What we decided                  (scope choices, chosen approach, what was cut and why,
                                     decisions made unattended that still await confirmation)
## How it's built                   (components, data flow, key design choices — prose,
                                     readable for someone not in the code)
## Current state                    (tested what, health score, issues found/fixed, PR status)
## Security                         (severity/category/status only — never exploit details
                                     on a wiki page)
## Open questions and risks         (aggregated from every phase's Open Questions sections)
## What we learned                  (retro: plan vs reality, keep/stop/try)
```

## Step 3 — Upsert to Confluence

Parent page resolution, in order: (1) a URL in the arguments; (2) `.augment/confluence-parent.txt` in the repo (saved from a previous run); (3) interactive → ask for the parent page URL, then offer to save it to that file; (4) unattended with no parent known → **skip the Confluence write**, save the rendered page to `.augment/features/<KEY>/confluence-draft.md`, and report that a parent URL is needed once.

With a parent: search for an existing child page titled `<KEY> — <summary>` (or starting with `<KEY>`). Exists → update it. Doesn't → create it. One page per feature, updated on every run.

## Step 4 — Report

```
gstack:document — <KEY>
Stage: <stage>   Files read: N
Confluence: created | updated | draft-only — <URL or draft path>
Sections published: <list>
```

Do NOT post to Jira. Do NOT write session files to the feature folder (except the draft fallback above) — this command reads.
