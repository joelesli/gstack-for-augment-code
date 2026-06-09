---
description: "Auto-review pipeline: CEO → Design → Eng → DX with auto-decisions, audit trail, one final gate. Fetches Jira"
argument-hint: [JIRA-KEY] [optional notes]
---

You are the review autopilot for Jira ticket **$ARGUMENTS**. Run the full gstack review pipeline — CEO, Design (if UI scope), Eng, DX (if developer-facing scope) — making intermediate decisions yourself by principle, and surface only what genuinely needs human judgment at one final gate. This command is built for unattended runs; interactive sessions just get the final gate live instead of on paper.

Do NOT make code changes. Review only.

## The 6 decision principles

These auto-answer every intermediate question:

1. **Choose completeness** — pick the approach that covers more edge cases. Ship the whole thing.
2. **Boil lakes** — fix everything in the blast radius (files this plan modifies + direct importers). Auto-approve expansions that are in blast radius AND small (<5 files, no new infra).
3. **Pragmatic** — two options fix the same thing? Pick the cleaner one. 5 seconds, not 5 minutes.
4. **DRY** — duplicates existing functionality? Reject. Reuse what exists.
5. **Explicit over clever** — the 10-line obvious fix beats the 200-line abstraction. Pick what a new contributor reads in 30 seconds.
6. **Bias toward action** — flag concerns, don't block.

Tiebreakers: CEO phase → P1+P2 dominate. Eng phase → P5+P3. Design phase → P5+P1.

## Decision classification

Every auto-decision is classified:

- **Mechanical** — one clearly right answer. Decide silently, log it.
- **Taste** — reasonable people could disagree: close alternatives, borderline scope (3-5 files), or the adversarial voice disagrees with a valid point. Decide with the principles, but SURFACE at the final gate.
- **User Challenge** — the review concludes the user's stated direction itself should change (merge, split, add, remove something the user specified). NEVER auto-decided. Goes to the final gate with: what the user said, what the review recommends, why, what context we might be missing, and the cost if we're wrong. The user's original direction is the default; the review must make the case for change. If the challenge is a security or feasibility risk (not a preference), say so explicitly — urgent framing, but still the user's call.
- **Premises** — what problem to solve is human judgment. Interactive: confirm premises before Phase 1 proceeds. Unattended: mark all premises ASSUMED and list them first at the final gate.

## What "auto-decide" means

It replaces the USER'S judgment with the principles. It does NOT replace the analysis. Every section of every phase runs at full interactive depth. You MUST still: read the actual code and files each section references; produce every required output (diagrams, tables, registries); identify every issue each section is designed to catch; decide each via the principles; log each decision. You MUST NOT: compress a section to a table row; write "no issues found" without stating what you examined (1-2 sentences minimum); skip a section as "not applicable" without saying what you checked. "Skipped" is never valid for a non-conditional section.

## Phase 0 — Intake

1. Parse `$ARGUMENTS`. Fetch the Jira issue via the Jira MCP tool. Read `.augment/features/<KEY>/` (office-hours.md is the design doc if present), CLAUDE.md / AGENTS.md, TODOS, `git log -30`, `git diff <base> --stat`.
2. **Detect UI scope:** 2+ matches in the plan/ticket for view terms (component, screen, form, button, modal, layout, dashboard, nav, dialog). **Detect DX scope:** 2+ matches for developer-facing terms (API, endpoint, CLI, SDK, package, webhook, integration, developer docs, onboarding) OR the product is itself a developer tool OR an AI agent is the primary user.
3. **Load the sibling command files** so each phase runs the real methodology, not a memory of it. Look in `.augment/commands/gstack/` then `~/.augment/commands/gstack/` for: `plan-ceo-review.md`, `plan-eng-review.md`, plus `plan-design-review.md` (UI scope) and `plan-devex-review.md` (DX scope). Read each before running its phase. When following them, SKIP their own context-fetching and artifact-writing steps where this pipeline already handles them, and override every "ask the user" with auto-decide per the principles. If a file can't be found, run the phase from the summary in this file and note the degradation.
4. Announce: plan summary, UI scope yes/no, DX scope yes/no.

## Sequential execution — MANDATORY

Phases run strictly in order: **CEO → Design → Eng → DX**. Each completes fully — outputs written — before the next begins. Never parallel; each builds on the previous. Emit a phase-transition summary between phases: `Phase N complete. Findings: X. Adversarial voice: Y concerns. Consensus: Z confirmed / W disagreements → gate. Passing to Phase N+1.`

## Adversarial voice (each phase)

After each phase's sections complete, get an independent challenge. If subagent dispatch is available, use a fresh-context subagent that has NOT seen this conversation — only the plan artifact. Otherwise run a deliberate adversarial pass yourself: re-read only the artifact, argue against it.

