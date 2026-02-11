# Gap Analysis: Current Agent Team vs. Research Best Practices

**Date:** 2026-02-11
**Baseline:** `~/.claude/agents/team-*.md` (7 agents)

---

## Executive Summary

The current 7-agent team has a solid foundation: hub-and-spoke topology (good), specialized roles (good), read-only protection for reviewers (good), and plan mode for architect (good). However, the agents lack the **interaction protocols** that research shows are critical for production-grade performance. The gaps are not in *what* the agents do, but in *how they coordinate*.

---

## Gap 1: No Structured Handoff Protocol

**Current state:** Agents have no explicit instructions on what to include when handing off work. The team lead passes context through Task prompts, but the format is ad hoc.

**Research says:** "Most 'agent failures' are actually orchestration and context-transfer issues." (Galileo) Structured handoff artifacts with schemas reduce context loss — the #1 cause of multi-agent failure.

**Impact:** HIGH — Context loss between waves causes rework, drift, and compounding errors.

---

## Gap 2: No Quality Gate Criteria

**Current state:** Wave transitions are implicit. There are no concrete checklists for what must be true before moving from design → implementation → verification → review.

**Research says:** Quality gates should combine automated checks (tests pass, linter clean) with manual checks (human approval for high-stakes changes). Each transition needs explicit criteria.

**Impact:** HIGH — Without gates, poor output propagates downstream, amplifying errors (up to 17.2x per the "Bag of Agents" research).

---

## Gap 3: No Error Escalation Ladder

**Current state:** Only team-implementer has a retry mechanism (max 3 iterations). Other agents have no defined failure recovery path.

**Research says:** Best practice is a 4-level escalation ladder: Retry (same agent) → Fallback (simpler approach) → Escalate (to lead/human) → Abort (save state, halt). Never retry state-modifying operations blindly.

**Impact:** MEDIUM — Agent failures currently result in either silent degradation or complete halt.

---

## Gap 4: No Message Format Standard

**Current state:** SendMessage content is free-form natural language. No schema, no required fields, no validation.

**Research says:** Structured message envelopes (sender, task_id, stage, action, content) reduce ambiguity and enable audit trails. "Make handoffs explicit, structured, and versioned." (Skywork AI)

**Impact:** MEDIUM — Free-form messages lose structure and are harder to parse/act on reliably.

---

## Gap 5: Incomplete Alignment Checkpoints

**Current state:** Only team-architect has `permissionMode: plan` (requires lead approval). Other agents auto-proceed without checkpoints.

**Research says:** A tiered autonomy model with 5-7 checkpoints per workflow. Human approval at: plan approval, code review, and release. Auto-proceed at: verification (if automated gates pass).

**Impact:** HIGH — Missing checkpoints allow agents to diverge without correction.

---

## Gap 6: No Workflow State Machine

**Current state:** The "wave" concept is documented in comments but not formalized in agent prompts. Agents don't know what state the workflow is in or what transitions are valid.

**Research says:** An explicit state machine (SPEC → PLAN → IMPLEMENT → VERIFY → REVIEW → RELEASE) with defined transitions and gate conditions is the industry standard.

**Impact:** MEDIUM — Without explicit states, coordination relies entirely on the lead's ad-hoc management.

---

## Gap 7: Missing Anti-Pattern Detection

**Current state:** No agent has instructions for recognizing failure modes like drift, echo chamber, infinite loops, or context amnesia.

**Research says:** MAST taxonomy identifies 14 failure modes with 41-86.7% failure rates across frameworks. Agents should recognize and flag these patterns.

**Impact:** MEDIUM — Agents cannot self-correct from failure modes they don't know to look for.

---

## Gap 8: No Context Passing Minimum

**Current state:** When spawning agents via Task, the lead decides ad hoc what context to include. There's no defined minimum.

**Research says:** Each handoff should include: task ID, current stage, completed work summary, pending items, key decisions made, and pointers to full artifacts. "Narrative casting" — reframing context for the receiving agent — prevents hallucination.

**Impact:** HIGH — Insufficient context causes agents to make assumptions or hallucinate.

---

## Gap 9: No Coordination Tax Tracking

**Current state:** No mechanism to track how much of the total token budget goes to inter-agent coordination vs. actual productive work.

**Research says:** Coordination overhead should be < 20% of total tokens. Track and optimize this ratio. "Prefer state-delta communication over full-history passing."

**Impact:** LOW — Invisible cost that compounds over large projects.

---

## Gap 10: Inconsistent Iteration Caps

**Current state:** team-implementer has maxTurns: 50, team-architect: 40, others: 30. Only implementer has an explicit "max 3 retry" in the prompt.

**Research says:** Every agent should have explicit iteration caps with escalation after the cap. The C compiler project succeeded partly because of progress-checking mechanisms.

**Impact:** MEDIUM — Agents without caps can spin indefinitely on difficult tasks.

---

## Summary Table

| Gap | Current | Best Practice | Impact | Effort |
|-----|---------|---------------|--------|--------|
| Structured handoffs | None | Schema-validated artifacts | HIGH | Medium |
| Quality gates | None | Checklist per transition | HIGH | Medium |
| Error escalation | Partial (implementer only) | 4-level ladder for all agents | MEDIUM | Low |
| Message format | Free-form | Structured envelope | MEDIUM | Low |
| Alignment checkpoints | Architect only | 5-7 per workflow | HIGH | Medium |
| Workflow state machine | Implicit | Explicit states + transitions | MEDIUM | Medium |
| Anti-pattern detection | None | Known failure mode recognition | MEDIUM | Low |
| Context passing minimum | Ad hoc | Defined minimum fields | HIGH | Low |
| Coordination tax tracking | None | < 20% overhead target | LOW | Low |
| Iteration caps | Inconsistent | All agents, with escalation | MEDIUM | Low |

---

## Priority Order for Improvement

1. **Structured handoffs + context passing** (Gaps 1, 8) — highest impact, moderate effort
2. **Quality gates** (Gap 2) — prevents error amplification
3. **Alignment checkpoints** (Gap 5) — keeps workflow on track
4. **Workflow state machine** (Gap 6) — formalizes the wave concept
5. **Error escalation** (Gap 3) — graceful failure handling
6. **Message format** (Gap 4) — reduces coordination noise
7. **Anti-pattern detection** (Gap 7) — self-correction capability
8. **Iteration caps** (Gap 10) — prevents runaway agents
9. **Coordination tax tracking** (Gap 9) — optimization metric
