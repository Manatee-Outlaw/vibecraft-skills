---
name: production-drift
description: >
  Audit the gap between what's in version control (GitHub) and what's
  actually running in production. Detects untracked changes, config drift,
  package drift, and undocumented infrastructure. Designed for single-server
  deployments (Raspberry Pi, VPS) and expandable to cloud environments.
  Read-only — produces a drift report, makes no changes.
---

# Production Drift Audit Skill

## When to use
- After a period of rapid development across multiple sessions
- Before a major release or infrastructure change
- When something works in production but can't be explained by git history
- After any manual server-side changes (crontab, .env, config files)
- Periodically as a health check (monthly or after major sprints)

This skill is read-only. It produces a report. Do not make changes.

---

## Step 1 — Git sync check

Verify the production server is in sync with the canonical git repository.

```bash
# On the production server
git log --oneline -5
git status
git diff HEAD
git remote get-url origin
```

Check:
- Is HEAD on the production server the same commit as origin/main?
- Are there any uncommitted changes to tracked files?
  (git status shows modified files, git diff HEAD shows the changes)
- Are there any merge conflicts or detached HEAD states?
- Does the remote URL point to the correct repository?

Flag:
- Production behind origin/main (commits not yet deployed)
- Production ahead of origin/main (local commits not pushed — dangerous,
  these changes exist only on the server and could be lost)
- Uncommitted changes to tracked files (CRITICAL — not in git, not
  recoverable if the server dies)
- Wrong remote URL

---

## Step 2 — Untracked file detection

Find files that exist in the production directory but aren't tracked by git:

```bash
git ls-files --others --exclude-standard
```

For each untracked file:
- Is it intentionally excluded by .gitignore (e.g., .env, *.db, __pycache__)?
- Or is it a file that SHOULD be in git but was accidentally created
  directly on the server?
- Or is it a file that SHOULD be documented somewhere (config, script)?

Also check for untracked directories:
```bash
find . -not -path './.git/*' -not -path './node_modules/*' -type d | \
  git check-ignore --stdin
```

---

## Step 3 — Crontab documentation audit

Read the production crontab and compare against any documentation:

```bash
crontab -l
```

For each cron entry:
- Is it documented anywhere? (README, CLAUDE.md, architecture docs)
- Was it added via git (in a setup script) or directly on the server?
- Is the command path still valid? Do the referenced scripts exist?
- Is the log file destination still valid?
- Does it match what the project documentation says should be running?

Flag:
- Cron jobs with no documentation anywhere
- Cron jobs that reference files not in git
- Cron jobs that were manually added and exist only in the crontab

---

## Step 4 — Execute permission audit (CRITICAL for cron)

This step catches silent failures where cron jobs appear correct in the
crontab but never execute because the script file lacks execute permission.
This is a common source of silent failures — the cron job fires on schedule,
gets "Permission denied", writes nothing to logs, and no alert is raised.

Check git-tracked file modes for all scripts called by cron:

```bash
# Check git file modes — should be 100755 for executable scripts, not 100644
git ls-files -s *.sh *.py | grep -v "100755"
```

Any .sh file showing mode 100644 (non-executable) will fail silently in cron.

Also verify execute bits on the production server:

```bash
ls -la *.sh *.py | grep -v "^-rwx"
```

Scripts called by cron that are missing the execute bit:

```bash
# For each script in crontab, check if it's executable
# Example: <scheduled-script>.sh, <scheduled-script>.py
stat -c "%a %n" <scheduled-script>.sh <scheduled-script>.py 2>/dev/null
```

Flag:
- Any .sh script tracked in git with mode 100644 (should be 100755)
- Any script called by cron that is not executable on the server
- Mismatch between git mode and server permissions (deploy will reset server bit)

Fix (when found):
```bash
git update-index --chmod=+x script_name.sh
git commit -m "fix: restore execute permission on script_name.sh"
```
Never use bare chmod +x without also fixing it in git — the next deploy
will strip the permission bit again, recreating the same silent failure.

**General rule:** Any file called directly by cron or a process manager
MUST have mode 100755 in git. Verify this after every session that adds
or modifies shell scripts.

