---
description: "Synthesise feature folder into a Confluence page"
argument-hint: [JIRA-KEY]
---

You are a technical writer synthesising the work done so far on **$ARGUMENTS** into a single, readable Confluence page.

## Step 0 — Read everything

Read ALL files present in `.augment/features/$ARGUMENTS/`:

| File | What it contains |
|------|-----------------|
| `office-hours.md` | Original framing, user problem, approach chosen |
| `plan-ceo-review.md` | Scope decisions, 10-star vision, risks |
| `plan-eng-review.md` | Architecture diagrams, hard questions, decisions |
| `test-plan.md` | Test matrix |
| `review.md` | Code review findings, auto-fixes, open items |
| `investigate.md` | Bug investigations and root causes |
| `qa-report.md` | Test results, bugs fixed, health score, verdict |
| `cso-report.md` | Security findings |
| `ship.md` | Branch, PR, test counts |
| `document-release.md` | Doc updates made |
| `retro.md` | Plan vs reality, team breakdown |

Only read what exists — skip files that are not yet present.

## Step 1 — Determine progress stage

Based on which files exist, determine the current stage:

- Only `office-hours.md` → **Ideation**
- `plan-*.md` exist → **Planning**
- `review.md` or `qa-report.md` exist → **In Development**
- `ship.md` exists → **Shipped**
- `retro.md` exists → **Complete**

## Step 2 — Synthesise a narrative

Write the Confluence page content as a clean narrative — NOT a dump of the raw files. A future engineer or product manager reading this page should understand what was built, why, and how it went, without having to read any other document.

Use this structure, including only sections for which content exists:

```
# Feature: $ARGUMENTS — <Jira issue summary if known, otherwise key>

**Status:** <Ideation / Planning / In Development / Shipped / Complete>
**Jira:** [$ARGUMENTS](<jira URL>)
**Last updated:** <today>

---

## What we're building and why
<From office-hours or plan-ceo-review. One or two paragraphs.
What problem does this solve? Who is the user? Why now?>

## What we decided
<From plan-ceo-review and plan-eng-review. The key decisions made —
scope choices, architectural approach, what was cut and why.
Do NOT paste diagrams verbatim — describe the architecture in prose.>

## How it's built
<From plan-eng-review. The components involved, the data flow,
the key design choices. Keep it readable for someone not in the code.>

## Current state
<From qa-report, review, ship. What has been tested, what passed,
what issues were found and fixed. Health score if available.
PR link if shipped.>

## Security
<Only if cso-report.md exists. Summary of findings and status.
Do not include exploit details — just severity, category, and status.>

## Open questions and risks
<From any phase. Unresolved items, deferred decisions, things to watch.>

## What we learned
<From retro.md if it exists. Plan vs reality, key takeaways.>
```

Write only sections for which real content exists. Do not write placeholder sections.

## Step 3 — Upsert the Confluence page

Using the Confluence MCP tool:

1. Ask the user: "Please provide the Confluence parent page URL where the feature page should be created (e.g. `https://your-org.atlassian.net/wiki/spaces/SPACE/pages/123456/Page+Title`)."
2. Search for an existing page titled `$ARGUMENTS — <summary>` or `$ARGUMENTS` under the provided parent page
3. **If the page exists:** update it with the new content
4. **If it does not exist:** create a new child page under the provided parent page with title: `$ARGUMENTS — <Jira summary or key>`

This is an upsert — one page per feature, updated every time you run `gstack:document`.

## Step 4 — Report

Output:

```
gstack:document — $ARGUMENTS
─────────────────────────
Stage:     <Ideation / Planning / In Development / Shipped / Complete>
Files read: N
Confluence: <created / updated> — <page URL>

Sections published:
  ✅ What we're building and why
  ✅ What we decided
  ✅ How it's built
  ✅ Current state
  ✅ Open questions
  ✓  Security — not yet run
  ✓  What we learned — not yet run
```

Do NOT post to Jira.
Do NOT write a local session file — this command reads, it does not write to the feature folder.
