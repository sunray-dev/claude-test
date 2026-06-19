---
name: env-setup-guard
description: "Trigger this skill when a developer mentions environment variables, .env files, API keys, secrets, or config. Also triggers automatically before any git commit, git push, or PR creation to scan for accidentally exposed secrets. Triggers on: 'set up env', 'I added an API key', 'configure secrets', 'env is not working', 'missing environment variable', or any time a .env file is mentioned."
---

# Environment Setup Guard

## Core Principles

1. **Secrets never go in code** — use environment variables, always
2. **`.env` files never get committed** — not even once, not even to test
3. **Every secret must have a name** — no hardcoded strings that look like keys
4. **All required vars must be documented** — in `.env.example`, never in `.env`
5. **Staging and production are separate** — never use prod secrets locally

---

## Automatic Pre-Commit Secret Scan

Before every commit, Claude will scan staged files for:

### Patterns That Block the Commit

| Pattern | Risk Level | Example |
|---|---|---|
| `sk-[a-zA-Z0-9]{32,}` | 🔴 Critical | OpenAI API key |
| `ghp_[a-zA-Z0-9]{36}` | 🔴 Critical | GitHub Personal Access Token |
| `AKIA[A-Z0-9]{16}` | 🔴 Critical | AWS Access Key |
| `password\s*=\s*['"][^'"]{6,}` | 🔴 Critical | Hardcoded password |
| `secret\s*=\s*['"][^'"]{8,}` | 🔴 Critical | Hardcoded secret |
| `api_key\s*=\s*['"][^'"]{8,}` | 🔴 Critical | Hardcoded API key |
| `bearer [a-zA-Z0-9\-_]{20,}` | 🔴 Critical | Bearer token in code |
| `.env` file staged | 🔴 Critical | Committed .env file |

### Patterns That Warn (Non-Blocking)

| Pattern | Risk Level | Note |
|---|---|---|
| `localhost:[0-9]{4}` | 🟡 Warning | Hardcoded local URL |
| `127.0.0.1` | 🟡 Warning | Hardcoded local IP |
| `TODO.*secret` | 🟡 Warning | Secret placeholder left in |
| Credentials in comments | 🟡 Warning | Remove before committing |

**If any critical pattern is found → the commit is blocked until resolved.**

---

## Workflow: Setting Up Environment Variables

When developer says: "set up env", "add an API key", "configure [service]"

1. Check if `.env.example` exists — if not, create it
2. Add the new variable to `.env.example` with a placeholder:
   ```
   SERVICE_API_KEY=your_service_api_key_here
   SERVICE_BASE_URL=https://api.service.com
   ```
3. Add the real value to `.env` (never commit this file)
4. Verify `.env` is in `.gitignore`:
   ```bash
   grep -n ".env" .gitignore
   ```
5. If `.env` is not in `.gitignore` → add it immediately
6. Use the variable in code via `process.env.SERVICE_API_KEY` — never paste the raw value

---

## Required .gitignore Entries

Claude will verify these are present in `.gitignore`:

```gitignore
# Environment variables
.env
.env.local
.env.*.local
.env.development
.env.production
.env.staging

# Secrets and credentials
*.pem
*.key
*.p12
*.pfx
credentials.json
service-account.json
```

If any are missing, Claude adds them automatically and flags a warning.

---

## Workflow: Diagnosing Missing Environment Variables

When developer says: "env is not working", "undefined env variable", "missing config"

1. List all variables referenced in the codebase:
   ```bash
   grep -r "process.env\." src/ --include="*.ts" --include="*.js" | \
     grep -oP 'process\.env\.\K[A-Z_]+' | sort -u
   ```
2. Compare with what is defined in `.env.example`
3. Check if the variable exists in the running environment:
   ```bash
   # Check if a specific var is set (shows only if it exists, not the value)
   node -e "console.log('VAR exists:', !!process.env.YOUR_VAR_NAME)"
   ```
4. Identify which variable is missing and where to get its value
5. Update `.env` with the missing value
6. Restart the dev server — env vars are loaded at startup

---

## .env.example Template

Every project should have a `.env.example` at the root:

```bash
# Application
NODE_ENV=development
PORT=3000
APP_URL=http://localhost:3000

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/dbname

# Authentication
JWT_SECRET=your_jwt_secret_here
JWT_EXPIRY=7d

# External APIs (get keys from team lead)
OPENAI_API_KEY=your_openai_api_key_here
CLICKUP_API_KEY=your_clickup_api_key_here

# Email
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=your_email@example.com
SMTP_PASS=your_smtp_password_here
```

Rules for `.env.example`:
- Every variable must have a descriptive placeholder value
- Group related variables with a comment header
- Note which values require a real key vs. can stay as-is locally
- Keep this file up to date — it is the source of truth for new developers

---

## Workflow: Rotating a Leaked Secret

If a secret is accidentally committed to git:

1. **Revoke the secret immediately** — do not wait
   - GitHub token → github.com/settings/tokens
   - API key → the service's dashboard
   - Database password → database admin panel

2. Generate a new secret/key from the provider

3. Remove the secret from git history:
   ```bash
   git filter-branch --force --index-filter \
     "git rm --cached --ignore-unmatch path/to/file" \
     --prune-empty --tag-name-filter cat -- --all
   ```
   Or use the simpler BFG Repo Cleaner tool.

4. Force push (this is one of the rare valid use cases):
   ```bash
   git push origin --force --all
   ```

5. Notify the team lead — assume the secret was compromised

6. Update `.env` with the new secret

7. Add the new secret to all deployment environments

**Do not just delete the file in a new commit — the secret is still in git history.**

---

## What Claude Will Never Do

- Stage or commit a `.env` file
- Write a real API key or secret directly into source code
- Suggest storing secrets in comments "just temporarily"
- Skip the secret scan before a commit
- Recommend committing credentials to test branches
