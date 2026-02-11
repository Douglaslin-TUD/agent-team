# Workflow Template: Refactoring

**When to use:** Restructuring existing code for clarity, performance, or maintainability without changing behavior.
**When NOT to use multi-agent:** Small refactors (rename, extract function, move file) touching < 5 files. Use a single agent.

---

## Wave Sequence

```
W1: UNDERSTAND  →  W2: DESIGN  →  [Gate-1]  →  W3: REFACTOR + VERIFY  →  [Gate-2]  →  DONE
```

Key difference from feature development: no new functionality is added. The entire focus is on preserving existing behavior while improving code structure.

## Agents and Responsibilities

| Wave | Agent | Task |
|------|-------|------|
| W1 | team-explorer | Map current structure, catalog tests, identify risks |
| W2 | team-architect | Design target structure, define refactoring steps |
| Gate-1 | Lead + Human | Approve refactoring plan |
| W3a | team-implementer | Execute refactoring steps |
| W3b | team-tester | Run tests continuously, add coverage for gaps |
| Gate-2 | Lead | Confirm all tests pass, no behavior change |

## Single-Agent vs. Team Decision

Use a **single agent** when: rename, extract function, move file, simplify logic, or any change touching < 5 files.

Use the **team** when: restructuring module boundaries, changing widely-used abstractions, migrating patterns across many files, performance work needing benchmarks, or areas with low test coverage.

## Critical Rule: Test Coverage Before Refactoring

Before any refactoring begins, the existing behavior must be captured in tests. If test coverage for the target area is below 80%, the tester's first job is to add characterization tests that document current behavior. Refactoring without this safety net is prohibited.

## Wave Details

### W1: Understand (team-explorer)

**Input:** Refactoring goal from user/lead (e.g., "split the auth module into auth and session").
**Output:** `understanding-report.md` containing:
- Current code structure (files, key functions, data flow)
- Dependency graph (what depends on the code being refactored)
- Existing test coverage assessment (which paths are tested, which are not)
- Risk areas (high coupling, global state, side effects)
- Current behavior snapshot (key inputs/outputs for regression testing)

**Budget:** Max 15 minutes / 50k tokens.

### W2: Design (team-architect)

**Input:** Understanding report.
**Output:** `refactoring-plan.md` containing:
- Target structure (what the code should look like after)
- Step-by-step transformation plan (ordered, each step keeps tests green)
- File ownership table
- Coverage gap list (tests to add before refactoring)
- Rollback strategy (how to revert if things go wrong)

**Key principle:** Each step in the plan must keep all tests passing. Never plan a "big bang" refactor where tests break and are fixed later.

**Budget:** Max 15 minutes / 50k tokens.

### Gate-1: Plan Approval

The lead and human review the refactoring plan:
- [ ] Steps are incremental (each keeps tests green)
- [ ] Coverage gaps are addressed in the plan
- [ ] Rollback strategy is defined
- [ ] No scope creep (no feature additions mixed in)

### W3: Refactor + Verify (coordinated)

**Phase 1 -- Coverage (tester first, if gaps exist):**
The tester adds characterization tests for any uncovered paths identified in W1. These tests must pass before refactoring starts. This phase blocks the implementer.

**Phase 2 -- Refactor (implementer, with tester monitoring):**
The implementer executes the plan step by step. After each step:
1. Implementer commits the change
2. Tester runs the full test suite
3. If tests fail, implementer fixes before proceeding to next step
4. If a step cannot be completed without breaking tests, escalate to lead

**Budget:** Max 30 minutes / 100k tokens (implementer), 20 minutes / 60k tokens (tester).

### Gate-2: Verification

The lead confirms:
- [ ] All pre-existing tests still pass
- [ ] No behavior changes (same inputs produce same outputs)
- [ ] New characterization tests pass
- [ ] Code structure matches the design target
- [ ] No leftover dead code, commented-out code, or TODO markers

---

## Anti-Patterns to Watch For

| Anti-Pattern | Signal | Prevention |
|-------------|--------|------------|
| Refactor + feature in one PR | New tests test new behavior, not existing | Reject; split into two separate workflows |
| Big bang refactor | Tests broken for multiple steps | Revert to last green state; replan with smaller steps |
| Coverage regression | Coverage % drops after refactoring | Tester adds tests before implementer deletes/moves code |
| Scope creep | "While I'm here, let me also fix..." | Only touch what the refactoring plan specifies |
| Premature abstraction | New interfaces/base classes for single use | Remove; follow YAGNI (You Ain't Gonna Need It) |

## Example: Splitting a Monolithic UserService

**Goal:** Split `src/services/user-service.ts` (800 lines) into `user-service.ts` (profile CRUD) and `session-service.ts` (auth/session).

**W1 Explorer finds:**
- 23 files import from `user-service.ts`
- Test coverage: 65% (below threshold -- need characterization tests)
- Risk: `getUser()` is called in 40+ places with implicit session side effects

**W2 Architect plans:**
1. Tester adds characterization tests to reach 85% coverage
2. Extract session-related types to `session-types.ts`
3. Extract session functions to `session-service.ts` (re-export from user-service for compatibility)
4. Update imports in consuming files (5 at a time, test between batches)
5. Remove re-exports from `user-service.ts`
6. Clean up unused imports

**W3 Execution:** Tester writes 12 characterization tests. Implementer executes steps 2-6, committing after each. Tester runs suite after each commit. All green at every step.
