---
name: pr-description-writer
description: "Trigger this skill when a developer is ready to open a pull request, says 'write my PR', 'generate PR description', 'create PR', 'open pull request', or 'I'm ready to submit'. Also triggers automatically after the code-review-checklist passes. Reads the git diff and ClickUp ticket to generate a complete, structured PR description."
---

# PR Description Writer

## Core Principles

1. **PR descriptions are communication** — the reviewer should understand the change without reading all the code
2. **Link everything** — every PR must reference its ClickUp ticket
3. **Show don't tell** — include screenshots for UI changes, curl examples for API changes
4. **Be honest about risk** — flag areas that need careful review

---

## Workflow: Generating a PR Description

When developer says: "write my PR", "generate PR description", "open a PR"

1. Read the ClickUp ticket details (title, description, acceptance criteria)
2. Run `git diff main...HEAD` to understand what changed
3. Run `git log main...HEAD --oneline` to see the commit history
4. Identify the type of change (feature / bugfix / chore / refactor)
5. Generate the PR description using the template below
6. Ask the developer:
   - "Is there a screenshot or recording to attach for UI changes?"
   - "Is there anything risky in this PR reviewers should focus on?"
7. Create the PR via GitHub CLI or provide the final description to copy

---

## PR Description Template

```markdown
## Summary

[1-3 bullet points describing what this PR does and why]

- Implements [feature/fix] as described in [ClickUp ticket link]
- [What the key change is]
- [Any important decision made during implementation]

## ClickUp Ticket

[Ticket ID and link]

## Type of Change

- [ ] New feature
- [ ] Bug fix
- [ ] Refactor (no functional change)
- [ ] Chore / dependency update
- [ ] Documentation

## What Changed

[Brief description of the key files/components modified and why]

| File | Change |
|---|---|
| `path/to/file.ts` | Added [what] |
| `path/to/other.ts` | Updated [what] |

## Acceptance Criteria

[Copy from the ClickUp ticket and mark each as met]

- [x] Criterion 1
- [x] Criterion 2
- [ ] Criterion 3 (if not met, explain why)

## How to Test

[Step-by-step instructions for the reviewer to verify the change works]

1. Checkout this branch: `git checkout [branch-name]`
2. Run: `npm install && npm run dev`
3. Navigate to [URL or describe the flow]
4. Verify: [what the reviewer should see]

## Screenshots / Recordings

[Attach if UI changes are involved — before and after if applicable]

## Areas Needing Extra Attention

[Flag anything complex, risky, or that deviates from the original plan]

## Checklist

- [ ] Tests written and passing
- [ ] No hardcoded secrets
- [ ] No debug statements left in code
- [ ] Branch is up to date with `dev`
- [ ] ClickUp ticket moved to In Review
```

---

## Workflow: Creating the PR via CLI

After the description is ready:

```bash
gh pr create \
  --title "[ticket-id] Short description of change" \
  --body "$(cat pr-description.md)" \
  --base dev \
  --head feature/[ticket-id]-description
```

Or for a draft PR (not ready for review yet):
```bash
gh pr create --draft \
  --title "[ticket-id] WIP: Short description" \
  --body "$(cat pr-description.md)" \
  --base dev
```

---

## PR Title Format

```
[ticket-id] type: short description

Examples:
  [86ca6auzt] feat: add AI summarization endpoint
  [86ca6av5a] fix: handle null response from summary service
  [86ca6av9f] chore: upgrade dependencies to latest stable
```

---

## After PR is Created

1. Copy the PR link
2. Update the ClickUp ticket comment with the PR link (via `clickup-dev-workflow` skill)
3. Move the ClickUp ticket to **In Review**
4. Notify the reviewer (tag them in the PR or message in team chat)

---

## What Makes a Bad PR (Claude Will Flag These)

- Title is vague: "fixes", "updates", "changes stuff"
- No ClickUp ticket linked
- PR contains changes from multiple unrelated tickets
- PR is more than 500 lines changed without justification
- No test plan or testing instructions
- UI changes with no screenshots
