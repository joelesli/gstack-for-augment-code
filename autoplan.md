---
description: "Full planning autopilot: CEO → Eng review + taste gate"
argument-hint: [JIRA-KEY]
model: opus
---

You are running the full planning pipeline for **$ARGUMENTS** automatically.

## Step 0 — Setup

Create `.augment/features/$ARGUMENTS/` if it does not exist.

Fetch the Jira issue **$ARGUMENTS** via the Jira MCP tool. Read description, acceptance criteria, linked Confluence pages.

Read any existing session files in `.augment/features/$ARGUMENTS/`.

## Encoded decision principles

Apply these automatically, in order, when resolving decisions during the run:

1. **Prefer completeness** — if two options have similar effort, pick the more complete one
2. **Match existing patterns** — if the codebase already does X one way, do it that way
3. **Choose reversible options** — prefer what is easier to undo or change later
4. **Prefer what was chosen for similar past decisions** — read existing session files
5. **Defer ambiguous items** — save them for the taste gate
6. **Escalate security** — any security concern is always a taste decision, never auto-resolved

## Step 1 — CEO Review (automated)

Run the logic from `/gstack:plan-ceo-review` in SELECTIVE EXPANSION mode.

- Clear decisions → apply automatically, log as `[AUTO] Decision: <what> → <choice> (principle: <N>)`
- Genuine tradeoffs → save as TASTE DECISION

## Step 2 — Eng Review (automated)

Run the logic from `/gstack:plan-eng-review`.

- Standard architectural decisions → apply encoded principles
- Ambiguous architecture tradeoffs → save as TASTE DECISION
- Test plan → write automatically, maximise coverage

## Step 3 — Taste gate

Present all deferred decisions:

```
TASTE DECISIONS — need your input
══════════════════════════════════

1. Scope: [description]
   Option A: ... (I lean A because ...)
   Option B: ...

2. Architecture: [description]
   Option A: ...
   Option B: ...

[Reply with numbers and letters: "1A, 2B"]
```

## Step 4 — Write session files

Apply the user's choices. Write:
- `.augment/features/$ARGUMENTS/plan-ceo-review.md`
- `.augment/features/$ARGUMENTS/plan-eng-review.md`
- `.augment/features/$ARGUMENTS/test-plan.md`

Do NOT post anything to Jira or Confluence.

Print summary:
```
Autoplan complete — $ARGUMENTS
────────────────────────────────
Auto-resolved: N decisions
Taste decisions: N (all resolved)

Ready to build. Run /gstack:review $ARGUMENTS after implementing.
```
