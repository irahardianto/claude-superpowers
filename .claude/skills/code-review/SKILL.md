---
name: code-review
description: >-
  Execute a structured code review protocol that inspects code quality against the full rule set.
  Use when auditing code written by yourself or another agent, during code audit workflows,
  or when the user asks for a code review. Produces a findings document with severity tags.
---

# Code Review Skill

## Purpose
Systematically review code against the full rule set. Catches issues that linters miss: architectural violations, missing observability, business logic errors, pattern inconsistencies.

## When to Invoke
- During code audit workflows (Phase 1: Code Review)
- When user asks for a code review outside any workflow
- **Best practice:** Invoke in a fresh conversation (not the same one that authored the code) to avoid confirmation bias

## Review Process

### 1. Scope the Review
Identify the files/features to review. Determine the review scope:
- **Feature review** — all files in a feature directory
- **PR review** — only changed files
- **Full codebase audit** — all features

### 2. Load the Rule Set
Read all applicable rules from `.claude/rules/`. Use `rule-priority.md` for severity classification.

### 3. Review Categories (Priority Order)

Review each file/feature against these categories, in order from `rule-priority.md`:

#### Critical (Must Fix)
- **Security** — injection, hardcoded secrets, broken auth
- **Data loss** — missing error handling on writes, no transaction boundaries
- **Resource leaks** — unclosed connections, missing cleanup

#### Major (Should Fix)
- **Testability** — I/O not behind interfaces, untested error paths
- **Observability** — missing logging on operations, no correlation IDs
- **Error handling** — empty catch blocks, swallowed errors
- **Architecture** — circular dependencies, wrong layer access

#### Minor (Nice to Fix)
- **Pattern consistency** — deviation from established codebase patterns
- **Naming** — unclear variable/function names
- **Code organization** — functions too long, mixed responsibilities

#### Nit (Optional)
- **Style** — formatting issues the linter would catch
- **Documentation** — missing comments on complex logic

### 4. Produce Findings

Output a structured findings document:

```markdown
# Code Review: {Feature/Module Name}
Date: {date}
Reviewer: AI Agent (fresh context)

## Summary
- **Files reviewed:** N
- **Issues found:** N (X critical, Y major, Z minor, W nit)

## Critical Issues
- [ ] **[SEC]** {description} — [{file}:{line}](file:///path)
- [ ] **[DATA]** {description} — [{file}:{line}](file:///path)

## Major Issues
- [ ] **[TEST]** {description} — [{file}:{line}](file:///path)
- [ ] **[OBS]** {description} — [{file}:{line}](file:///path)

## Minor Issues
- [ ] **[PAT]** {description} — [{file}:{line}](file:///path)

## Nit
- [ ] {description} — [{file}:{line}](file:///path)

## Rules Applied
List of rules referenced during this review.
```

### 5. Save the Report

When invoked via an audit workflow, you **MUST** persist the findings to the repo:

**Path:** `docs/audits/review-findings-{feature}-{YYYY-MM-DD}-{HHmm}.md`

1. Create `docs/audits/` if it doesn't exist
2. Write the findings document to that path
3. This makes the report accessible from other conversations and agents

When invoked as a standalone review (not via audit), saving to `docs/audits/` is recommended but optional.

### 6. Severity Tags

| Tag      | Category             | Rule Source                                        |
| -------- | -------------------- | -------------------------------------------------- |
| `[SEC]`  | Security             | `security-principles.md`                           |
| `[DATA]` | Data integrity       | `error-handling-principles.md`                     |
| `[RES]`  | Resource leak        | `resources-and-memory-management` skill            |
| `[TEST]` | Testability          | `architectural-pattern.md`, `testing-strategy.md`  |
| `[OBS]`  | Observability        | `logging-and-observability-mandate.md`             |
| `[ERR]`  | Error handling       | `error-handling-principles.md`                     |
| `[ARCH]` | Architecture         | `architectural-pattern.md`, `project-structure.md` |
| `[PAT]`  | Pattern consistency  | `code-organization-principles.md`                  |
| `[INT]`  | Integration contract | `api-design-principles` skill                      |
| `[DB]`   | Database design      | `database-design-principles` skill                 |
| `[CFG]`  | Configuration        | `configuration-management-principles` skill        |

### 7. Language-Specific Anti-Patterns

Load the anti-pattern checklist for the language(s) under review:

| Language | Anti-Patterns |
|---|---|
| **Go** | `languages/go.md` |
| **TypeScript** | `languages/typescript.md` *(placeholder — create when needed)* |
| **Flutter/Dart** | `languages/flutter.md` *(placeholder — create when needed)* |
| **Rust** | `languages/rust.md` *(placeholder — create when needed)* |

