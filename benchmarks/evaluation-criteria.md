# Evaluation Criteria and Benchmarks

**Purpose:** Define measurable criteria for evaluating agent team performance.
**Based on:** Production case studies research, Qodo 2026 metrics framework, Google DeepMind scaling laws, MAST failure taxonomy.

---

## 1. Effectiveness Metrics

### 1.1 Task Completion Rate

**Definition:** Percentage of assigned tasks that reach DONE without human takeover.
**Target:** > 70%
**How to measure:** `completed_tasks / total_assigned_tasks * 100`. A task counts as "completed" only if it passes Gate-2 (all tests pass, no critical findings). A task where a human took over the implementation counts as incomplete.
**When target is missed:** Analyze failed tasks by category (explore/design/implement/test). If failures cluster in one wave, the agent prompt or context for that wave needs improvement. If failures are spread evenly, the task decomposition by the lead may be too ambitious.

### 1.2 First-Attempt Success Rate

**Definition:** Percentage of tasks passing review (W4) on the first submission, without being sent back for fixes.
**Target:** > 50%
**How to measure:** `tasks_passing_first_review / tasks_reaching_review * 100`. "First attempt" means zero FINDING messages with severity CRITICAL or HIGH sent back to implementer.
**When target is missed:** Common causes: (1) design spec was ambiguous, (2) implementer deviated from design, (3) reviewer standards are inconsistent. Check handoff artifact quality between W2 and W3.

### 1.3 Human Intervention Rate

**Definition:** Percentage of tasks where a human had to step in to unblock, correct, or take over.
**Target:** < 30%
**How to measure:** Count any task where the human sent a message after Gate-1 approval that changed the implementation direction, fixed agent output, or resolved a blocker the agents could not. `tasks_with_intervention / total_tasks * 100`.
**When target is missed:** Categorize interventions: (a) tool/permission issues (fix infra), (b) ambiguous requirements (improve task descriptions), (c) agent capability limits (simplify task or use single agent), (d) coordination failures (improve handoff artifacts).

### 1.4 Coordination Overhead

**Definition:** Percentage of total tokens spent on inter-agent communication vs. productive work.
**Target:** < 20% of total tokens
**How to measure:** Sum tokens from: SendMessage calls, handoff artifact writing/reading, status updates, re-reads of other agents' output. Divide by total tokens consumed across all agents. `coordination_tokens / total_tokens * 100`.
**When target is missed:** Causes: (1) messages are too verbose (enforce structured message format), (2) too many status updates (reduce frequency), (3) agents re-reading large files for context (pass pointers, not content), (4) too many agents for the task size (use fewer agents).

---

## 2. Quality Metrics

### 2.1 Defect Density

**Definition:** Number of bugs found in agent-generated code after merge, per 1000 lines of code.
**Target:** Below team's historical average for human-written code (track trend, not absolute number initially).
**How to measure:** Track bugs reported against agent-generated code in the first 30 days post-merge. `bugs_found / (lines_of_code_generated / 1000)`.
**When target is missed:** Audit the review wave (W4). Are reviewers catching issues? Is the security auditor active? Consider adding a second review pass or requiring higher test coverage.

### 2.2 Test Coverage

**Definition:** Percentage of agent-generated code lines covered by tests.
**Target:** > 80% for new code
**How to measure:** Run coverage tool (istanbul/nyc, coverage.py, etc.) on the diff introduced by the agent. `covered_lines_in_diff / total_lines_in_diff * 100`.
**When target is missed:** Check if the tester agent received adequate context (test strategy from architect). Verify file ownership -- if the tester does not know which files were created, they cannot test them. Ensure the tester runs after the implementer (not just in parallel).

### 2.3 Static Analysis Issues

**Definition:** Number of linter, type-checker, and SAST findings per 1000 lines of agent-generated code.
**Target:** Decreasing trend over time; zero CRITICAL/HIGH severity findings at merge.
**How to measure:** Run the project's standard linter and SAST tools on agent-generated diffs. Count issues by severity. `issues / (lines_of_code / 1000)`.
**When target is missed:** Add linting as a pre-commit check that agents must pass. Include "run linter and fix issues" as an explicit step in the implementer's instructions. For SAST findings, involve the security auditor earlier.

### 2.4 PR Rejection Rate

**Definition:** Percentage of agent-generated PRs rejected or requiring significant rework during human review.
**Target:** < 40% (industry baseline for AI-generated PRs is 67.3% per LinearB)
**How to measure:** `rejected_or_reworked_PRs / total_agent_PRs * 100`. "Rejected" means the PR is closed without merge. "Reworked" means the human made substantial changes (> 20% of the diff) before merge.
**When target is missed:** Analyze rejection reasons: (a) correctness issues (strengthen W4 review), (b) style/convention violations (add codebase conventions to agent context), (c) scope issues (improve task scoping by lead), (d) test inadequacy (strengthen tester agent instructions).

---

## 3. Efficiency Metrics

### 3.1 Time to Completion

