---
name: push-to-git
description: >
  Commit all changes, push to GitHub, and auto-deploy to the Pi in one step.
  Trigger immediately when the user says "push to git" — with or without a commit
  message. Also triggers on "deploy", "ship it", "push changes".
---

# Push to Git

Commits all staged and unstaged changes, pushes to GitHub, and pulls + restarts
the service on the Pi. One command, full deploy cycle.

## Trigger
`push to git` / `push to git [message]` / `deploy` / `ship it` / `push changes`

---

## Execution — Run All Steps in Order

### Step 1 — Get the commit message

If the user included a message (e.g. "push to git fix discord bot"), use that as
the commit message.

If no message was provided, ask:
> "What's the commit message? (Or say 'default' for a timestamp.)"

If the user says "default" or similar, generate: `Deploy YYYY-MM-DD HH:MM` using
today's date.

---

### Step 2 — Create task.md

Use bash_tool to create `/mnt/user-data/outputs/task.md` with the following content,
substituting [MESSAGE] with the commit message from Step 1:

```
Run these four commands in order. Show the full output of each before moving to the
next. Stop and report if any command fails.

1. git add -u
2. git commit -m "[MESSAGE]"
3. git push
4. ssh <user>@<production-server> "cd <project-dir> && git pull origin main && sudo systemctl restart <service> && echo 'Deploy complete'"
```

Note: use `git add -u` (updates tracked files only) not `git add .` (adds all untracked
files). If there are intentional new files to include, add them explicitly with
`git add <filename>` before running this task.

---

### Step 3 — Present the file

Present task.md for download. Tell the user:

> "Save this to `<projects-dir>\<your-repo>\task.md`, then in Claude Code type:
> `read task.md and execute all commands`"

---

### Step 4 — Confirm

After the user pastes the Claude Code output back here:

- If all four commands succeeded and "Deploy complete" appears → confirm the deploy is live
- If git commit shows "nothing to commit" → tell the user there were no changes to push
- If git push fails with auth error → SSH key needs re-adding to GitHub
  (Settings → SSH keys → New → Authentication Key → paste `cat $HOME\.ssh\id_ed25519.pub`)
- If SSH to Pi fails → check Pi is reachable: ping <production-server>
- If systemctl restart fails → check gunicorn_error.log on the Pi for Python syntax errors

---

## Hard Rules

- Never skip Step 4 — always confirm the Pi pulled successfully, not just that the push succeeded
- If the user says the Pi SSH step failed but push succeeded, the code is on GitHub
  but NOT live — flag this clearly
- Always use the exact SSH command above — do not abbreviate or skip the restart
- Never use `git add .` blindly — use `git add -u` for modified tracked files
