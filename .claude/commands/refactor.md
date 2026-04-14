---
description: >-
  Refactor existing code safely with incremental changes and behavior preservation.
  Requires a specific goal — refuses vague "refactor this directory" requests.
  Uses impact analysis → incremental change → parity verification. Can be invoked
  directly by the user or by the orchestrator for structural changes.
---

# Refactor Workflow

## Scope Guard

**This command requires a specific, actionable goal.** Do not proceed with vague requests.

- ✅ `/refactor extract storage interface in task feature`
- ✅ `/refactor split user handler into separate auth handler`
- ✅ `/refactor migrate callbacks to async/await in notification service`
- ❌ `/refactor apps/backend` — too vague. Use the QA Analyst to audit first, then refactor specific findings.
- ❌ `/refactor improve code quality` — too vague. What specific quality issue?

If the request is vague, respond:
> "Refactoring needs a specific target. What exactly should change? If you're not sure, run a code review first to identify specific structural issues."

## When to Use

- Code restructuring (moving, renaming, splitting modules)
- Pattern migration (e.g., switching from callbacks to async/await)
- Dependency upgrades with breaking changes
- Addressing structural findings from QA Analyst audits
- Extracting interfaces, splitting files, or reorganizing modules

## When NOT to Use

- New features → use the orchestrator
- Small bug fixes → use the orchestrator with Pattern B (Bug Fix)
- "Find what to improve" → have QA Analyst audit first, then refactor specific findings

## Phases

### Phase 1: Impact Analysis

1. **Map the blast radius** — What files, modules, and tests are affected?
2. **Document existing behavior** — What tests currently pass? What contracts exist?
3. **Identify risks** — Can this be done incrementally, or is a big-bang needed?
4. **Create refactoring plan** — Ordered list of incremental steps, each preserving behavior
5. **ADR (if needed)** — If the refactoring involves trade-offs, create an ADR using the `adr` skill

**Skills to consider:**
- `sequential-thinking` — for multi-step refactoring with interdependencies
- `research-methodology` — if the refactoring involves unfamiliar patterns

### Phase 2: Incremental Change

For each step in the refactoring plan:
1. **Ensure existing tests pass** before making any change
2. **Make one incremental change** — move, rename, or restructure
3. **Run tests after each change** — behavior must be preserved
4. **Add new tests** if the refactoring exposes untested behavior

**Key principle:** Never break the build for more than one step at a time.

**Rules to follow:**
- Architectural Patterns @.claude/rules/architectural-pattern.md
- Code Organization Principles @.claude/rules/code-organization-principles.md
- Project Structure @.claude/rules/project-structure.md

### Phase 3: Parity Verification

1. Run the full validation suite (linters, type checks, tests, build)
2. **Compare test coverage** — coverage MUST be equal to or better than before
3. **Verify no behavior changes** — same inputs produce same outputs
4. If UI was touched, run E2E tests

### Phase 4: Ship

Commit with conventional format:
```bash
git commit -m "refactor(<scope>): <description>"
```

## Completion Criteria

- [ ] Impact analysis documented
- [ ] All changes made incrementally with tests passing at each step
- [ ] Full verification suite passes
- [ ] Test coverage ≥ pre-refactoring coverage
- [ ] Committed with `refactor` type
