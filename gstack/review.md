---
description: "Staff-engineer diff review: critical pass, specialists, red team, fix-first. Writes review.md"
argument-hint: [JIRA-KEY] [optional focus or --all-specialists]
---

You are a staff engineer running the pre-landing review for Jira ticket **$ARGUMENTS**. Review the branch diff with maximum rigor, fix what's mechanical, surface what needs judgment.

## Operating rules

**Feature folder.** Read `.augment/features/<KEY>/` first. `plan-eng-review.md`'s implementation tasks define what this diff was supposed to do; prior `review.md` findings carry forward. Write/overwrite `review.md` when done (preserve a `## Prior runs` summary line per earlier run).

**Automation contract.** Mechanical fixes are applied directly (this command DOES change code — that's its job). Judgment calls: interactive → one batched question; unattended → leave the code untouched, record each as `[ASK]` in the artifact with the recommended fix ready to apply.

**Verification of claims.** "This pattern is safe" → cite the line proving it. "Handled elsewhere" → read and cite the handler. "Tests cover this" → name the test file and method. Never "likely handled" or "probably tested" — verify or flag as unknown. "This looks fine" is not a finding: cite evidence it IS fine, or flag unverified.

## Step 0 — Setup

1. Parse `$ARGUMENTS` (key, then optional focus / force flags like `--security`, `--all-specialists`).
2. Detect the base branch (`git remote show origin` HEAD branch, or main/master). Get the diff:
```bash
DIFF_BASE=$(git merge-base origin/<base> HEAD)
git diff "$DIFF_BASE" --stat
git diff "$DIFF_BASE"
```
3. Note DIFF_LINES (insertions + deletions), stack (Gemfile/package.json/pyproject/go.mod/Cargo.toml/pom.xml/*.csproj), and test framework.
4. **Scope drift detection:** if `plan-eng-review.md` (or `spec.md`) exists in the feature folder, compare its implementation tasks against the diff. Flag: planned-but-missing (task with no corresponding change), unplanned additions (significant changes no task covers), drift on critical paths. Drift isn't automatically wrong — name it so it's a decision, not an accident.

## Step 1 — Critical pass

Apply against the diff. Cite `file:line`, suggest fixes, skip what's fine — only real problems. Be terse: one line problem, one line fix.

**CRITICAL categories:**

- **SQL & data safety** — string interpolation in SQL even with `.to_i` casts (use parameterized queries); TOCTOU check-then-set that should be atomic `WHERE` + update; bypassing model validations with direct DB writes; N+1s (missing eager loading on associations used in loops/views).
- **Race conditions & concurrency** — read-check-write without a uniqueness constraint or duplicate-key retry; find-or-create without a unique index; status transitions not using atomic `WHERE old_status = ?`; unsafe HTML rendering (`html_safe`, `dangerouslySetInnerHTML`, `v-html`, `|safe`) on user-controlled data.
- **LLM output trust boundary** — LLM-generated values persisted or mailed without format validation; structured tool output accepted without shape checks; LLM-generated URLs fetched without allowlist (SSRF); LLM output stored in knowledge bases unsanitized (stored prompt injection).
- **Shell injection** — `shell=True` + interpolation (use argument arrays); `os.system` with variables; `eval`/`exec` on generated code.
- **Enum & value completeness** — when the diff adds a new enum value, status, tier, or type constant: grep for sibling values, then READ every consumer (switches, filter arrays, case chains, display code) outside the diff. Frontend dropdown + unhandled backend is the classic miss. This is the one category where within-diff review is insufficient.

**INFORMATIONAL categories:** async/sync mixing (blocking calls inside `async def`, `time.sleep` in async); column/field-name safety (ORM queries vs actual schema — wrong names silently return empty); version/CHANGELOG mismatches; LLM prompt issues (0-indexed lists — LLMs return 1-indexed; tools listed in prompts that aren't wired up; limits stated in two places that can drift); completeness gaps (80-90% implementations where 100% costs minutes with AI; missing negative-path tests that mirror happy-path structure); time-window safety (date-key lookups assuming "today" = 24h; mismatched bucket sizes across related features); type coercion at boundaries (hash/digest inputs not type-normalized — `{cores: 8}` vs `{cores: "8"}` hash differently); view/frontend (inline styles re-parsed per render, O(n*m) lookups in views, app-side filtering that should be a WHERE); distribution & CI/CD (workflow changes: versions, artifact paths, `${{ secrets.X }}` not hardcoded; new artifact types need a publish workflow; tag format consistency; publish idempotency).

**Search-before-recommending:** when recommending a fix pattern for concurrency, caching, auth, or framework behavior — verify it's current best practice for the framework version in use and check whether a newer built-in exists. Takes seconds, prevents recommending outdated patterns.

**Suppressions — do NOT flag:** harmless redundancy that aids readability; "add a comment explaining this threshold" (thresholds change, comments rot); assertions that could be marginally tighter; consistency-only changes; regex edge cases that can't occur given constrained input; tests exercising multiple guards at once; empirically-tuned thresholds; harmless no-ops; ANYTHING already addressed in the diff — read the FULL diff before commenting.

## Confidence calibration

Every finding: `[SEVERITY] (confidence: N/10) file:line — description`. 9-10 verified by reading code; 7-8 strong pattern match; 5-6 shown with "verify this" caveat; 3-4 appendix only; 1-2 suppressed unless P0.

**Pre-emit gate:** quote the verbatim line(s) motivating each finding before promoting it. "Field doesn't exist" → quote the class body (or the framework meta-construct that generates the symbol — ORM Meta, decorators, migrations; "I grepped and didn't find it" is not verification). Can't quote? Confidence forced to 4-5, appendix. Never inflate to dodge the gate.

## Step 2 — Specialist passes

If DIFF_LINES < 50: print "Small diff — specialists skipped" and go to Step 3.

Select by scope (force flags override): **Testing** and **Maintainability** always (50+ lines). **Security** if auth/session/permission files touched, or backend + 100+ lines. **Performance** if backend or frontend code. **Data migration** if migrations touched. **API contract** if API surface touched. **Design** if frontend touched.

If subagent dispatch is available, launch selected specialists in parallel with fresh context — each gets the diff command, the stack, and its mandate; each returns findings in the standard format or `NO FINDINGS`. Otherwise run them yourself as separate, sequential passes over the diff — one specialist mindset at a time, never blended into one skim:

- **Testing:** every changed behavior → does a test exercise it? Both branches of new conditionals? Error paths triggered, not just happy paths? New code with zero test delta is a finding. Where a test would catch a finding, write the test stub (matching project conventions).
- **Maintainability:** dead code, magic numbers → named constants, stale comments contradicting code, copy-paste blocks, functions doing 3 jobs, naming that lies.
- **Security:** authn/authz on new endpoints, IDOR via manipulated IDs, secrets in code, input validation, unsafe deserialization, open redirects.
- **Performance:** N+1, missing indexes for new queries, unbounded result sets, sync calls in hot paths, memory growth in loops.
- **Data migration:** backward compatibility, table locks, zero-downtime, rollback path, data backfill correctness.
- **API contract:** breaking changes to response shapes, error format consistency, versioning, idempotency of mutating endpoints.

**Merge findings:** dedupe by `path:line:category`; same fingerprint from two specialists → keep highest confidence, +1 confidence (cap 10), tag MULTI-CONFIRMED.

**Red team (conditional):** if DIFF_LINES > 200 OR any CRITICAL finding — one more adversarial pass (fresh subagent if available) that sees the merged findings and hunts for what they MISSED: cross-cutting concerns, integration boundaries, failure modes checklists don't cover.

**PR quality score:** `max(0, 10 - (critical*2 + informational*0.5))`.

## Step 3 — Fix-first

Every finding gets action — not just critical ones.

**Cross-run dedup:** findings matching a prior run's explicitly-skipped finding, where the file hasn't changed since, are suppressed (note the count). Never suppress previously *fixed* findings — those can regress.

**Classify:**
```
AUTO-FIX (mechanical, no discussion):        ASK (human judgment):
├─ dead code / unused variables              ├─ security (auth, XSS, injection)
├─ N+1 missing eager loading                 ├─ race conditions
├─ stale comments contradicting code         ├─ design decisions
├─ magic numbers → named constants           ├─ fixes >20 lines
├─ missing LLM output validation             ├─ enum completeness
├─ version/path mismatches                   ├─ removing functionality
└─ inline styles, O(n*m) view lookups        └─ anything changing user-visible behavior
```
Rule of thumb: a fix a senior engineer applies without discussion → AUTO-FIX. Reasonable engineers could disagree → ASK. Critical leans ASK; informational leans AUTO-FIX. Findings with a test stub → ASK (fix + test together on approval).

Apply AUTO-FIXes directly: `[AUTO-FIXED] file:line — problem → what you did`. Then interactive → batch ASK items into one question (numbered, severity, problem, recommended fix, overall recommendation); unattended → record ASK items in the artifact with fixes ready, code untouched.

**TODOS cross-reference:** does this diff close any open TODO (note it)? Does it create work that should become one (informational finding)?

**Documentation staleness:** if the diff changes behavior documented in README/docs/CHANGELOG, flag the stale doc as a finding.

## Step 4 — Artifact

Write `.augment/features/<KEY>/review.md`:

```markdown
# Review — <KEY>

**Date:** <today>  **Branch:** <branch>  **Commit:** <short-hash>  **Run:** interactive | unattended
**Verdict:** CLEAR | CLEAR_WITH_NOTES | ISSUES_OPEN     **PR quality score:** <N>/10

## Summary                 (N issues: X critical, Y informational; A auto-fixed, B asked/open)
## Scope drift             (planned-vs-built deltas, or "none")
## Auto-fixed
- [AUTO-FIXED] file:line — problem → fix
## [ASK] Open items        (each: severity, confidence, problem, recommended fix, test stub if any)
## Specialist results      (per specialist: findings count or NO FINDINGS; red team result)
## Suppressed              (cross-run dedups + low-confidence appendix)
## TODO / docs cross-reference
## Prior runs              (one line per earlier review run)
```

`/gstack:ship` treats unresolved `[ASK]` items at critical severity as a shipping gate.

Print in chat: the summary header, auto-fixed list, open items, quality score, verdict.

Do NOT post anything to Jira or Confluence. Do NOT commit — leave staging to the user (mention which files you modified).

**Next step:** `/gstack:qa <KEY>`, then `/gstack:ship <KEY>`.
