# gstack for Augment Code

A port of [Garry Tan's gstack](https://github.com/garrytan/gstack) as Augment Code
commands. Every command accepts a Jira key, reads/writes a local feature folder
`.augment/features/<KEY>/`, and a `gstack:document` command synthesises the folder
into a Confluence page on demand.

![](gstack_for_augment_400.jpg)

---

## Install

### macOS / Linux

**User-level install** (available in every project):
```bash
cp -r gstack ~/.augment/commands/
```

**Workspace-level install** (available only in one project):
```bash
cp -r gstack /path/to/your/project/.augment/commands/
```

### Windows

**User-level install** (available in every project):

PowerShell:
```powershell
Copy-Item -Recurse gstack "$env:USERPROFILE\.augment\commands\"
```

Command Prompt:
```cmd
xcopy /E /I gstack "%USERPROFILE%\.augment\commands\gstack"
```

**Workspace-level install** (available only in one project):

PowerShell:
```powershell
Copy-Item -Recurse gstack "C:\path\to\your\project\.augment\commands\"
```

Command Prompt:
```cmd
xcopy /E /I gstack "C:\path\to\your\project\.augment\commands\gstack"
```

If the `.augment\commands\` directory does not yet exist, create it first:
```powershell
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.augment\commands"
```

---

User commands take precedence over workspace commands when names conflict.

In either case, add to `.gitignore` in the target project:
```
.augment/features/
```

Verify (all platforms):
```bash
auggie command list
```
Expected output includes: `gstack:autoplan`, `gstack:careful`, `gstack:cso`, `gstack:document`, `gstack:document-release`, `gstack:freeze`, `gstack:guard`, `gstack:investigate`, `gstack:office-hours`, `gstack:plan-ceo-review`, `gstack:plan-eng-review`, `gstack:qa`, `gstack:retro`, `gstack:review`, `gstack:ship`, `gstack:unfreeze`

---

## How it works

Every command writes a structured markdown file to `.augment/features/<JIRA-KEY>/`.
Commands are independent — run them in any order — and each one reads whatever
already exists in the folder for context.

```
.augment/features/
└── JIRA-999/
    ├── office-hours.md        ← written by gstack:office-hours
    ├── plan-ceo-review.md     ← written by gstack:plan-ceo-review
    ├── plan-eng-review.md     ← written by gstack:plan-eng-review
    ├── test-plan.md           ← written by gstack:plan-eng-review, read by gstack:qa
    ├── review.md              ← written by gstack:review, read by gstack:ship
    ├── investigate.md         ← written by gstack:investigate (appended each run)
    ├── qa-report.md           ← written by gstack:qa
    ├── cso-report.md          ← written by gstack:cso
    ├── ship.md                ← written by gstack:ship
    ├── document-release.md    ← written by gstack:document-release
    └── retro.md               ← written by gstack:retro
```

`gstack:document` reads the whole folder and upserts a single Confluence page —
one page per feature, updated every time you run it.

---

## Command reference

### Planning (fetch Jira at start)

| Command | What it does |
|---------|-------------|
| `/gstack:office-hours JIRA-999` | YC-style reframe. Six forcing questions. Writes `office-hours.md`. |
| `/gstack:plan-ceo-review JIRA-999` | Scope modes, 10-section review, risks. Writes `plan-ceo-review.md`. |
| `/gstack:plan-eng-review JIRA-999` | Architecture diagrams, hard questions, test plan. Writes `plan-eng-review.md` + `test-plan.md`. |
| `/gstack:autoplan JIRA-999` | Runs CEO → Eng automatically, taste gate at end. Writes all three plan files. |

### Implementation (Jira key as label)

| Command | What it does |
|---------|-------------|
| `/gstack:review JIRA-999` | Staff engineer review of `git diff`. Auto-fixes obvious bugs, flags ambiguous ones. Writes `review.md`. |
| `/gstack:investigate JIRA-999 <symptom>` | Root cause debugger. Iron Law: no fix without diagnosis. Appends to `investigate.md`. |
| `/gstack:qa JIRA-999` | Tests against `test-plan.md`. Fixes bugs, generates regression tests. Writes `qa-report.md`. |
| `/gstack:cso JIRA-999` | OWASP Top 10 + STRIDE. 8/10 confidence gate. Writes `cso-report.md`. |

### Shipping

| Command | What it does |
|---------|-------------|
| `/gstack:ship JIRA-999` | Reads review gate from folder. Sync, test, coverage audit, push, open PR. Writes `ship.md`. |
| `/gstack:document-release JIRA-999` | Updates all project docs. Writes `document-release.md`. |
| `/gstack:retro JIRA-999` | Git history retro scoped to feature. Plan vs reality, per-person. Writes `retro.md`. |

### Documentation

| Command | What it does |
|---------|-------------|
| `/gstack:document JIRA-999` | Reads entire feature folder. Synthesises narrative. Upserts Confluence page under J's notebook. |

### Safety (no Jira key needed)

| Command | What it does |
|---------|-------------|
| `/gstack:careful` | Warns before destructive commands for this session. |
| `/gstack:freeze <path>` | Locks file edits to one directory. |
| `/gstack:guard <path>` | careful + freeze combined. |
| `/gstack:unfreeze` | Removes the edit lock. |

---

## Jira integration

Commands in the planning phase (office-hours, plan-ceo-review, plan-eng-review,
autoplan) fetch the Jira issue at the start to read acceptance criteria, description,
and linked Confluence pages.

All other commands use the key as a label and folder name only — no Jira calls.

**Nothing is ever posted to Jira automatically.** The Confluence page is the only
external write, and only when you explicitly run `gstack:document`.

---

## gstack:document — the Confluence upsert

`gstack:document JIRA-999` reads every file in `.augment/features/JIRA-999/` and writes
a single clean narrative page to Confluence:

- **Parent page:** prompted at runtime — you provide the Confluence parent page URL when the command runs
- **Title:** `JIRA-999 — <Jira summary>`
- **Behaviour:** upsert — creates on first run, updates on every subsequent run
- **Content:** a readable narrative covering what was built, why, key decisions, current state, open questions, and learnings — synthesised from the folder, not pasted raw

Run it at any stage — after planning for a progress snapshot, after shipping for a
post-implementation record, after retro for a complete feature history.

---

## Typical sprint

```bash
# Start
auggie command gstack:office-hours JIRA-999

# Plan
auggie command gstack:plan-eng-review JIRA-999

# After building
auggie command gstack:review JIRA-999
auggie command gstack:qa JIRA-999

# Ship
auggie command gstack:ship JIRA-999

# Document
auggie command gstack:document JIRA-999   # → Confluence page upserted
```

---

## What is NOT ported

Browser-dependent gstack commands require Playwright and cannot run as Augment commands:
`/browse`, live `/qa`, `/design-shotgun`, `/design-html`, `/design-consultation`,
`/design-review`, `/benchmark`, `/canary`, `/land-and-deploy`, `/open-gstack-browser`,
`/pair-agent`, `/codex`.

---

## License

MIT. Credits: original gstack by [Garry Tan](https://github.com/garrytan/gstack).