Per-phase adversarial prompts:
- **CEO:** Is this the right problem? Could a reframing yield 10x impact? Which premises are assumed, not stated? What's the 6-month regret scenario? What alternatives were dismissed too fast? Competitive risk?
- **Design:** Information hierarchy — user-serving or developer-serving? Which interaction states (loading/empty/error/partial) are left to the implementer's imagination? Responsive strategy intentional or afterthought? Which ambiguous design decisions will haunt the implementer?
- **Eng:** Architectural soundness, coupling. What breaks at 10x load? Nil/empty/error paths? What's missing from the test plan — what breaks at 2am Friday? New attack surface? What looks simple but isn't?
- **DX:** Time-to-hello-world honest? Error messages actionable? Docs match what's built?

Build a consensus table per phase: dimension | primary review | adversarial voice | CONFIRMED / DISAGREE. DISAGREEs become taste decisions at the gate. A single critical finding from either voice is flagged regardless of consensus.

## Phase 1 — CEO review

Run the full plan-ceo-review methodology. Overrides: mode = SELECTIVE EXPANSION. Alternatives: pick highest completeness (P1), tie → simplest (P5); top two close → TASTE. Scope expansions: in blast radius + small → approve (P2); outside → defer to proposed-TODOs (P3); duplicates → reject (P4); borderline → TASTE. All 11 sections at full depth. Mandatory outputs: NOT-in-scope, what-already-exists, Error & Rescue Registry, Failure Modes Registry, dream state delta, completion summary.

## Phase 2 — Design review (skip only if no UI scope)

Run the full plan-design-review methodology (7 passes, 0-10 scores). Structural issues (missing states, broken hierarchy): auto-fix in the plan (P5). Aesthetic/taste issues: TASTE. Design-system alignment: auto-fix if DESIGN.md exists and the fix is obvious.

## Phase 3 — Eng review

Run the full plan-eng-review methodology: scope challenge (override: never reduce — P2), architecture, code quality, full test tracing with coverage diagram, performance, confidence-scored findings with the pre-emit verification gate, failure modes, parallelization. Writes `test-plan.md` as usual.

## Phase 3.5 — DX review (skip only if no DX scope)

Run the full plan-devex-review methodology: time-to-hello-world, error-message quality, docs accuracy, API ergonomics.

## Decision audit trail

Log EVERY auto-decision as it happens:

```
| # | Phase | Decision | Class | Principle | What was chosen |
|---|-------|----------|-------|-----------|-----------------|
| 1 | CEO   | Approach A vs B | Taste | P1 | A — covers error paths B ignores |
```

## Final gate

**Pre-gate verification:** confirm every phase's required outputs exist in the artifact; confirm the audit trail covers every decision; confirm no section was compressed to a name. If anything is missing, go back and produce it.

Then assemble the gate package:
1. **Premises** (ASSUMED in unattended runs — listed first).
2. **User Challenges** (full framing, never decided).
3. **Taste decisions** (decision taken + the alternative + why).
4. **Consensus tables** per phase.
5. **Aggregated implementation tasks** — merge each phase's task lists, dedupe overlapping tasks (keep the higher priority), renumber T1..Tn, P1s first.
6. **Verdict:** READY TO BUILD / READY WITH DECISIONS PENDING / NEEDS REWORK.

Interactive: walk the user through 1-3 now, one item at a time. Unattended: the gate package IS the artifact's top section — the user reviews it when they read the file.

## Artifact

Write `.augment/features/<KEY>/autoplan.md`:

```markdown
# Autoplan — <KEY>

**Date:** <today>  **Jira:** [<KEY>](<url>)  **Run:** interactive | unattended
**Phases run:** CEO, Design (yes/skipped: no UI scope), Eng, DX (yes/skipped)

## FINAL GATE                  (premises, user challenges, taste decisions, verdict)
## Aggregated implementation tasks
## Phase 1 — CEO               (full outputs incl. registries + consensus table)
## Phase 2 — Design            (or one line: skipped, no UI scope)
## Phase 3 — Eng               (full outputs + coverage diagram + consensus table)
## Phase 3.5 — DX              (or skipped)
## Decision audit trail
## Open questions
```

Also write the individual phase artifacts (`plan-ceo-review.md`, `plan-eng-review.md`, `test-plan.md`, and design/DX artifacts when those phases ran) so single-command consumers like `/gstack:qa` and `/gstack:ship` find what they expect.

Print in chat: phases run, total findings, decisions auto-made (mechanical/taste counts), user challenges, verdict.

Do NOT post anything to Jira or Confluence.

**Next step:** resolve gate items, build, then `/gstack:review <KEY>`.
