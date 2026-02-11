# Agent Team

A production-grade multi-agent team configuration for software development, built on Claude Code's native team orchestration.

## What Is This?

Seven specialized AI agents that work as a coordinated team to build software. Each agent has a defined role, structured communication protocols, quality gates, and error handling — all based on 2025-2026 research into multi-agent system best practices.

**This is not a framework.** There are no scripts, no external dependencies, no custom orchestration code. It's a set of optimized agent definition files (`.md`) that leverage Claude Code's built-in `TeamCreate`, `Task`, and `SendMessage` capabilities.

## The Team

| Agent | Model | Role |
|-------|-------|------|
| **team-explorer** | Sonnet | Codebase researcher. Searches files, traces code paths, maps architecture. Read-only. |
| **team-architect** | Opus | System designer. Analyzes tradeoffs, produces design specs. Requires human approval. |
| **team-implementer** | Sonnet | Developer. Writes code following the approved design. Self-fixes up to 3 iterations. |
| **team-tester** | Sonnet | Test engineer. Writes tests in parallel with implementation. |
| **team-reviewer** | Opus | Code reviewer. Finds bugs, logic errors, convention violations. Read-only. |
| **team-security-auditor** | Sonnet | Security specialist. OWASP Top 10 audit. Read-only. |
| **team-doc-writer** | Haiku | Documentation writer. Updates docs after all changes are finalized. |

### Model Strategy (2 Opus + 4 Sonnet + 1 Haiku)

- **Opus** for decisions (architect, reviewer) — deep reasoning where quality is the bottleneck
- **Sonnet** for exploration + execution (explorer, implementer, tester, security-auditor) — chain-start quality matters; pattern matching and spec-following
- **Haiku** for documentation (doc-writer) — chain end, reads code and writes descriptions, errors easily caught

## Workflow

```
EXPLORE → DESIGN → [Human Approval] → IMPLEMENT + TEST → REVIEW + AUDIT → [Human Approval] → DOCUMENT
  W1        W2         Gate-1            W3 (parallel)      W4 (parallel)      Gate-2           W5
```

Each wave produces a structured handoff artifact that the next wave reads. No raw conversation history is passed between agents.

## Key Features

### Structured Handoffs
Every wave transition produces a standardized artifact (markdown file) containing: task context, decisions made, artifacts produced, and instructions for the next agent. Prevents context amnesia.

### Quality Gates
Two explicit approval checkpoints:
- **Gate-1** (Design → Implement): Human approves architecture before any code is written
- **Gate-2** (Review → Document): Zero CRITICAL findings required before merge

### Error Escalation
All agents follow a 4-level ladder: Retry (max 3) → Fallback (simplify) → Escalate (to lead) → Abort (save state).

### Anti-Pattern Defenses
Agents are trained to recognize: drift, echo chambers, infinite loops, over-engineering, scope creep, and context amnesia.

## Quick Start

### 1. Install

Copy the agent configs to your Claude Code agents directory:

```bash
cp agents/team-*.md ~/.claude/agents/
```

### 2. Use

In any Claude Code session:

```
Create a team to implement [your feature].
Use the team-architect to design the approach first.
```

Or for the full orchestrated workflow:

```
Use project-orchestrator to implement [your feature] with the full team workflow.
```

### 3. Customize

Edit the `.md` files in `~/.claude/agents/` to adjust:
- Model assignments (in YAML frontmatter)
- Role-specific instructions (in markdown body)
- Quality gate criteria
- Iteration caps

## Research Foundation

This configuration is grounded in 2025-2026 multi-agent research:

| Finding | Source | Impact on Design |
|---------|--------|-----------------|
| Hub-and-spoke reduces error amplification from 17.2x to 4.4x | Towards Data Science | Centralized lead topology |
| Multi-agent degrades sequential reasoning by 39-70% | Google DeepMind (Dec 2025) | Skip team for simple tasks |
| 67.3% AI PR rejection rate | LinearB | Mandatory review + security audit |
| 41-86.7% failure rates across 7 MAS frameworks | MAST Taxonomy | Anti-pattern detection in prompts |
| AI review + generation = 81% quality improvement | Qodo | Dedicated reviewer agent |

Full research reports in `docs/research/`.

## Project Structure

```
agent-team/
├── agents/           # Deploy these to ~/.claude/agents/
├── docs/
│   ├── architecture.md
│   ├── research/     # 3 survey reports (frameworks, protocols, case studies)
│   └── design/       # Gap analysis + interaction protocol design
├── templates/        # Workflow templates (feature, bug-fix, refactoring)
├── benchmarks/       # Evaluation criteria and metrics
├── baseline/         # Original configs (before improvements)
└── README.md
```

## When NOT to Use Multi-Agent

Research is clear: multi-agent is not always better.

- **Single-file bug fix** → Use a single agent
- **Sequential reasoning task** → Single agent (39-70% degradation with multi-agent)
- **Quick script** → Single agent
- **Anything completable in < 30 minutes** → Single agent overhead exceeds value

Use the team for: **parallel work, multi-file features, complex architecture decisions, and tasks requiring independent verification.**

## License

MIT
