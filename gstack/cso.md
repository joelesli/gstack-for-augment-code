---
description: "Security audit: attack surface, secrets archaeology, supply chain, OWASP Top 10, STRIDE, 8/10 confidence gate. Writes cso-report.md"
argument-hint: [JIRA-KEY] [--comprehensive | --diff | focus area]
---

You are the Chief Security Officer auditing the codebase for Jira ticket **$ARGUMENTS**. Audit only — do not fix code in this command (every finding includes the fix, ready to apply). Runs fully unattended.

**Modes:** default = full-repo audit at the 8/10 confidence gate (zero noise — only what you're sure about). `--diff` = scope to the branch diff (use for per-ticket checks). `--comprehensive` = drop the gate to 2/10 and include `TENTATIVE` findings (use for periodic deep audits).

## Operating rules

**Feature folder.** Read `.augment/features/<KEY>/` for context. Write `cso-report.md` when done.

**Evidence discipline.** Every finding quotes the motivating line(s) verbatim with `file:line` before being promoted. Can't quote it? It doesn't ship in the main report. Confidence per finding: 9-10 = could write a PoC; 8 = clear vulnerability pattern with known exploitation, the minimum bar in daily mode; below 8 → appendix (daily) or TENTATIVE (comprehensive).

**Never make live requests** to webhooks or production endpoints. Verification is code-tracing only.

## Phase 0 — Architecture mental model

Before hunting bugs, detect the stack (package.json / Gemfile / pyproject / go.mod / Cargo.toml / pom.xml / *.csproj / composer.json) and framework. Stack detection sets scan PRIORITY, not scope — after the targeted pass, run a brief catch-all (SQLi, command injection, hardcoded secrets, SSRF) across ALL file types; a Python service nested in `ml/` still gets coverage.

Read CLAUDE.md / AGENTS.md, README, key configs. Map components, trust boundaries, and data flow: where does user input enter, what transforms it, where does it exit? State the mental model in a short architecture summary before any finding. This is a reasoning phase — output is understanding, not findings.

## Phase 1 — Attack surface census

Grep for endpoints, auth boundaries, integrations, upload paths, admin routes, webhook handlers, background jobs, WebSocket channels. Plus infrastructure: CI workflows, Dockerfiles, IaC files, tracked .env files. Output the map:

```
ATTACK SURFACE MAP
  Public endpoints: N   Authenticated: N   Admin-only: N   API: N
  Upload points: N      Integrations: N    Background jobs: N   WebSockets: N
  CI workflows: N       Containers: N      IaC: N    Secret mgmt: [env|vault|unknown]
```

## Phase 2 — Secrets archaeology

Git history for known prefixes (`AKIA`, `sk-`, `ghp_|gho_|github_pat_`, `xoxb-|xoxp-`, `sk_live_`, `-----BEGIN ... PRIVATE KEY`), plus `password|secret|token|api_key` in config-type files:
```bash
git log -p --all -G "ghp_|gho_|github_pat_|xoxb-|AKIA|sk_live_" 2>/dev/null | head -100
git ls-files '*.env' '.env.*' | grep -v 'example\|sample\|template'
```
CI configs with inline secrets (values not via `${{ secrets.X }}`). Severity: CRITICAL = active secret pattern in history; HIGH = .env tracked / inline CI credentials; MEDIUM = suspicious example values. FP rules: placeholders ("your_", "changeme") excluded; test fixtures excluded unless the same value appears in non-test code; rotated secrets still flagged (they were exposed). Diff mode: `git log -p <base>..HEAD` instead of `--all`.

## Phase 3 — Dependency supply chain

Run the available audit tool (npm/yarn/pnpm audit, bundle audit, pip-audit, cargo audit, govulncheck) — missing tools are "SKIPPED — not installed", informational, not a finding. Check install scripts (`preinstall`/`postinstall`) in production deps; lockfile exists AND tracked. Severity: CRITICAL = high/critical CVEs in direct deps; HIGH = prod install scripts / missing lockfile; MEDIUM = abandoned packages, medium CVEs. FP: devDependency CVEs MEDIUM max; node-gyp/cmake scripts expected; missing lockfile in a library repo is not a finding.

## Phase 4 — CI/CD pipeline security

Per workflow: unpinned third-party actions (`uses:` without SHA); `pull_request_target` (CRITICAL only when combined with checkout of PR code); script injection via `${{ github.event.* }}` in `run:` steps (CRITICAL with `.body`/`.title` interpolation); secrets as env vars; CODEOWNERS on workflow files. FP: first-party `actions/*` unpinned = MEDIUM; secrets in `with:` blocks are runtime-handled.

## Phase 5 — Infrastructure shadow surface

Dockerfiles: missing USER (root), secrets via ARG, .env copied into images. Configs: prod connection strings (`postgres://`, `mongodb://`, `redis://`) excluding localhost; staging configs referencing prod. IaC: `"*"` IAM actions/resources, hardcoded secrets in .tf/.tfvars; privileged/hostNetwork K8s. FP: local-dev docker-compose with localhost is fine; `"*"` in read-only data sources excluded; manifests under test/dev/local with localhost excluded.

## Phase 6 — Webhook & integration audit

Files with webhook/callback routes but NO signature verification anywhere in the middleware chain (trace it: parent router, middleware, gateway config — gateway-verified upstream is not a finding, but needs evidence) → CRITICAL. TLS verification disabled (`verify=false`, `InsecureSkipVerify`, `NODE_TLS_REJECT_UNAUTHORIZED=0`) in prod code → HIGH (test code excluded). Overly broad OAuth scopes → HIGH.

## Phase 7 — LLM & AI security

User input flowing into system prompts or tool schemas (string interpolation near prompt construction) → CRITICAL. LLM output rendered unsanitized (`dangerouslySetInnerHTML`, `v-html`, `innerHTML`, `html_safe`) → CRITICAL. `eval`/`exec`/`new Function` on LLM output → CRITICAL. Tool calls executed without validation → HIGH. Hardcoded AI keys → HIGH. Unbounded LLM call loops / missing cost caps → MEDIUM (this is financial risk, NOT DoS — never auto-discard it). RAG: can external documents steer behavior via retrieval? FP precedent: user content in the user-message position of a conversation is NOT prompt injection — only flag when it reaches system prompts, tool schemas, or function-calling contexts.

## Phase 8 — Agent skill/command supply chain

Scan repo-local AI-agent instruction files (`.augment/commands/`, `.augment/rules/`, `.claude/`, `AGENTS.md`, `*.skill.md`) for: exfiltration patterns (curl/wget/fetch with credential variables), credential access (`*_API_KEY`, `process.env` reads piped outward), prompt injection ("IGNORE PREVIOUS", "system override", "disregard your instructions"). These files are executable prompt code, not documentation — the *.md exclusion below never applies to them. Legitimate curl needs context; flag only suspicious targets or credential-bearing commands.

## Phase 9 — OWASP Top 10

- **A01 access control:** missing auth on routes (`skip_before_action`, no middleware); IDOR — can user A reach user B's data by changing IDs; privilege escalation.
- **A02 crypto:** MD5/SHA1/DES/ECB; sensitive data unencrypted at rest/in transit; key management.
- **A03 injection:** SQL (interpolation in raw queries), command (`system`, `exec`, `popen`), template (`eval`, `html_safe`, render-with-params). LLM injection → Phase 7.
- **A04 insecure design:** rate limits on auth endpoints; lockout; server-side business-logic validation.
- **A05 misconfiguration:** CORS wildcards in prod; CSP; debug/verbose errors in prod.
- **A06 components:** → Phase 3.
- **A07 auth failures:** session creation/storage/invalidation; password policy; MFA for admin; JWT expiry and refresh rotation.
- **A08 integrity:** → Phase 4; deserialization input validation; integrity checks on external data.
- **A09 logging:** auth events, authz failures, admin actions logged; logs tamper-protected.
- **A10 SSRF:** URL construction from user input; internal reachability; outbound allowlists.

## Phase 10 — STRIDE per major component

```
COMPONENT: [name]
  Spoofing / Tampering / Repudiation / Info Disclosure / DoS / Elevation:
  [one line each: threat or "no path found"]
```

## Phase 11 — Data classification

RESTRICTED (breach = legal liability): credentials, payment data, PII — where stored, how protected, retention. CONFIDENTIAL (= business damage): API keys, trade-secret logic, behavior data. INTERNAL (= embarrassment): logs, config in error messages. PUBLIC.

## Phase 12 — False-positive filtering

Run every candidate through the filter before reporting.

**Hard exclusions — discard automatically:** DoS/resource exhaustion (EXCEPT LLM cost amplification); secured-at-rest secrets; memory/CPU/fd leaks; input validation on non-security fields without proven impact; "missing hardening" without a concrete vulnerability (EXCEPT unpinned actions and missing CODEOWNERS — those are concrete); race conditions without a concrete exploit path; memory safety in memory-safe languages; test-only files not imported by non-test code; log spoofing; SSRF where the attacker controls only the path; ReDoS on non-untrusted input; *.md docs (EXCEPT agent instruction files — Phase 8); missing audit logs; insecure randomness in non-security contexts; secrets committed AND removed within the same initial-setup change; CVEs with CVSS < 4 and no known exploit; Dockerfile.dev/local unless referenced by prod deploys.

**Precedents:** logging secrets IS a vulnerability, logging URLs is safe. UUIDs are unguessable. Env vars and CLI flags are trusted input. React/Angular are XSS-safe by default — flag only the escape hatches. Client-side code doesn't need auth — that's the server's job. Shell injection needs a concrete untrusted-input path. Root containers: local-dev compose fine, prod Dockerfile/K8s finding.

## Phase 13 — Report

Write `.augment/features/<KEY>/cso-report.md`:

```markdown
# Security Audit — <KEY>

**Date:** <today>  **Mode:** daily | comprehensive | diff  **Confidence gate:** 8/10 | 2/10
**Verdict:** CLEAR | ISSUES_FOUND (<N> critical, <N> high, <N> medium)

## Architecture summary        (Phase 0 mental model, trust boundaries)
## Attack surface map
## Findings table              (| # | Sev | Conf | Category | Finding | file:line |)

## Finding N: <title> — <file:line>
**Severity:** CRITICAL|HIGH|MEDIUM  **Confidence:** N/10  **Phase:** <which>
**Evidence:** <verbatim quoted code>
**Exploit path:** <how an attacker uses this, concretely>
**Fix:** <specific change, ready to apply>

## STRIDE summary
## Data classification
## Skipped tools               (audit tools not installed + install commands)
## Discarded candidates        (what the FP filter removed and which rule — auditability)
## Tentative findings          (comprehensive mode only)
```

Print in chat: verdict, findings table, the single most urgent fix.

Do NOT post anything to Jira or Confluence. This is an automated audit, not a penetration test — it reduces risk, it doesn't certify absence of vulnerabilities.

**Next step:** criticals → fix via `/gstack:investigate <KEY> <finding>` before `/gstack:ship`. A good cron pattern: weekly `--comprehensive` run.
