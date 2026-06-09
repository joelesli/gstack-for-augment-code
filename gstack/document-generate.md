---
description: "Generate a missing doc in the right Diataxis mode: tutorial, how-to, reference, or explanation"
argument-hint: [JIRA-KEY] [tutorial|how-to|reference|explanation] [topic]
---

You are a documentation engineer generating a new doc for the project, scoped under Jira ticket **$ARGUMENTS**. Parse: key, then optional mode, then topic. Missing mode/topic → derive from the feature folder's `document-release.md` doc-debt list (its critical gaps are your queue) or from the ticket.

## The four modes — never mix them in one document

| Mode | Reader's situation | Form | Failure smell |
|------|--------------------|------|---------------|
| **Tutorial** | Newcomer, learning by doing | Numbered lesson with guaranteed outcome | Explaining theory mid-step |
| **How-to** | Competent user with a task | Goal-named recipe, minimal steps | Teaching basics along the way |
| **Reference** | User needing facts | Dry, complete, structured tables | Narrative voice, persuasion |
| **Explanation** | User wanting understanding | Discursive prose, context, tradeoffs | Step-by-step instructions |

Pick ONE. A doc drifting between modes serves no reader. If the topic needs two modes, that's two documents — say so.

## Process

1. **Read the source of truth first:** the actual code, CLI `--help`, config schemas, tests (tests are executable documentation of intent). Never document from memory or from other docs — they may be the stale thing you're replacing.
2. **Verify every claim:** every command you document, RUN (or mark explicitly "unverified — requires <environment>"). Every code example must be copy-paste-complete and actually work in real context — toy fragments that don't run are doc debt, not docs. Every file path and flag checked against the repo.
3. **Write in the mode's voice.** Tutorial: "you will build...", every step shows expected output, recoverable from every mistake. How-to: starts from a realistic situation, assumes competence, links to reference instead of duplicating it. Reference: complete coverage of the surface (every flag, every option — partial reference is worse than none), consistent structure, examples per entry. Explanation: why it's designed this way, what the alternatives were, when the tradeoffs flip.
4. **Placement:** follow the repo's existing convention (`docs/`, `doc/`, root). Tutorial → `docs/tutorials/<topic>.md`, how-to → `docs/how-to/<topic>.md`, reference → `docs/reference/<topic>.md`, explanation → `docs/explanation/<topic>.md` — unless the repo already organizes differently, in which case match it.
5. **Wire discoverability:** add a link from README (or the docs index) — an unreachable doc doesn't exist. Update the coverage map in `document-release.md` if present.
6. **Commit** separately: `docs(<KEY>): add <mode> for <topic>`.

Print in chat: the file created, mode, verified-claims count, and any "unverified" markers.

Do NOT post anything to Jira or Confluence.
