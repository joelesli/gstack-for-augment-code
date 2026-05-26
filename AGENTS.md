# gstack for Augment Code

## Available commands

| Command | Specialist | Jira fetch | Writes to |
|---------|-----------|-----------|-----------|
| `/gstack:office-hours <KEY>` | YC Partner | ✅ yes | `.augment/features/<KEY>/office-hours.md` |
| `/gstack:plan-ceo-review <KEY>` | CEO/Founder | ✅ yes | `.augment/features/<KEY>/plan-ceo-review.md` |
| `/gstack:plan-eng-review <KEY>` | Eng Manager | ✅ yes | `.augment/features/<KEY>/plan-eng-review.md` + `test-plan.md` |
| `/gstack:autoplan <KEY>` | Review Autopilot | ✅ yes | All three plan files |
| `/gstack:review <KEY>` | Staff Engineer | label only | `.augment/features/<KEY>/review.md` |
| `/gstack:investigate <KEY> <symptom>` | Debugger | label only | `.augment/features/<KEY>/investigate.md` |
| `/gstack:qa <KEY>` | QA Lead | label only | `.augment/features/<KEY>/qa-report.md` |
| `/gstack:cso <KEY>` | Security Officer | label only | `.augment/features/<KEY>/cso-report.md` |
| `/gstack:ship <KEY>` | Release Engineer | label only | `.augment/features/<KEY>/ship.md` |
| `/gstack:document-release <KEY>` | Tech Writer | label only | `.augment/features/<KEY>/document-release.md` |
| `/gstack:retro <KEY>` | Eng Manager | label only | `.augment/features/<KEY>/retro.md` |
| `/gstack:document <KEY>` | Narrator | label only | Confluence upsert (reads folder, writes nothing locally) |
| `/gstack:careful` | Safety | — | session only |
| `/gstack:freeze <path>` | Safety | — | session only |
| `/gstack:guard <path>` | Safety | — | session only |
| `/gstack:unfreeze` | Safety | — | session only |

## Feature folder

Every command that accepts a Jira key reads from and writes to:
```
.augment/features/<JIRA-KEY>/
```

This folder is gitignored, local working memory, never committed. Each command is independent — run them in any order, they read whatever exists and skip what doesn't.

Add to `.gitignore`:
```
.augment/features/
```

## Proactive routing

When the user's request matches a command's purpose, suggest it — but do not insist.

- New feature idea / "should we build this?" → suggest `/gstack:office-hours <KEY>`
- "What should the architecture look like?" → suggest `/gstack:plan-eng-review <KEY>`
- Bug or "why is this broken?" → suggest `/gstack:investigate <KEY> <symptom>`
- "Is this ready to ship?" → suggest `/gstack:review <KEY>` then `/gstack:qa <KEY>`
- "Ship it" → suggest `/gstack:ship <KEY>`
- "Security check" → suggest `/gstack:cso <KEY>`
- "Write it up" / "document this" → suggest `/gstack:document <KEY>`

Do NOT suggest commands when the user gives a specific direct instruction.

## Discipline nudges

**Think before heavy actions.** For complex operations, state your approach in one sentence before executing.

**Mark tasks complete individually.** Do not batch-complete a multi-step plan at the end.

**Iron Law.** Never fix a bug without confirming the root cause first.

**One commit per logical change.** Keep commits bisectable.

**No debug output in commits.** Remove `System.out.println`, `Console.WriteLine`, commented-out code before committing.

**Search before building.** If unsure whether something exists in the codebase, search first.

## Feature folder awareness

At the start of any command that receives a Jira key, silently check `.augment/features/<KEY>/` and read relevant prior session files. Prior decisions compound — if a race condition was flagged in `review.md` and not resolved, flag it again in `qa` and `ship`.

## Sprint order

```
office-hours → plan-ceo-review → plan-eng-review
    → [build] → review → investigate (if needed)
    → qa → cso → ship → document-release
    → gstack:document → retro
```

Commands are independent — skip any step — but never skip `review` before `ship` without explicitly acknowledging it.