**Definition:** Wall-clock time from task assignment to merge-ready (passing Gate-2).
**Target:** Track trend; aim for improvement over time. No absolute target since task complexity varies.
**How to measure:** Record timestamps at task creation and Gate-2 approval. `gate2_time - task_creation_time`. Normalize by task complexity (estimated story points or lines changed) for cross-task comparison.
**When target is missed:** Identify bottlenecks: (a) long explore phase (provide better initial context), (b) long review cycles (improve first-attempt quality), (c) gate approval delays (human response time -- not agent's fault), (d) sequential dependencies (increase parallelism).

### 3.2 Token Efficiency

**Definition:** Ratio of useful output tokens (code, tests, docs, review findings) to total tokens consumed.
**Target:** > 50%
**How to measure:** Estimate useful output as tokens in: committed code, test files, review reports, documentation. Total tokens come from API usage logs. `useful_output_tokens / total_tokens * 100`. This is approximate -- use character counts as proxy if token counts are unavailable.
**When target is missed:** Causes: (1) agents reading too many irrelevant files (improve context pointers), (2) verbose internal reasoning (not directly fixable, but reduce prompt verbosity), (3) failed attempts that produced no output (improve task clarity), (4) redundant work between agents (check file ownership overlaps).

### 3.3 Agent Utilization

**Definition:** Percentage of time agents are doing productive work vs. waiting for dependencies or approvals.
**Target:** > 70%
**How to measure:** Track agent active time (time between spawn and completion) vs. idle time (waiting for gate approvals, waiting for other agents). `active_time / (active_time + idle_time) * 100`. In practice, measure wall-clock time each agent is alive and subtract known wait periods.
**When target is missed:** Causes: (1) gate approvals are slow (batch approvals or pre-approve low-risk decisions), (2) sequential dependencies where parallel execution was possible (redesign wave structure), (3) agents finish early and wait (assign secondary tasks).

### 3.4 Cost per Task

**Definition:** Total API cost (all agents combined) per completed task.
**Target:** Decreasing trend over time.
**How to measure:** Sum API costs from all agent sessions contributing to a task. Track per-task and compute running averages. Segment by task type (feature, bug fix, refactor) for meaningful comparison.
**When target is missed:** Identify cost drivers: (a) too many agents for simple tasks (use single-agent for small work), (b) excessive retries (improve first-attempt quality), (c) context re-reads (pass pointers), (d) over-scoped exploration (set exploration budgets).

---

## 4. Anti-Pattern Signals

These are early warning indicators that something is going wrong. Monitor continuously.

### 4.1 Repeated Edits to Same File

**Signal:** An agent edits the same file more than 3 times in one session.
**Indicates:** Agent drift, infinite fix loop, or unclear requirements.
**Detection:** Count per-file edit operations per agent session.
**Action:** If detected, the lead should: (1) check if the agent is looping on the same error, (2) review the task description for ambiguity, (3) cap further edits and escalate to Level 2 (fallback) or Level 3 (lead intervention) per the error escalation ladder.

### 4.2 Growing Context Without Progress

**Signal:** Agent's token consumption is increasing but no new artifacts (commits, test results, reports) are being produced.
**Indicates:** Context amnesia, confusion, or the agent is stuck reading without acting.
**Detection:** Track ratio of tokens consumed to artifacts produced per 5-minute window. If tokens > 10k with zero artifacts for 10+ minutes, flag.
**Action:** Reset the agent with fresh context. Provide a more specific task description. If the agent was exploring, it may need explicit pointers to relevant files instead of open-ended search.

### 4.3 Agent Disagreements Exceeding 3 Cycles

**Signal:** Reviewer sends FINDING, implementer "fixes," reviewer sends same or similar FINDING again, repeated 3+ times.
**Indicates:** Echo chamber, incompatible interpretations of requirements, or a genuine design disagreement.
**Detection:** Count FINDING -> FIX -> FINDING cycles per issue.
**Action:** The lead must intervene: (1) read both agents' reasoning, (2) make a decision, (3) communicate the decision as a binding directive. Do not let agents continue debating. If the disagreement is about architecture, escalate to human.

### 4.4 Token Spike Without Output

**Signal:** An agent consumes > 50k tokens in a single turn without producing a commit, test result, or report.
**Indicates:** Cost explosion, typically caused by reading very large files, hallucinating long outputs that get discarded, or getting stuck in a reasoning loop.
**Detection:** Monitor per-turn token consumption. Flag turns exceeding 50k tokens with no tool output.
**Action:** Set per-turn token budgets in agent configuration. If the agent needs to read large files, direct it to specific sections (line ranges) rather than entire files. Consider whether the task is too complex for the current agent and needs decomposition.

---

## 5. Measurement Infrastructure

### What to Log

For every agent session, capture:
- **Task ID and type** (feature, bug fix, refactor)
- **Agent role** (explorer, architect, implementer, tester, reviewer, auditor)
- **Start and end timestamps**
- **Total tokens consumed** (input + output)
- **Artifacts produced** (files created/modified, commits, reports)
- **Messages sent and received** (count + total tokens)
- **Escalations** (count, level reached, resolution)
- **Gate outcomes** (approved, rejected, with reasons)

### Reporting Cadence

- **Per-task:** Generate a summary after each task completes (automated).
- **Weekly:** Aggregate metrics across all tasks. Identify trends in completion rate, token efficiency, and anti-pattern frequency.
- **Monthly:** Review targets. Adjust thresholds based on accumulated data. Update agent prompts and workflow templates based on findings.

### Baseline Establishment

Before evaluating the team, establish baselines:
1. Run 5 tasks of each type (feature, bug fix, refactor) using a **single agent**
2. Record all metrics above
3. Run the same or equivalent tasks using the **multi-agent team**
4. Compare metrics to determine where multi-agent adds value and where it adds overhead

This baseline prevents the perception gap (METR finding: developers felt 20% faster but were 19% slower). Measure, do not estimate.
