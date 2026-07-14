---
name: security-audit
description: >
  Focused security review of a web application. Covers authentication
  inventory, input validation, injection risks, XSS exposure, credential
  handling, session security, file system risks, and rate limiting gaps.
  Produces a prioritized findings report without making any changes.
---

# Security Audit Skill

## When to use
- Before exposing a new endpoint to the public
- After significant feature additions that touch auth or user input
- As part of a comprehensive code audit
- After a security incident or near-miss
- Before onboarding external users to a platform

This skill is read-only. It produces a report. Do not make changes.

---

## Step 1 — Route auth inventory

Read every Flask route (or equivalent) in the application. For each route:

| Route | Method | Auth Decorator | Who Can Call It | Notes |
|-------|--------|---------------|-----------------|-------|

Auth decorators to look for:
- @require_auth (Bearer JWT)
- @require_manager
- @require_session_auth
- @require_scout
- <scoped-token> (?token= param or X-Overlay-Token header)
- No decorator (public)

Flag:
- Routes with no auth that serve or modify non-public data (HIGH)
- Routes where the auth level seems wrong for the sensitivity of the data
- Routes that return different data based on role but don't validate role
  on the server (trusts client)
- Admin/debug routes accidentally left public

---

## Step 2 — Input validation audit

For every POST, PUT, and PATCH route, check:
- Is the request body validated before use?
- Are string lengths bounded?
- Are enum fields validated against an allowed list?
- Are integer fields checked for range?
- Are file paths or filenames sanitized before use in filesystem calls?
- Is the handle/username format validated before database writes?
  (Prevents garbage data and potential injection)

Flag:
- Fields accepted without any validation (HIGH if they reach SQL or filesystem)
- Validation that's present on the client but absent on the server (HIGH —
  client validation is convenience, server validation is security)
- Fields that accept arbitrary strings and write them to the database
  without length limits

---

## Step 3 — SQL injection scan

Search for every database query in the application. For each:
- Is it using parameterized queries (?, :name placeholders)?
- Is any user-supplied value concatenated directly into a SQL string?
- Are there any f-strings or format() calls building SQL?

Flag:
- Any string concatenation into SQL as CRITICAL
- Any f-string SQL as CRITICAL
- Even "safe" string operations should be flagged for review — the goal
  is zero string interpolation in SQL

**Note:** SQLite's Python library (sqlite3) uses ? placeholders. Any
deviation from this pattern is a red flag.

---

## Step 4 — XSS (Cross-Site Scripting) scan

Search for every place user-supplied content is rendered into HTML:
- Template strings that include user data without escaping
- innerHTML assignments with user data
- Any use of raw interpolation (${userValue}) in HTML generation
- HTML rendered from database fields without escaping

Flag:
- User-supplied text rendered to HTML without escaping (HIGH)
- Internal-staff-supplied text rendered without escaping (MEDIUM —
  lower risk but still exploitable via compromised accounts)
- Places where escaping is inconsistent (some fields escaped, others not)

---

## Step 5 — Credential and secret handling

Search the codebase for:
- Hardcoded passwords, tokens, or API keys anywhere in source files
- Credentials in comments
- Credentials in log output (check logging calls)
- Credentials in error messages
- Environment variables that should be in .env but might be hardcoded
- .env file committed to git (check .gitignore)
- Old credentials in git history (flag for BFG purge)

---

## Step 6 — Session security

Check:
- Cookie flags: httpOnly, SameSite, Secure (for HTTPS deployments)
- JWT: expiry set? Algorithm is HS256 or better (not 'none')?
- Session token entropy: is the session ID cryptographically random?
- Logout: does logout actually invalidate the session server-side,
  or just clear the client cookie?
- Session fixation: is the session regenerated on privilege escalation?
- Remember-me tokens: are they stored safely?

---

## Step 7 — File system security

Check every route that reads from or writes to the filesystem:
- Is any user-supplied value used in a file path without sanitization?
  (Path traversal: ../../etc/passwd)
- Does the route validate that the resolved path is within an allowed
  directory?
- Are file uploads (if any) validated for type and size?
- Does the chat-log API endpoint sanitize the username and date
  parameters before using them in glob patterns?

---

## Step 8 — Rate limiting and abuse potential

For every unauthenticated or lightly authenticated endpoint:
- Could it be called thousands of times per second without consequence?
- Does it have any rate limiting?
- Does it do expensive computation or database writes on each call?
- Could it be used to enumerate user accounts, file paths, or other
  sensitive information?

Flag:
- Unauthenticated endpoints that write to the database (CRITICAL)
- Login endpoints with no rate limiting (HIGH — brute force risk)
- Enumeration risks (endpoints that return different errors for
  "user exists" vs "wrong password")

---

## Step 9 — Cross-project auth

For any API call made from one project to another:
- Does the caller authenticate to the target?
- Is the credential stored safely (not hardcoded)?
- Does the target validate the credential on every request?
- Is there any inter-service communication that happens without auth
  because "it's internal"?

---

## Output format

Produce a report with these sections:

### SEVERITY RATINGS
CRITICAL: exploitable now, data at risk
HIGH: significant risk, fix before next release
MEDIUM: real issue but requires specific conditions or access
LOW: defense-in-depth improvement, no immediate risk
INFO: observation for awareness

### ROUTE AUTH INVENTORY
Complete table of all routes with auth level.

### FINDINGS (by severity)
Each finding: location, description, severity, what an attacker could do,
recommended fix.

### CREDENTIAL EXPOSURE SUMMARY
Any credentials found outside of environment variables.

### KNOWN ACCEPTED RISKS
Issues that are known and deliberately accepted with documented reasoning.

### IMMEDIATE ACTIONS (CRITICAL and HIGH only)
Short list for quick triage.
