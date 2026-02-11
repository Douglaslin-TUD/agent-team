# Agent Team Architecture

**Version:** 2.0
**Date:** 2026-02-11

---

## Overview

A production-grade multi-agent team for software development, built on Claude Code's native team orchestration (TeamCreate, Task, SendMessage). No external frameworks or scripts — all orchestration lives in agent prompt definitions (.md files).

### Design Principles

1. **Hub-and-spoke topology** — Centralized team lead coordinates all agents. Reduces error amplification from 17.2x (flat) to 4.4x (centralized). (Source: Google DeepMind, Dec 2025)
2. **Structured handoffs** — Every wave transition produces a schema-validated artifact. Prevents context amnesia, the #1 cause of multi-agent failure.
3. **Quality gates** — Explicit checklist criteria at each state transition. No output passes to the next wave without validation.
4. **Tiered autonomy** — Read-only operations auto-proceed. Code changes need lead approval. Architecture and merge need human approval.
5. **Fail-safe escalation** — 4-level ladder (retry → fallback → escalate → abort) with iteration caps on every agent.
6. **Minimal coordination tax** — Target < 20% of total tokens on inter-agent coordination. Pass pointers, not content.

---

## Team Composition

### Model Allocation Strategy

| Tier | Model | Agents | Rationale |
|------|-------|--------|-----------|
| **Brain** | Opus | architect, reviewer | Design decisions, subtle bug detection — highest-value reasoning |
| **Hands** | Sonnet | explorer, implementer, tester, security-auditor | Exploration (chain start quality matters), execution, pattern-matching audit |
| **Light** | Haiku | doc-writer | Chain end — reads code, writes descriptions, errors easily caught |

**Why this split:** Research shows architecture quality is the bottleneck (everything downstream depends on it). Review quality catches the 67.3% of AI-generated code that would otherwise be rejected. Explorer needs sonnet-level reasoning as the chain start — poor exploration cascades downstream. Security auditor is primarily pattern matching, with the opus reviewer as a second line of defense.

---

## Workflow State Machine

```
EXPLORE → DESIGN → [GATE-1] → IMPLEMENT + TEST → VERIFY + AUDIT → [GATE-2] → DOCUMENT → DONE
  W1        W2     (approve)    W3 (parallel)      W4 (parallel)   (approve)     W5
```

### Wave Details

| Wave | Agents | Parallelism | Key Output |
|------|--------|-------------|------------|
| W1 | explorer | Single | Exploration report: key files, patterns, dependencies |
| W2 | architect | Single | Design spec: components, APIs, file ownership, test strategy |
| Gate-1 | Lead + Human | — | Plan approval |
| W3 | implementer + tester | Parallel | Code + tests (no file overlap via ownership) |
| W4 | reviewer + security-auditor | Parallel | Review report + security audit |
| Gate-2 | Lead + Human | — | Merge approval |
| W5 | doc-writer | Single | Updated documentation |

### When to Skip Waves

| Task Type | Skip | Why |
|-----------|------|-----|
| Bug fix (clear root cause) | W1, sometimes W2 | Requirements are the bug itself |
| Small refactor | W1 | Patterns already known |
| Single-file change | W4 (security), W5 | Overhead exceeds value |
| Sequential reasoning task | ALL — use single agent | Multi-agent degrades sequential reasoning by 39-70% |

---

## Communication Architecture

### Channels

1. **Task Board** (TaskCreate/TaskUpdate/TaskList) — Global project state. All agents read. Lead writes.
2. **Direct Messages** (SendMessage) — Agent-to-agent communication. Structured envelope format.
3. **Handoff Artifacts** (markdown files) — Persistent output from each wave. Survives context compaction.
4. **Git** — Source of truth for code changes. Commits as checkpoints.

### Message Flow

```
          ┌─────────┐
          │  Human   │
          └────┬─────┘
               │ approve/reject
          ┌────▼─────┐
          │   Lead    │◄──── Task Board (global state)
          └────┬─────┘
        ┌──────┼──────┬──────────┐
        ▼      ▼      ▼          ▼
    explorer architect implementer reviewer
                       tester     security-auditor
                                  doc-writer
```

