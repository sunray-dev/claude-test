---
name: objective-code-review
description: >
  Runs an objective code review of the current branch diff by spawning a fresh subagent (claude-sonnet-4-6)
  that has no context from the current conversation — so its findings are unbiased. Reviews for bugs,
  security vulnerabilities, and simplification opportunities, then returns a structured list of findings.
  Use this skill whenever the user asks to "review my code", "check this branch", "do a code review",
  "review the diff", "look for bugs", "security review", or any similar request about reviewing changes
  on the current branch. Prefer this over doing an inline review yourself — the separate context is the point.
---

# Objective Code Review

The goal of this skill is to give the user an unbiased code review by delegating the actual review
to a fresh subagent that has seen none of the current conversation. This matters because an inline
reviewer (you) has heard the user's explanations, design rationale, and context — which can soften
findings. The subagent gets only the raw diff and repo info.

## Your role (the orchestrator)

You are NOT the reviewer. Your job is to:
1. Gather the diff and context
2. Spawn the reviewer subagent
3. Present its findings to the user cleanly

## Step 1 — Gather the diff

Run these commands to get what the reviewer needs:

```bash
# Current branch name
git branch --show-current

# Diff vs main (or master if main doesn't exist)
git diff main...HEAD 2>/dev/null || git diff master...HEAD 2>/dev/null || git diff HEAD~1

# File list for context
git diff --name-only main...HEAD 2>/dev/null || git diff --name-only HEAD~1
```

If the diff is empty, tell the user there are no changes to review and stop.

## Step 2 — Spawn the reviewer subagent

Use the Agent tool with these exact settings:
- `model: "sonnet"` — this ensures it runs as claude-sonnet-4-6
- The prompt must be entirely self-contained — include the full diff inline

**Subagent prompt template:**

```
You are doing an objective code review. You have no context from any prior conversation — just the diff below.

Review this diff for:
1. **Bugs** — logic errors, off-by-ones, null/undefined risks, incorrect assumptions, race conditions
2. **Security** — injection vulnerabilities, insecure defaults, exposed secrets, missing auth/validation, OWASP Top 10
3. **Simplifications** — dead code, unnecessary complexity, duplication, better stdlib/language features available

Branch: <branch name>
Changed files: <file list>

--- DIFF START ---
<full diff output>
--- DIFF END ---

Return your findings as a structured list using this exact format:

## Bugs
- [FILE:LINE or general] Description of the bug and why it's a problem.
(or "None found." if clean)

## Security
- [FILE:LINE or general] Description of the vulnerability and the risk.
(or "None found." if clean)

## Simplifications
- [FILE:LINE or general] What to simplify and why it's better.
(or "None found." if clean)

## Summary
One paragraph overall assessment. Be direct — don't soften findings to be polite.

Be specific: reference file names and line numbers where possible. If there are no issues in a category, say so explicitly.
```

## Step 3 — Present the results

Take the subagent's output verbatim and present it to the user. Add a short header like:

> **Code Review** (reviewed by a fresh subagent with no conversation context)

Don't editorialize, summarize, or soften the findings — the whole point is objectivity.
If the user wants to discuss a finding, you can do that inline after presenting.
