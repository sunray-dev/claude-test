---
name: onboarding
description: "Trigger this skill when someone is new to the project, says 'I just joined', 'help me get started', 'walk me through the project', 'first day', 'how do I set up', 'show me the codebase', or 'onboarding'. Also trigger at the start of any session where the developer has never committed to this repo before. Walks new team members through the full project setup and team workflow."
---

# Developer Onboarding

## Welcome

This skill walks new developers through everything they need to know to contribute to this project effectively. Follow each section in order — do not skip ahead.

---

## Step 1: Environment Setup

### Prerequisites Check

Before anything else, verify these are installed:

```bash
node --version      # should be 18+
npm --version       # should be 9+
git --version       # any recent version
gh --version        # GitHub CLI — install if missing
```

Install GitHub CLI if missing:
```bash
# macOS
brew install gh
gh auth login       # authenticate with your GitHub account
```

### Clone the Repository

```bash
git clone https://github.com/sunray-dev/claude-test.git
cd claude-test
```

### Environment Variables

```bash
cp .env.example .env
```

Open `.env` and fill in the required values. Ask your team lead for any secret keys — **never share these in Slack, email, or commit them to git**.

Required variables are documented in `.env.example`. If a value is missing or you are unsure what it should be, ask before guessing.

### Install Dependencies

```bash
npm install
```

### Verify Setup

```bash
npm run dev       # should start without errors
npm test          # all tests should pass
```

If either command fails, do not proceed — fix the error first or ask for help.

---

## Step 2: Tools You Will Use Every Day

| Tool | Purpose | Where to find it |
|---|---|---|
| **ClickUp** | Task and ticket management | app.clickup.com |
| **GitHub** | Code hosting and PRs | github.com/sunray-dev |
| **Claude Code** | AI coding assistant | this tool |
| **Slack** | Team communication | ask your lead for invite |

### ClickUp Setup

1. Ask your team lead to invite you to the ClickUp workspace
2. Workspace ID: `36046572`
3. You will work from the **Backlog** list — tickets assigned to you will appear there
4. Never move a ticket without confirming with the team first

---

## Step 3: Project Structure

```
/
├── .claude/
│   ├── skills/          # Claude Code workflow skills
│   └── settings.json    # hooks and automation config
├── src/                 # main application source code
├── tests/               # all test files live here
├── .env.example         # environment variable template
├── CLAUDE.md            # project instructions for Claude
└── README.md
```

Key rules:
- All new code goes in `src/`
- All tests go in `tests/` — mirror the `src/` folder structure
- Never modify `.claude/` files without team approval

---

## Step 4: Your Daily Workflow

Every working day follows this loop:

### Morning
1. Open Claude Code in the project directory
2. Claude will automatically fetch your ClickUp tickets and show what to work on
3. Pick a ticket and confirm you are starting it
4. Claude moves it to **In Progress** and creates the branch for you

### During the Day
1. Work on the ticket — commit frequently with clear messages
2. If you get stuck, use the `debug-workflow` skill before asking for help
3. If you are blocked by something external, add a blocker comment to the ClickUp ticket

### End of Day
1. Push your branch: `git push origin [your-branch]`
2. Add a progress comment to the ClickUp ticket
3. Note what you will continue tomorrow

### When You Finish a Ticket
1. Run the `code-review-checklist` skill to verify your code
2. Use `pr-description-writer` to generate your PR
3. Move the ClickUp ticket to **In Review**
4. Tag a reviewer in the PR

---

## Step 5: Git Rules (Read Carefully)

These are non-negotiable:

1. **Never commit to `main` or `dev`** — always use a feature branch
2. **Branch naming:** `feature/[ticket-id]-short-description`
3. **Commit messages:** use Conventional Commits format (see `git-workflow` skill)
4. **One ticket per branch** — do not mix work
5. **Pull before you start** — always `git pull origin dev` before creating a branch

If you are unsure about any git operation, ask Claude first.

---

## Step 6: Getting Help

| Situation | What to do |
|---|---|
| Stuck on a bug | Use `debug-workflow` skill first |
| Unclear ticket requirements | Comment on the ClickUp ticket and ask |
| Git conflict | Use `git-workflow` skill for step-by-step help |
| Blocked by another person or ticket | Add a blocker comment to your ticket |
| Something feels wrong with the codebase | Tell your team lead — never work around it silently |

### Asking Good Questions

Before asking a teammate for help, always share:
1. What you are trying to do
2. What you tried
3. What happened (exact error message or behavior)
4. What you expected to happen

This saves everyone time and helps you think through the problem.

---

## Step 7: Checklist Before Your First Commit

- [ ] Environment is set up and `npm run dev` works
- [ ] `npm test` passes with no failures
- [ ] You are invited to ClickUp and can see the Backlog
- [ ] You are added to the GitHub org (sunray-dev)
- [ ] You have a ticket assigned and it is In Progress
- [ ] Your branch follows the naming convention
- [ ] You understand the daily workflow

Once all items are checked — you are ready to contribute. Welcome to the team.
