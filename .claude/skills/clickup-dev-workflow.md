---
name: clickup-dev-workflow
description: "Trigger this skill whenever a developer mentions a ClickUp ticket, starts working on a feature, asks to update ticket status, or begins a coding session. Covers: fetching ticket details before coding, moving tickets through statuses (Backlog → In Progress → In Review → Done), adding progress comments, and enforcing quality checks before closing tickets. Also triggers on session start if open tickets are mentioned, or when a developer says 'what should I work on', 'start ticket', 'mark as done', 'I finished', or similar phrases."
---

# ClickUp Developer Workflow

## Core Principles

1. **Always fetch before acting** — read the ticket details before starting any work
2. **Status reflects reality** — only move a ticket when the work state actually changed
3. **Comment everything** — every status change gets a comment explaining why
4. **Never skip statuses** — Backlog → In Progress → In Review → Done, always in sequence
5. **Quality before closing** — tests must pass before moving to Done
6. **Blocked means commented** — if stuck, add a blocker comment instead of moving status

---

## Workspace Reference

```
Workspace ID:   36046572
Space:          Claude Test
Space URL:      https://app.clickup.com/36046572/v/s/901511152790

List IDs:
  Backlog:      901523813899
  In Progress:  901523813907
  In Review:    901523813910
  Done:         901523813912
```

---

## Workflow: Session Start

When a developer starts a Claude Code session and wants to know what to work on:

1. Fetch all tasks from the Backlog list filtered by status `unstarted`
2. Present the list ordered by priority (urgent → high → normal → low)
3. Ask the developer which ticket they want to start
4. Fetch full ticket details including subtasks and acceptance criteria
5. Display a working brief:
   - Ticket title and ID
   - Full description
   - Acceptance criteria checklist
   - Any dependencies or blockers noted in comments
6. Ask: "Ready to start? I'll move this to In Progress."
7. On confirmation → move to In Progress + add comment (see comment templates below)

---

## Workflow: Starting a Ticket

When developer says: "start ticket [name/ID]", "work on [feature]", "begin [task]"

1. Search for the ticket by name or ID
2. Fetch full details (description, acceptance criteria, subtasks, current status)
3. Confirm with developer: show ticket summary and ask if this is the right one
4. Move ticket from Backlog → In Progress
5. Add comment:
   ```
   🚀 Started development.
   Branch: [ask developer for branch name or read from git]
   Developer: [name if known]
   ```
6. If ticket has subtasks, show them as a checklist for the developer to work through

---

## Workflow: Updating Progress

When developer says: "update ticket", "add progress", "I finished [subtask]"

1. Fetch current ticket status and comments
2. Add a progress comment with what was completed
3. If all subtasks are done → suggest moving to In Review
4. Never move status without asking the developer to confirm

Comment format:
```
📝 Progress update:
- Completed: [what was done]
- Remaining: [what is left]
- Notes: [any relevant details, decisions made, issues encountered]
```

---

## Workflow: Moving to In Review

When developer says: "done", "ready for review", "finished implementation", "PR ready"

1. Fetch current ticket and verify it is In Progress
2. Run quality checks before allowing status change:
   - Ask: "Have tests been run and passing?"
   - Ask: "Is there a PR or branch ready for review?"
3. If quality checks pass → move In Progress → In Review
4. Add comment:
   ```
   👀 Ready for review.
   PR: [link if available]
   What was implemented: [1-2 sentence summary]
   Acceptance criteria met:
   - [x] criterion 1
   - [x] criterion 2
   Testing notes: [what was tested and how]
   ```
5. If subtasks exist, verify all are complete before moving parent ticket

---

## Workflow: Moving to Done

When developer or reviewer says: "approved", "merged", "close ticket", "mark as done"

1. Fetch ticket and verify it is In Review
2. Confirm: "Are you sure this is ready to close? This should only happen after review/merge."
3. On confirmation → move In Review → Done
4. Add comment:
   ```
   ✅ Completed.
   Closed by: [developer name]
   Summary: [what was delivered in 1-2 sentences]
   Merged: [yes/no + PR link if available]
   ```

---

## Workflow: Blocked Ticket

When developer says: "I'm blocked", "can't proceed", "waiting on [something]"

1. Do NOT move the ticket status
2. Add a blocker comment:
   ```
   🚫 Blocked.
   Reason: [what is blocking progress]
   Waiting on: [person, ticket, or external dependency]
   Estimated unblock: [date or unknown]
   ```
3. Suggest: "Should I flag this to the PM by adding a high priority tag?"

---

## Workflow: Checking Ticket Status

When developer says: "what's the status", "show open tickets", "what's in progress"

1. Fetch tasks filtered by the requested status
2. Display in a clean list:
   ```
   📋 In Progress (2)
   ├── [ID] Backend: Build summarization service — High
   └── [ID] Frontend: Display summary beneath article title — Normal

   📋 Backlog (5)
   ├── [ID] Research & select AI summarization API — High
   ...
   ```
3. Ask: "Want to start one of these or update an existing ticket?"

---

## Comment Templates (Quick Reference)

| Situation | Emoji | Key fields |
|---|---|---|
| Started work | 🚀 | Branch, developer |
| Progress update | 📝 | Completed, remaining, notes |
| Ready for review | 👀 | PR link, criteria met, testing notes |
| Completed | ✅ | Summary, merge link |
| Blocked | 🚫 | Reason, waiting on, estimated unblock |
| Question/decision | 💬 | Context, options considered, decision made |

---

## Branch Naming Convention

Use this convention so hooks and middleware can extract ticket IDs automatically:

```
feature/[ticket-id]-short-description
bugfix/[ticket-id]-short-description
chore/[ticket-id]-short-description

Examples:
  feature/86ca6auzt-ai-summary
  bugfix/86ca6av5a-fix-summarization-cache
  chore/86ca6av9f-add-summary-field
```

---

