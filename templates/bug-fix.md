# Workflow Template: Bug Fix

**When to use:** Fixing a reported bug, regression, or failing test.
**When NOT to use single-agent:** When the bug spans multiple subsystems or the root cause is unclear.
**When to use single-agent:** Bug is localized (1-2 files), root cause is obvious, fix is straightforward.

---

## Wave Sequence (Shortened Pipeline)

```
W1: REPRODUCE + DIAGNOSE  →  W2: FIX  →  W3: VERIFY  →  DONE
```

Skipped waves compared to feature development:
- No DESIGN wave (the fix is guided by the diagnosis, not a new architecture)
- No DOCUMENT wave (unless the bug reveals a doc gap)
- Gate-1 is replaced by diagnosis confirmation

## Agents and Responsibilities

| Wave | Agent | Task |
|------|-------|------|
| W1 | team-explorer | Reproduce the bug, trace root cause, produce diagnosis |
| — | Lead | Confirm diagnosis before proceeding (lightweight gate) |
| W2a | team-implementer | Write the fix |
| W2b | team-tester | Write regression test (parallel with W2a) |
| W3 | team-reviewer | Verify fix correctness and no regressions |

For simple bugs, collapse W2a + W2b into a single implementer who writes both fix and test.

## Single-Agent vs. Team Decision

Use a **single agent** when:
- Root cause is already known or strongly suspected
- Fix touches 1-2 files
- Existing tests cover the area (just need the fix)
- No security implications

Use the **team** when:
- Root cause is unknown and requires exploration
- Bug spans multiple modules or layers
- Fix has security implications (need auditor)
- Regression risk is high (need dedicated tester)

## Wave Details

### W1: Reproduce and Diagnose (team-explorer)

**Input:** Bug report (error message, steps to reproduce, affected version).
**Output:** `diagnosis.md` containing:
- Reproduction steps (confirmed working)
- Root cause analysis (file:line, what goes wrong, why)
- Scope of impact (what else might be affected)
- Proposed fix approach (brief, not a full design)

**Key actions:**
1. Read the bug report and identify the failure mode
2. Find the relevant code paths
3. Write or run a minimal reproduction (failing test preferred)
4. Trace the root cause to a specific location
5. Document the diagnosis

**Budget:** Max 15 minutes / 50k tokens.

### Lightweight Gate: Diagnosis Confirmation

The lead reviews the diagnosis and either:
- **Approves:** Proceeds to W2 with the proposed fix approach
- **Redirects:** Asks explorer to investigate further or try a different angle
- **Escalates:** Involves human if the diagnosis is uncertain or risky

### W2: Fix + Regression Test (parallel)

**Implementer input:** Diagnosis with root cause and proposed fix.
**Tester input:** Diagnosis with reproduction steps.

**Implementer actions:**
1. Apply the minimal fix (resist urge to refactor surrounding code)
2. Run existing tests to confirm no regressions
3. Self-review the change

**Tester actions:**
1. Write a regression test that fails WITHOUT the fix
2. Confirm the test passes WITH the fix
3. Add edge case tests if the diagnosis reveals related risks

**Budget:** Max 20 minutes / 60k tokens each.

### W3: Verify (team-reviewer)

**Input:** Fix diff, test results, diagnosis.
**Checks:**
- Does the fix address the root cause (not just symptoms)?
- Are there regressions in the test suite?
- Is the regression test meaningful (would it catch a revert)?
- Any security implications?

**Budget:** Max 10 minutes / 30k tokens.

---

## Escalation Paths

| Situation | Action |
|-----------|--------|
| Cannot reproduce | Ask lead for more context, check environment differences |
| Multiple possible root causes | Diagnose each, present options to lead for decision |
| Fix breaks other tests | Investigate if the other tests were wrong or if the fix is too broad |
| Fix is too large / risky | Escalate to lead; may need to convert to a feature-development workflow |

## Example: Login Returns 500 on Valid Credentials

**W1 Explorer:**
- Reproduces: `POST /login` with valid creds returns 500
- Traces: `src/auth/login.ts:42` calls `user.getSession()` which throws when session table has a null `expires_at`
- Root cause: migration `003_add_session_expiry.sql` allows NULL but code assumes non-null
- Proposed fix: add null check in `getSession()`, default to 24h expiry

**Lead confirms** the diagnosis and fix approach.

**W2 Implementer:** Adds null-coalescing in `getSession()` (3-line change).
**W2 Tester:** Writes test: insert user with null `expires_at`, call login, assert 200.

**W3 Reviewer:** Confirms fix is minimal, regression test is meaningful, no security issue.