Agents communicate UP to lead, never directly to each other (except reviewer/auditor findings → implementer for fixes, routed via lead).

---

## Error Handling

### Escalation Ladder

```
L1: Retry     → Same agent, modified approach (max 3)
L2: Fallback  → Simplify task, decompose, try alternative
L3: Escalate  → Send to lead with full context
L4: Abort     → Save state to git, halt, notify human
```

### Iteration Caps

| Agent | maxTurns | Retry Cap | Rationale |
|-------|----------|-----------|-----------|
| explorer | 30 | 2 | If files aren't found in 2 tries, escalate |
| architect | 40 | 2 | Design disagreements resolved by lead |
| implementer | 50 | 3 | Self-fix loop: implement → test → fix |
| tester | 40 | 3 | Test failures may need implementer coordination |
| reviewer | 30 | 1 | Review is a single-pass operation |
| security-auditor | 30 | 1 | Audit is a single-pass operation |
| doc-writer | 30 | 2 | If docs don't match code, escalate |

---

## Quality Gates

### Gate-1: Design → Implementation

- [ ] All requirements addressed in design
- [ ] File ownership assigned (no overlaps)
- [ ] Testing strategy defined
- [ ] Risk flags documented
- [ ] Human approved plan (via plan_approval_response)

### Gate-2: Review → Documentation

- [ ] All tests pass
- [ ] Zero CRITICAL findings from reviewer
- [ ] Zero CRITICAL findings from security auditor
- [ ] HIGH findings resolved or accepted with justification
- [ ] Lead confirms merge-readiness

---

## Anti-Pattern Defenses

| Pattern | Detection | Mitigation |
|---------|-----------|------------|
| Bag of Agents (17.2x error) | — | Hub-and-spoke topology enforced |
| Agent Drift | Work outside task scope | Clear task descriptions, scope constraints |
| Echo Chamber | Agents agreeing without verification | Independent review in W4 |
| Infinite Loop | > 3 fix cycles | Iteration caps with escalation |
| Context Amnesia | Contradicting prior decisions | Handoff artifacts as external memory |
| Over-engineering | Abstractions not in design | KISS/YAGNI in every agent prompt |
| Cost Explosion | Token spike without progress | maxTurns caps, coordination tax monitoring |

---

## File Structure

```
agent-team/
├── agents/                     # Improved agent definitions (deploy to ~/.claude/agents/)
│   ├── team-explorer.md
│   ├── team-architect.md
│   ├── team-implementer.md
│   ├── team-tester.md
│   ├── team-reviewer.md
│   ├── team-security-auditor.md
│   └── team-doc-writer.md
├── docs/
│   ├── architecture.md         # This document
│   ├── research/               # Survey reports
│   │   ├── frameworks-survey.md
│   │   ├── interaction-protocols-survey.md
│   │   └── production-case-studies.md
│   └── design/                 # Design specifications
│       ├── gap-analysis.md
│       └── interaction-protocols.md
├── templates/                  # Workflow templates
│   ├── feature-development.md
│   ├── bug-fix.md
│   └── refactoring.md
├── benchmarks/                 # Evaluation criteria
│   └── evaluation-criteria.md
├── baseline/                   # Original agent configs (for comparison)
│   └── team-*.md
└── README.md
```

---

## Research Foundation

This architecture is grounded in research from 2025-2026:

- **Google DeepMind** (Dec 2025): Centralized MAS improves parallelizable tasks by 80.8%, degrades sequential reasoning by 39-70%
- **MAST Taxonomy** (2025): 14 failure modes identified, 41-86.7% failure rates across 7 frameworks
- **Bag of Agents** research: 17.2x error amplification without topology vs 4.4x with hub-and-spoke
- **Anthropic C Compiler**: 16 parallel Claudes, 100K lines, file-based coordination
- **Production quality data**: 67.3% AI PR rejection rate, but teams using AI for review + generation see 81% quality improvement

Full citations in `docs/research/`.
