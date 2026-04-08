---
name: guardrails
description: >-
  Run pre-flight checklists before writing code and post-implementation self-review after writing code.
  Use at the start of implementation to verify readiness, and after writing code but before verification
  to catch architectural violations, missing observability, and security oversights early.
---

# Agent Guardrails Skill

## Purpose
Structured checklists that ensure the agent considers all applicable rules before and after writing code. Catches issues that would otherwise only surface during verification.

## When to Invoke
- **Pre-Flight:** At the start of implementation, before writing any code
- **Self-Review:** After writing code but before verification phases

---

## Pre-Flight Checklist

Run through this checklist **before writing any code**:

- [ ] Identified all applicable rules from `.claude/rules/`
- [ ] Searched for existing patterns in codebase (Pattern Discovery Protocol from `architectural-pattern.md`)
- [ ] Confirmed project structure alignment (`project-structure.md`)
- [ ] Identified I/O boundaries that need abstraction
- [ ] Determined test strategy (unit/integration/E2E)
- [ ] Reviewed `rule-priority.md` for any potential conflicts

If any item cannot be checked, **stop and resolve** before proceeding.

---

## Post-Implementation Self-Review

Run through this checklist **after writing code, before verification**:

### Security
- [ ] No hardcoded secrets or configuration values
- [ ] All user input validated at system boundaries
- [ ] Parameterized queries (no string concatenation for SQL)

### Testability
- [ ] All I/O operations behind interfaces/abstractions
- [ ] Business logic is pure (no side effects in calculations)
- [ ] Dependencies injected, not hardcoded

### Observability
- [ ] All public operation entry points logged (start/success/failure)
- [ ] Structured logging with correlation IDs
- [ ] Appropriate log levels (not everything is INFO)

### Error Handling
- [ ] Error paths handled explicitly (no empty catch blocks)
- [ ] Errors provide context (wrapped with additional info)
- [ ] Resources cleaned up in error paths (defer/finally)

### Testing
- [ ] Tests cover happy path
- [ ] Tests cover at least 2 error paths
- [ ] Tests cover edge cases relevant to the domain
- [ ] If I/O adapters were modified: integration tests exist and pass
- [ ] If UI was modified: E2E tests exist (or are planned)

### Consistency
- [ ] Follows existing codebase patterns (>80% consistency)
- [ ] Naming conventions match the codebase
- [ ] File organization matches `project-structure.md`

---

## Language-Specific Self-Review

After completing the universal checklist above, load the relevant language-specific checklist:

| Language | Checklist |
|---|---|
| **Go** | `languages/go.md` |
| **TypeScript** | `languages/typescript.md` *(placeholder — create when needed)* |
| **Flutter/Dart** | `languages/flutter.md` *(placeholder — create when needed)* |
| **Rust** | `languages/rust.md` *(placeholder — create when needed)* |

> Only load the file for languages you are actively writing. If the file doesn't exist yet, skip — but flag its absence so it can be created.

---

## Rule Compliance
This skill enforces:
- All mandates (always-on rules in `.claude/rules/`)
- Architectural Patterns @.claude/rules/architectural-pattern.md
- Testing Strategy @.claude/rules/testing-strategy.md
- Rule Priority @.claude/rules/rule-priority.md
