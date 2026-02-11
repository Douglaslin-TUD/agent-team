# Agent Team Interaction Protocols v2.0

**Date:** 2026-02-11
**Based on:** Framework survey, protocol research, production case studies

---

## 1. Workflow State Machine

All agent team work follows this explicit state machine:

```
EXPLORE → DESIGN → [APPROVE] → IMPLEMENT + TEST → VERIFY → REVIEW → [APPROVE] → DOCUMENT → DONE
  W1        W2       Gate-1     W3 (parallel)    W4      W4        Gate-2       W5
```

### States and Ownership

| State | Wave | Agent(s) | Output Artifact |
|-------|------|----------|-----------------|
| EXPLORE | W1 | team-explorer | `exploration-report.md` |
| DESIGN | W2 | team-architect | `design-spec.md` |
| GATE-1 | — | Lead + Human | Approval (plan_approval_response) |
| IMPLEMENT | W3 | team-implementer | Code changes + git commits |
| TEST | W3 | team-tester | Test files + coverage report |
| VERIFY | W4 | team-reviewer | `review-report.md` |
| AUDIT | W4 | team-security-auditor | `security-audit.md` |
| GATE-2 | — | Lead + Human | Approval of merge-readiness |
| DOCUMENT | W5 | team-doc-writer | Updated docs |
| DONE | — | Lead | Summary to user |

### Gate Conditions

**Gate-1 (Design → Implement):**
- [ ] All requirements addressed in design
- [ ] File ownership assigned (no overlaps between implementer and tester)
- [ ] Testing strategy defined
- [ ] Risk flags documented
- [ ] Human has approved the plan (via plan_approval_response)

**Gate-2 (Review → Document):**
- [ ] All tests pass
- [ ] Zero CRITICAL findings from reviewer
- [ ] Zero CRITICAL findings from security auditor
- [ ] All HIGH findings resolved or accepted with justification
- [ ] Lead confirms merge-readiness

---

## 2. Handoff Artifact Format

Every wave transition produces a structured artifact. The receiving agent reads this, not raw conversation history.

### Standard Handoff Template

```markdown
# Handoff: [FROM_AGENT] → [TO_AGENT]

## Task
- **ID:** [task-id]
- **Stage:** [current state in state machine]
- **Summary:** [1-2 sentence description of what was done]

## Key Decisions
- [Decision 1]: [rationale]
- [Decision 2]: [rationale]

## Artifacts Produced
- [file path 1]: [description]
- [file path 2]: [description]

## For Next Agent
- **Your task:** [clear instruction for what the next agent should do]
- **Key constraints:** [must-follow rules]
- **Watch out for:** [known risks or tricky areas]

## Open Questions
- [Any unresolved items the next agent should be aware of]
```

### Per-Transition Specifics

**Explorer → Architect:**
Include: key files with line numbers, existing patterns found, dependencies, architecture observations.

**Architect → Implementer:**
Include: component design, file ownership assignments, API contracts, error handling strategy, testing strategy (for tester).

**Architect → Tester:**
Include: testing strategy, test scenarios (happy/error/edge), coverage targets, mocking strategy.

**Implementer → Reviewer:**
Include: files changed (with descriptions), test results, self-assessment of risk areas, any deviations from design.

**Implementer → Security Auditor:**
Include: files changed, new external inputs, auth changes, data handling changes.

**Reviewer/Auditor → Implementer (fixes):**
Include: findings list (file:line), severity, concrete fix for each.

---

## 3. Message Envelope Format

All SendMessage between agents should follow this structure:

```
## [ACTION_TYPE]: [Brief Title]

**From:** [agent-name]
**Stage:** [current state]
**Task:** [task-id or description]

### Content
[The actual message content]

### Action Required
[NONE | REVIEW | FIX | APPROVE | BLOCK]
[Specific ask if action needed]
```

### Action Types

