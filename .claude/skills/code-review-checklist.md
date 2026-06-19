---
name: code-review-checklist
description: "Trigger this skill when a developer asks to review code, prepare for PR, check their work before submitting, or says 'is my code ready', 'review this', 'check my changes', 'pre-PR check'. Also triggers automatically before any ticket moves to In Review status. Runs a structured quality checklist to catch issues before they reach a senior developer or production."
---

# Code Review Checklist

## Core Principles

1. **Catch issues early** — the earlier a bug is found, the cheaper it is to fix
2. **Consistency over cleverness** — readable code beats clever code every time
3. **No broken windows** — don't leave TODO comments, dead code, or debug statements
4. **Security is not optional** — every PR gets a security scan before merge
5. **Tests are not optional** — untested code is assumed broken

---

## Pre-Review: What Claude Checks First

Before starting a review, Claude will:

1. Read the linked ClickUp ticket to understand the intended behavior
2. Identify which files were changed (`git diff --name-only`)
3. Understand the scope — is this a feature, bugfix, refactor, or chore?

---

## Checklist: Security

- [ ] No hardcoded API keys, tokens, passwords, or secrets
- [ ] No `.env` file or credentials file accidentally staged
- [ ] No sensitive data logged to console (user emails, tokens, PII)
- [ ] User input is validated and sanitized before use
- [ ] No SQL queries built via string concatenation (use parameterized queries)
- [ ] Authentication and authorization checks are in place where needed
- [ ] No `eval()` or dynamic code execution on user input

**If any security item fails → block the PR immediately and explain the risk.**

---

## Checklist: Code Quality

- [ ] No `console.log`, `print`, `debugger`, or `TODO` left in the code
- [ ] No commented-out code blocks (delete dead code, don't comment it out)
- [ ] No unused imports or variables
- [ ] Functions do one thing — if a function is doing 3 things, suggest splitting it
- [ ] Variable and function names are descriptive (no `x`, `temp`, `data2`)
- [ ] No magic numbers — use named constants instead of raw values like `86400`
- [ ] Error handling is present — no silent catch blocks (`catch(e) {}`)
- [ ] No deeply nested conditionals (more than 3 levels → refactor)

---

## Checklist: Tests

- [ ] New functionality has corresponding unit tests
- [ ] Edge cases are covered (empty input, null values, large inputs)
- [ ] Existing tests still pass (`npm test` / `pytest` / equivalent)
- [ ] Tests are readable — test names describe what they verify
- [ ] No tests that always pass regardless of implementation

**If tests are missing → do not allow the ticket to move to In Review. Offer to help write them.**

---

## Checklist: Functionality

- [ ] The code actually solves the ClickUp ticket requirements
- [ ] All acceptance criteria from the ticket are met
- [ ] No obvious logic errors or off-by-one mistakes
- [ ] Edge cases from the ticket description are handled
- [ ] No breaking changes to existing APIs or interfaces (unless intentional and documented)

---

## Checklist: Style & Conventions

- [ ] Follows the project's naming conventions (camelCase, snake_case, etc.)
- [ ] File structure matches the existing project layout
- [ ] Commit messages follow Conventional Commits format
- [ ] Branch name follows the `feature/[ticket-id]-description` convention
- [ ] No files modified outside the scope of the ticket

---

## Checklist: Performance (Flag, Don't Block)

These are suggestions, not blockers:

- [ ] No N+1 database queries in loops
- [ ] Large arrays are not unnecessarily copied or duplicated
- [ ] No synchronous blocking calls in async code paths
- [ ] Images and assets are optimized if added

---

## Review Output Format

After completing the checklist, Claude summarizes:

```
## Code Review Summary

**Ticket:** [ID] — [Title]
**Branch:** [branch-name]
**Files changed:** [count]

### ✅ Passed
- Security: no secrets detected
- Tests: 4 new tests added, all passing
- Style: follows project conventions

### ⚠️ Warnings (non-blocking)
- consider extracting the auth logic in auth.service.ts into a helper

### ❌ Blockers (must fix before PR)
- console.log found in api/summary.controller.ts line 42
- Missing error handling in the catch block at utils/fetch.ts line 18

**Status:** ❌ Not ready for PR — fix 2 blockers first.
```

---

## How to Use with ClickUp Workflow

This checklist is automatically triggered when:
- A ticket is about to move from **In Progress → In Review**
- The developer explicitly asks for a code review

If all blockers pass → proceed to `pr-description-writer` skill to generate the PR.