---

## Step 5 — Environment variable drift

Compare the production .env against any template or documented variables:

```bash
# List keys only (never print values — they're secrets)
cat /path/to/.env | cut -d= -f1 | sort
```

Check:
- Are there variables in .env that aren't in any template or documentation?
  (Undocumented config — if the server dies, these are lost)
- Are there variables documented or referenced in code that aren't in .env?
  (Missing config — code may fail silently)
- Has any variable been changed from its last documented value?
  (Document when this was changed and why, without exposing the value)

Never print or log the actual values of environment variables.

---

## Step 6 — Package drift

Compare installed packages against the requirements file:

```bash
# What's actually installed
pip3 list --format=freeze | sort

# What's required
cat requirements.txt | sort
```

Check:
- Are there packages installed that aren't in requirements.txt?
  (Manual installs that would be lost on a fresh deploy)
- Are there packages in requirements.txt that aren't installed?
  (Requirements file out of date)
- Are installed versions different from pinned versions in requirements.txt?
  (Version drift — may cause non-reproducible builds)

Flag:
- Any installed package not in requirements.txt (deployment gap)
- Major version differences between installed and pinned versions

---

## Step 7 — Config file drift

Check any configuration files that live outside git:

```bash
# System service file
sudo cat /etc/systemd/system/<your-app>.service

# Logrotate config
sudo cat /etc/logrotate.d/<your-app>

# Any nginx/apache config if applicable
```

For each config file:
- Is it documented anywhere in the project?
- Was it set up via a documented process or manually?
- Does it match what the project documentation says it should contain?
- Would a fresh deployment recreate it correctly?

---

## Step 8 — Disk and filesystem health

Quick sanity check on the server's storage:

```bash
df -h
du -sh /path/to/project/
du -sh /path/to/project/* | sort -h | tail -20
```

Flag:
- Disk usage above 70% (planning threshold)
- Disk usage above 85% (action threshold)
- Any unexpected large files or directories
- Log files or data directories growing faster than expected

---

## Step 9 — Process and service health

Verify the application is running as expected:

```bash
# Service status
sudo systemctl status <service>

# What's actually running
ps aux | grep -E 'gunicorn|python|node'

# Port bindings
sudo ss -tlnp | grep -E '5000|8000|80|443'
```

Check:
- Is the service running under the expected user?
- Are the expected number of workers running?
- Is the application bound to the expected port?
- Is there anything unexpected running on the server?

---

## Output format

Produce a report with these sections:

### DRIFT SUMMARY
One-line status: "X items in sync, Y items drifted, Z items undocumented"

### CRITICAL FINDINGS
Changes that exist only on the production server and are not in git.
These are at risk of being lost if the server fails.

### GIT SYNC STATUS
Commit comparison between production and origin/main.

### EXECUTE PERMISSION AUDIT
List every cron-called script. Flag any missing execute bit or git mode mismatch.

### UNTRACKED FILES
Files on the server not in git, with assessment of whether each is
intentional (in .gitignore) or a concern.

### CRONTAB AUDIT
Every cron job with documentation status and any unexpected entries.

### ENVIRONMENT VARIABLE AUDIT
Key names present/absent. No values. Flag undocumented keys.

### PACKAGE DRIFT
Packages installed but not in requirements.txt, and vice versa.

### CONFIG FILE STATUS
System config files with documentation status.

### DISK HEALTH
Storage usage summary and any growth concerns.

### REMEDIATION CHECKLIST
Ordered list of actions to bring production back in sync with git,
and to document anything that should be tracked but isn't.

### EXPANDING TO OTHER ENVIRONMENTS
When a project migrates from one host to another, re-run this audit
with the following adjustments:
- Replace crontab check with cloud scheduler (AWS EventBridge, GCP Cloud
  Scheduler, etc.)
- Replace .env check with secrets manager (AWS Secrets Manager, etc.)
- Replace systemd service check with container/process manager check
- Replace disk usage with cloud storage/volume metrics
- Add: load balancer config, CDN config, DNS records documentation
