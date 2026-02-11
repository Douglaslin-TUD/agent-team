# Workflow Template: Feature Development

**When to use:** Adding a new capability, endpoint, UI component, or module to the codebase.
**When NOT to use:** Small additions (< 50 lines) that touch 1-2 files -- use a single agent instead.

---

## Wave Sequence

```
W1: EXPLORE  →  W2: DESIGN  →  [Gate-1]  →  W3: IMPLEMENT + TEST  →  W4: VERIFY  →  [Gate-2]  →  W5: DOCUMENT  →  DONE
```

## Agents and Responsibilities

| Wave | Agent | Task |
|------|-------|------|
| W1 | team-explorer | Map affected code, find existing patterns, identify dependencies |
| W2 | team-architect | Write design spec, assign file ownership, define test strategy |
| Gate-1 | Lead + Human | Approve design via plan_approval_response |
| W3a | team-implementer | Write production code per design spec |
| W3b | team-tester | Write tests per test strategy (parallel with W3a) |
| W4a | team-reviewer | Review code quality, correctness, adherence to design |
| W4b | team-security-auditor | Audit for security issues (parallel with W4a) |
| Gate-2 | Lead + Human | Approve merge-readiness |
| W5 | team-doc-writer | Update relevant documentation |

## Gate Conditions

### Gate-1: Design Approval

All must be true before spawning implementer and tester:

- [ ] All user requirements addressed in the design spec
- [ ] File ownership assigned with zero overlaps between implementer and tester
- [ ] API contracts defined (function signatures, data shapes)
- [ ] Testing strategy defined (unit, integration, edge cases)
- [ ] Risk flags documented (breaking changes, performance concerns, security surface)
- [ ] Human has approved the plan

### Gate-2: Merge Readiness

All must be true before proceeding to documentation:

- [ ] All tests pass (both new and existing)
- [ ] Zero CRITICAL findings from reviewer and security auditor
- [ ] All HIGH findings resolved or explicitly accepted with justification
- [ ] Test coverage meets target (> 80% for new code)
- [ ] Lead confirms merge-readiness

## Wave Details

### W1: Explore (team-explorer)

**Input:** User's feature request + relevant file hints from lead.
**Output:** `exploration-report.md` containing:
- Key files with line numbers and their roles
- Existing patterns the new code should follow
- Dependencies (internal modules, external packages)
- Potential conflict areas

**Budget:** Max 15 minutes / 50k tokens.

### W2: Design (team-architect)

**Input:** Exploration report.
**Output:** `design-spec.md` containing:
- Component breakdown (what to build)
- File ownership table (implementer files vs. tester files)
- API contracts and data types
- Error handling strategy
- Testing strategy (passed to tester as their instruction set)

**Budget:** Max 15 minutes / 50k tokens.

### W3: Implement + Test (parallel)

**Implementer input:** Design spec (component design, file ownership, API contracts).
**Tester input:** Design spec (testing strategy, scenarios, mocking approach).

Both work in parallel on non-overlapping files. Implementer commits code; tester commits tests. The tester runs tests against the implementation at the end.

**Budget:** Max 30 minutes / 100k tokens each.

### W4: Verify (parallel)

**Reviewer input:** Changed files list, test results, self-assessment from implementer.
**Security auditor input:** Changed files, new inputs/auth changes.

Both produce reports with findings categorized as CRITICAL / HIGH / MEDIUM / LOW.

**Budget:** Max 15 minutes / 50k tokens each.

### W5: Document (team-doc-writer)

**Input:** Design spec + final implementation.
**Output:** Updated docs (README, API docs, inline comments where needed).

**Budget:** Max 10 minutes / 30k tokens.

---

## Example: Adding a /health Endpoint to an Express API

**W1 Explorer discovers:** `src/routes/index.ts` (route registry), `src/middleware/auth.ts` (auth middleware), existing pattern: each route in its own file under `src/routes/`.

**W2 Architect designs:**
- New file: `src/routes/health.ts` (implementer owns)
- New file: `tests/routes/health.test.ts` (tester owns)
- Modify: `src/routes/index.ts` to register route (implementer owns)
- Contract: `GET /health` returns `{ status: "ok", uptime: number }`
- No auth required (skip auth middleware for this route)
- Test scenarios: 200 response, correct shape, uptime > 0

**W3 Implementer writes** the route and registers it. **Tester writes** integration tests using supertest.

**W4 Reviewer** confirms adherence to patterns. **Security auditor** confirms no auth bypass risk.

**W5 Doc writer** adds `/health` to the API reference.