| Type | When | Example |
|------|------|---------|
| `STATUS` | Progress update, no action needed | "Implementation 70% complete, 12/18 tests passing" |
| `HANDOFF` | Completing stage, passing to next agent | "Design complete, handing off to implementer" |
| `FINDING` | Issue discovered during review/audit | "CRITICAL: SQL injection at auth.py:45" |
| `QUESTION` | Need clarification from another agent | "Should we use bcrypt or argon2 for hashing?" |
| `BLOCKER` | Cannot proceed, need help | "Test infrastructure not set up, blocking test writing" |
| `ESCALATE` | Exceeded retry cap, need lead intervention | "3 fix attempts failed, need architecture change" |

---

## 4. Error Escalation Ladder

Every agent follows this escalation path when encountering failures:

```
Level 1: RETRY (same agent, modified approach)
  - Max 3 attempts
  - Change strategy between attempts, don't repeat same approach
  - Log what was tried and why it failed
  ↓ if all retries fail
Level 2: FALLBACK (simplify the approach)
  - Decompose into smaller tasks
  - Try a simpler implementation
  - Skip non-essential parts
  ↓ if fallback doesn't work
Level 3: ESCALATE (to team lead)
  - Send ESCALATE message with full context
  - Include: what was tried, what failed, what you think the root cause is
  - Lead decides: reassign, redesign, or involve human
  ↓ if lead cannot resolve
Level 4: ABORT (save state, halt)
  - Commit any partial work to git
  - Write a failure report with full context
  - Notify human for manual intervention
```

### Rules
- **Never retry state-modifying operations** (file writes, git pushes) without verifying state first
- **Never exceed 3 retries** — escalate instead
- **Always log attempts** — the next agent (or human) needs to know what was tried
- **Git commits as checkpoints** — commit working state before risky operations

---

## 5. Tiered Autonomy Model

### Tier 3: Auto-Proceed (no approval needed)
- Read-only operations (file reads, grep, glob, git log)
- Running tests
- Linting and formatting
- Writing handoff artifacts

### Tier 2: Lead Approval (team lead decides)
- File modifications (implementation)
- Git commits
- Task reassignment
- Spawning additional agents

### Tier 1: Human Approval (user must approve)
- Architecture decisions (via plan mode)
- Merge to main branch
- Deployment
- External-facing API changes
- Deleting files or reverting commits

---

## 6. Anti-Pattern Detection

Every agent should recognize and flag these patterns:

| Pattern | Signal | Action |
|---------|--------|--------|
| **Agent Drift** | Working on something not in the task description | Stop, re-read task, ask lead if unclear |
| **Echo Chamber** | Agreeing with previous agent without independent verification | Verify independently, challenge if evidence differs |
| **Context Amnesia** | Making decisions that contradict earlier decisions | Re-read handoff artifacts, ask lead for clarification |
| **Over-engineering** | Adding abstractions, patterns, or features not in design | Remove, follow KISS, only implement what's specified |
| **Infinite Loop** | Same fix attempted more than twice | Escalate to Level 2 (fallback) or Level 3 (lead) |
| **Scope Creep** | "While I'm here, I'll also fix..." | Stop. Only do what's assigned. Create a new task for the other thing. |

---

## 7. Context Passing Rules

### Minimum Context for Every Agent Spawn

When the lead spawns a teammate via Task, always include:
1. **Task description** — what to do
2. **Current workflow state** — where we are in the state machine
3. **Relevant handoff artifact** — from the previous wave
4. **File ownership** — which files this agent may modify
5. **Iteration budget** — max turns/retries allowed

### Context Optimization

- **Pass pointers, not content** — "Read the design at docs/design-spec.md" instead of pasting the entire design
- **Summarize decisions** — "We chose bcrypt because X" instead of the full discussion
- **Exclude irrelevant history** — tester doesn't need explorer's raw search results

---

## 8. Coordination Tax Budget

Target: **< 20% of total tokens** spent on inter-agent coordination.

### What counts as coordination:
- SendMessage tokens
- Handoff artifact writing/reading
- Status updates
- Re-reads of other agents' output

### What counts as productive work:
- Code writing
- Test writing
- Code review analysis
- Security analysis
- Documentation writing

### How to optimize:
- Use structured messages (less tokens than free-form)
- Pass minimal context (pointers not content)
- Avoid unnecessary status updates
- Batch findings instead of individual messages
