---
description: "Eng review: architecture diagrams and test plan"
argument-hint: [JIRA-KEY]
model: opus
---

You are the best technical lead on the team. The product direction has been set. Now make **$ARGUMENTS** buildable.

## Step 0 — Read context

Fetch the Jira issue **$ARGUMENTS** via the Jira MCP tool. Read description, acceptance criteria, and any linked Confluence pages (architecture docs, data models, runbooks).

Read `.augment/features/$ARGUMENTS/` for any existing session files.

Read the repo's CLAUDE.md, README, ARCHITECTURE docs, and any relevant source files.

## Step 1 — Architecture

Draw the system with ASCII diagrams:

**Component diagram** — what are the major pieces and how do they connect?
**Data flow diagram** — how does data move from input to output?
**State machine** — states and valid transitions for the core entity changed by this feature.
**Sequence diagram** — for the most important user action, what happens step by step?

Rule: if you cannot draw it, the design is not finished.

## Step 2 — Force the hard questions

Answer each, or flag as unresolved:

**Data**
- Schema changes required? New columns, tables, indexes?
- What can be null? What are the constraints?
- What is computed vs stored?

**State**
- Where is canonical state? Who owns it?
- Can state get out of sync? How is it prevented?
- What happens to state when an operation fails halfway?

**Concurrency**
- Can two users touch the same resource simultaneously?
- Where are the race windows? How are they prevented?
- Are transactions scoped correctly for this feature?

**Failure modes**
- What happens when each external dependency is unavailable?
- What happens when the operation partially fails?
- Is there a retry strategy? Is it idempotent?

**Trust**
- What does the client supply that the server must not trust?
- Where is input validated? What happens on validation failure?

**Performance**
- N+1 queries? Unbounded fetches?
- Worst-case data size — will it scale?

## Step 3 — Completeness gaps

Flag any shortcut where the complete version costs less than 30 minutes of Claude Code time. Name both versions explicitly.

## Step 4 — Write session files

**Write `.augment/features/$ARGUMENTS/plan-eng-review.md`:**

```markdown
# Eng Review — $ARGUMENTS

**Date:** <today>
**Jira:** [$ARGUMENTS](<jira URL>)

## Architecture diagrams
<ASCII diagrams>

## Hard questions — answers
<answered and unresolved items>

## Completeness gaps
<list>

## Review Readiness Dashboard
| Review        | Status  | Date   |
|---------------|---------|--------|
| CEO Review    | <from folder or —> | |
| Eng Review    | CLEAR   | <today> |
```

**Write `.augment/features/$ARGUMENTS/test-plan.md`:**

```markdown
# Test Plan — $ARGUMENTS

| Scenario | Type | Priority | Notes |
|----------|------|----------|-------|
| Happy path — <core action> | Integration | P0 | |
| Null input to <method> | Unit | P1 | |
| Concurrent access to <resource> | Integration | P1 | |
| <External dep> unavailable | Integration | P1 | |
| <Edge case from ACs> | Unit | P2 | |
```

Do NOT post anything to Jira or Confluence.

Tell the user: "Architecture is locked. Build against this plan, then run `/gstack:review $ARGUMENTS`."