> Anti-patterns listed in language files are **auto-fail** — they require no judgment call. If the pattern exists in the code, it is a finding.

### 8. Cross-Boundary Checks

For full audits, apply the cross-boundary dimension checklist below. For standalone reviews, apply only the dimensions that are relevant to the project.

#### Dimension Selection

At the start of cross-boundary checks, you MUST state which dimensions are active:
> "Activating dimensions: A, B, C, D, E. Skipping F (no mobile app)."

| Dimension | Activate When |
|---|---|
| **A. Integration Contracts** | Project has both a frontend and a backend |
| **B. Database & Schema** | Project uses a relational/document database |
| **C. Configuration & Environment** | Always — universal |
| **D. Dependency Health** | Always — universal |
| **E. Test Coverage Gaps** | Always — universal |
| **F. Mobile ↔ Backend** | Project has a mobile app and a backend |

#### Dimension A: Integration Contracts
*Applies to: full-stack projects with frontend + backend*

- [ ] Map every backend endpoint (route + method) against its frontend adapter — flag any unmapped endpoints in either direction
- [ ] Verify request/response field names, types, and status codes match across the boundary
- [ ] Verify all outbound HTTP calls use the project's centralized API client (not raw `fetch`/`axios`)
- [ ] Build an auth coverage matrix: which endpoints require auth, do the frontend adapters send tokens for each?
- [ ] Check error contract alignment: does the frontend handle the full set of error codes the backend can return?

#### Dimension B: Database & Schema
*Applies to: projects using a relational or document database*

- [ ] Verify all tables have required base columns (`id`, `created_at`, `updated_at`)
- [ ] Check all foreign keys have corresponding indexes
- [ ] If using Supabase or Postgres RLS: verify RLS policies exist on every table storing user data
- [ ] Cross-reference the application's struct/model field names against actual DB column names — flag any drift
- [ ] Check migrations are reversible (up + down) and follow the additive-first strategy
- [ ] Scan storage adapters for N+1 query patterns

#### Dimension C: Configuration & Environment
*Always active*

- [ ] No hardcoded secrets, tokens, URLs, or credentials in source code
- [ ] `.env.template` exists and covers every env var referenced in the codebase
- [ ] Startup validation fails fast on missing required config (does not silently fall back to bad defaults)
- [ ] Secrets are never logged (not in debug, not in error messages)

#### Dimension D: Dependency Health
*Always active*

- [ ] No unused top-level dependencies in `go.mod` / `package.json` / `Cargo.toml`
- [ ] No circular dependencies between feature modules
- [ ] Cross-module imports only use each module's public API (not internal files)
- [ ] Run `npm audit` / `go list -m -json all | nancy` / `cargo audit` — flag high-severity CVEs

#### Dimension E: Test Coverage Gaps
*Always active*

- [ ] A handler/controller test exists for every API endpoint
- [ ] An integration test exists for every storage/database adapter
- [ ] Every error path (catch block, error return) has at least one test that exercises it
- [ ] E2E tests cover the primary user journeys (login, main feature flow, error states)

#### Dimension F: Mobile ↔ Backend
*Applies to: projects with a mobile app and a backend*

- [ ] API version compatibility — mobile must not call endpoints that no longer exist
- [ ] Offline data sync: conflict resolution and retry logic are tested
- [ ] Auth token refresh flows work correctly when the access token expires mid-session

---

#### Zero-Findings Guard

If this review produces fewer than 3 findings, you MUST produce a "Dimensions Covered" attestation section in the findings document:

```markdown
## Dimensions Covered
| Dimension | Status | Files / Queries Examined |
|---|---|---|
| A. Integration Contracts | ✅ Checked / ⏭ Skipped (reason) | e.g., all 26 routes vs 11 adapters |
| B. Database & Schema | ✅ Checked / ⏭ Skipped (reason) | e.g., 8 tables + 4 storage adapters |
| C. Configuration & Environment | ✅ Checked | e.g., scanned for secrets, verified .env.template |
| D. Dependency Health | ✅ Checked | e.g., ran npm audit, checked for unused deps |
| E. Test Coverage Gaps | ✅ Checked | e.g., verified handler tests for all endpoints |
| F. Mobile ↔ Backend | ⏭ Skipped | No mobile app in this project |
```

Only then may you declare a clean result.

---

## Rule Compliance
This skill enforces all rules in `.claude/rules/`. Key references:
- Rule Priority @.claude/rules/rule-priority.md (severity classification)
- Security Principles @.claude/rules/security-principles.md
- Architectural Patterns @.claude/rules/architectural-pattern.md
- Testing Strategy @.claude/rules/testing-strategy.md
- Logging and Observability Mandate @.claude/rules/logging-and-observability-mandate.md
- Error Handling Principles @.claude/rules/error-handling-principles.md
