---
description: "Security audit: OWASP Top 10 + STRIDE threat model"
argument-hint: [JIRA-KEY]
---

You are the Chief Security Officer auditing the implementation of **$ARGUMENTS**.

## Step 0 — Read context

Read `.augment/features/$ARGUMENTS/plan-eng-review.md` for the trust boundaries and data flow documented during planning. Security findings should be evaluated against what was intended.

Run `git diff main` — focus the audit on changed files, but also check any authentication, authorisation, or data-handling files they touch.

**Signal gate: only report findings at 8/10 confidence or higher.** If you are not sure it is exploitable, skip it.

## Step 1 — OWASP Top 10

For each finding provide: location, evidence (exact code), exploit scenario, and fix.

**A01 — Broken Access Control**
- Lower-privilege user accessing higher-privilege resource by changing an ID?
- Resource ownership checks server-side or only client-side?
- IDOR patterns?

**A02 — Cryptographic Failures**
- PHI or sensitive data stored/transmitted without encryption?
- Weak hashing (MD5, SHA1 for passwords)?
- Hardcoded secrets or API keys?
- Token comparison not using constant-time?

**A03 — Injection**
- SQL: string concatenation into queries? ORM bypass?
- Command injection: user input to shell?
- Log injection: user input written to logs unsanitised?
- Prompt injection: user input inserted into AI prompts?

**A04 — Insecure Design**
- Business logic flaws: workflow exploitable for unintended gain?
- Missing rate limiting on sensitive endpoints?
- Missing idempotency on payment or write operations?

**A05 — Security Misconfiguration**
- Debug mode in production paths?
- Overly permissive CORS?
- Error messages exposing stack traces to the client?

**A06 — Vulnerable Components**
- Dependencies with known CVEs in `pom.xml` or `packages.config`?

**A07 — Authentication Failures**
- Session tokens in localStorage?
- Missing brute-force protection on login?
- Password reset tokens single-use and expiring?

**A08 — Data Integrity Failures**
- Deserialisation of untrusted data without validation?

**A09 — Logging & Monitoring**
- Authentication failures logged?
- PHI or PII being logged?
- Audit log gaps for sensitive operations?

**A10 — SSRF**
- Endpoint that fetches a URL provided by the user?

## Step 2 — STRIDE on core domain objects

For each core entity changed by **$ARGUMENTS**:

| Threat | Finding |
|--------|---------|
| Spoofing | Can an attacker impersonate another user? |
| Tampering | Can data be modified without detection? |
| Repudiation | Can a user deny an action they took? |
| Information disclosure | Can an attacker read data they shouldn't? |
| Denial of service | Can an attacker make the system unavailable? |
| Elevation of privilege | Can an attacker gain capabilities beyond their role? |

## Step 3 — Write session file

Write `.augment/features/$ARGUMENTS/cso-report.md`:

```markdown
# Security Audit — $ARGUMENTS

**Date:** <today>
**Files scanned:** N
**Findings:** N critical, N high, N medium

## Critical
[C1] <category> — <file:line>
Evidence: <exact code>
Exploit: <how an attacker abuses this>
Fix: <specific change>

## High
...

## Medium
...

## STRIDE summary
| Threat | Status |
|--------|--------|
| Spoofing | CLEAR / FINDING |
...

## False positives excluded
<list of patterns checked and ruled out>
```

Do NOT post anything to Jira or Confluence.
