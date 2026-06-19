---
name: git-workflow
description: "Trigger this skill whenever a developer mentions branching, committing, pushing, merging, rebasing, or creating a PR. Also triggers on: 'create a branch', 'how should I name this branch', 'ready to push', 'commit my changes', 'merge this', 'rebase', 'I have a conflict', 'squash commits', or any git-related action. Enforces consistent git hygiene across the team."
---

# Git Workflow

## Core Rules

1. **Never commit directly to `main` or `dev`** — always work on a feature branch
2. **One ticket = one branch** — do not mix multiple tickets in one branch
3. **Commit often, push daily** — small commits are easier to review and revert
4. **Always pull before starting work** — avoid diverged branches
5. **No force pushes to shared branches** — coordinate with the team first

---

## Branch Naming Convention

```
feature/[ticket-id]-short-description
bugfix/[ticket-id]-short-description
chore/[ticket-id]-short-description
hotfix/[ticket-id]-short-description
docs/[ticket-id]-short-description

Examples:
  feature/86ca6auzt-ai-summary
  bugfix/86ca6av5a-fix-summarization-cache
  chore/86ca6av9f-add-summary-field
  hotfix/86ca6aw1b-fix-login-crash
```

Rules:
- All lowercase, hyphens only (no underscores or spaces)
- Include the ClickUp ticket ID so hooks can auto-link
- Keep the description short (3-5 words max)

---

## Workflow: Starting New Work

When developer says: "start a branch", "create branch", "begin work on [ticket]"

1. Confirm the ClickUp ticket ID they are working on
2. Pull latest from the base branch:
   ```bash
   git checkout dev
   git pull origin dev
   ```
3. Create and checkout the new branch:
   ```bash
   git checkout -b feature/[ticket-id]-short-description
   ```
4. Confirm branch was created and remind developer to update the ClickUp ticket to In Progress

---

## Commit Message Format

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <short summary>

[optional body]

[optional footer]
```

**Types:**
| Type | When to use |
|---|---|
| `feat` | New feature |
| `fix` | Bug fix |
| `chore` | Tooling, dependencies, config |
| `docs` | Documentation only |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `test` | Adding or updating tests |
| `style` | Formatting, missing semicolons (no logic change) |

**Examples:**
```
feat(auth): add JWT refresh token support
fix(api): handle null response from summarization service
chore(deps): upgrade axios to 1.6.0
docs(readme): add local setup instructions
test(auth): add unit tests for token expiry
```

Rules:
- Summary line max 72 characters
- Use imperative mood ("add" not "added" or "adds")
- Reference ClickUp ticket in footer if relevant: `Refs: #86ca6auzt`

---

## Workflow: Committing Changes

When developer says: "commit my changes", "save my work"

1. Show what will be committed:
   ```bash
   git status
   git diff --staged
   ```
2. Check for issues before committing:
   - No `.env` files staged
   - No `console.log` or debug statements
   - No hardcoded API keys or secrets
3. If clean → suggest a commit message following the convention above
4. Commit:
   ```bash
   git add [specific files]   # prefer specific files over git add -A
   git commit -m "feat(scope): description"
   ```

---

## Workflow: Pushing and Creating a PR

When developer says: "push my changes", "open a PR", "ready for review"

1. Ensure all changes are committed
2. Push the branch:
   ```bash
   git push -u origin [branch-name]
   ```
3. Trigger the `pr-description-writer` skill to generate the PR description
4. Update the ClickUp ticket to In Review (via `clickup-dev-workflow` skill)

---

## Workflow: Handling Merge Conflicts

When developer says: "I have a conflict", "merge conflict", "can't merge"

1. Do NOT panic — conflicts are normal
2. Pull the latest base branch:
   ```bash
   git fetch origin
   git rebase origin/dev
   ```
3. Open conflicting files — look for `<<<<<<`, `======`, `>>>>>>` markers
4. Resolve each conflict manually — keep the correct code, remove markers
5. After resolving:
   ```bash
   git add [resolved files]
   git rebase --continue
   ```
6. If stuck → abort and ask for help:
   ```bash
   git rebase --abort
   ```

---

## Workflow: Cleaning Up After Merge

After a PR is merged:

1. Switch back to base branch and pull:
   ```bash
   git checkout dev
   git pull origin dev
   ```
2. Delete the local branch:
   ```bash
   git branch -d feature/[ticket-id]-description
   ```
3. Delete the remote branch if not auto-deleted:
   ```bash
   git push origin --delete feature/[ticket-id]-description
   ```

---

## What Claude Will Never Do

- Force push to `main` or `dev`
- Commit directly to `main` or `dev`
- Run `git add -A` without first showing `git status`
- Amend commits that have already been pushed to a shared branch
- Skip commit message conventions
