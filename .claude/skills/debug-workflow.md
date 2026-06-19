---
name: debug-workflow
description: "Trigger this skill when a developer is stuck, reports a bug, says 'I don't know why this isn't working', 'something is broken', 'I'm getting an error', 'why is this failing', 'help me debug', or 'I'm confused about this behavior'. Forces a structured debugging process before escalating or changing unrelated code. Prevents random trial-and-error that wastes time and introduces new bugs."
---

# Debug Workflow

## Core Principles

1. **Understand before fixing** — never change code you do not understand
2. **One variable at a time** — isolate the problem before trying solutions
3. **Read the error message fully** — most answers are in the error
4. **Document as you go** — write down what you tried and what happened
5. **Escalate with context** — if asking for help, bring your debug log

---

## Step 1: Stop and Read the Error

Before doing anything else:

1. Read the full error message — not just the first line
2. Identify:
   - **What** went wrong (error type)
   - **Where** it happened (file name + line number)
   - **When** it happens (always, sometimes, on specific input)

Claude will ask:
- "What is the exact error message?"
- "What line does it point to?"
- "When did this start happening — did it ever work?"

---

## Step 2: Reproduce the Bug Reliably

A bug you cannot reproduce consistently is very hard to fix.

1. Find the exact steps that trigger the bug
2. Can you reproduce it every time? If not — is it random or conditional?
3. What is the smallest possible input or action that causes it?

Document your reproduction steps:
```
Steps to reproduce:
1. [action]
2. [action]
3. Bug occurs: [describe what happens]

Expected: [what should happen]
Actual: [what actually happens]
```

---

## Step 3: Form a Hypothesis

Before touching any code, write down your best guess:

```
Hypothesis: I think the bug is caused by [reason]
because [evidence].
```

Examples:
- "I think the API response is returning null because the token expired"
- "I think the loop is running one too many times because the index starts at 1"
- "I think the async function is not being awaited and the value is undefined"

If you cannot form a hypothesis yet, go to Step 4 first.

---

## Step 4: Gather Evidence

Use these techniques to understand what is actually happening:

### Check the logs
```bash
# Check application logs
npm run dev        # watch the console output
# or check log files in /logs
```

### Add temporary debug output
```javascript
console.log('[DEBUG] variable value:', JSON.stringify(variable, null, 2))
console.log('[DEBUG] function reached:', 'functionName')
console.log('[DEBUG] type:', typeof value, 'value:', value)
```

**Important:** Remove all debug logs before committing.

### Check the network (for API issues)
- Open browser DevTools → Network tab
- Check: request URL, method, headers, request body, response status, response body
- Is the API being called at all? Is it returning what you expect?

### Check the data flow
Trace the data from where it enters to where it breaks:
```
Input → [function A] → [function B] → [function C] → Output
                           ↑
                      bug is here?
```

Add a debug log at each step to find exactly where it goes wrong.

---

## Step 5: Test Your Hypothesis

1. Make **one small change** to test your hypothesis
2. Run the app or test again
3. Did it change the behavior?
   - **Yes, fixed** → go to Step 6
   - **Yes, different error** → you found a related issue, update your hypothesis
   - **No change** → your hypothesis was wrong, go back to Step 3

**Never make multiple changes at once** — you will not know which one fixed it.

---

## Step 6: Verify the Fix

Before calling the bug fixed:

1. The original error is gone
2. The feature works as described in the ClickUp ticket
3. You have not broken anything else:
   ```bash
   npm test    # all existing tests must still pass
   ```
4. Remove all debug logs and temporary code

---

## Step 7: Document and Update the Ticket

After fixing the bug:

1. Write a clear commit message explaining what was wrong and what fixed it:
   ```
   fix(auth): resolve null token on session expiry

   The token was not being refreshed before expiry check.
   Added a proactive refresh when token has less than 5 minutes remaining.
   ```

2. Add a comment to the ClickUp ticket:
   ```
   🐛 Bug found and fixed.
   Root cause: [what was wrong]
   Fix: [what was changed and why]
   ```

---

## When to Escalate

Escalate to a senior developer when:

- You have completed Steps 1–5 and still cannot find the cause after 30 minutes
- The bug is in a part of the codebase you do not own or understand
- The fix requires changing shared infrastructure, database schema, or APIs
- The bug appears to be a security vulnerability

### How to Escalate Well

Do not just say "it's broken." Share your debug log:

```
I'm stuck on a bug and need help.

Ticket: [ClickUp link]
Error: [exact error message]
Where: [file and line]
Steps to reproduce: [your steps]
Hypothesis I tested: [what you tried]
Result: [what happened]
What I need: [specific question or help]
```

This shows your work and gets you a faster, better answer.

---

## Common Bug Patterns for New Developers

| Symptom | Likely Cause | What to check |
|---|---|---|
| `undefined` or `null` value | Missing await, wrong variable name, data not loaded yet | Add console.log before the failing line |
| `Cannot read property of undefined` | Object is null or not initialized | Check where the object is created |
| API returns 401 | Token expired or missing | Check Authorization header |
| API returns 404 | Wrong URL or route not registered | Check the URL, check the route file |
| Function not called | Event listener not attached, wrong function name | Add log at function entry |
| Tests failing after your change | You broke existing behavior | Read the test name — it tells you what broke |
| Works locally, broken in staging | Environment variable missing | Check `.env` vs staging config |

---

## What Claude Will Not Do

- Randomly change code hoping it fixes the problem
- Skip straight to a solution without understanding the cause
- Make multiple simultaneous changes to "try things"
- Leave debug logs in the codebase
- Mark a ticket as fixed without running the tests
