---
name: verify-before-versioning
description: >
  Before editing or publishing any skill or bundle, work from its CURRENT state — never
  a remembered or older local copy — and never create a new version-suffixed copy of a
  repo file. Public skills live in the git repo and are edited IN PLACE (git history is
  the version log); private skills still live in Google Drive, where the -vN.N convention
  still applies. Also run the privacy triage before publishing anything to the public
  repo. Trigger automatically, every time, before any skill or bundle file is created,
  edited, or published — no trigger phrase needed.
---

# Verify Before Versioning

## Why this skill exists

Two separate sessions, six days apart, each independently created a new file named
"engineering-v1.5.md" — identical filenames, completely different content, neither aware
the other existed. Both sat live in the same Google Drive folder for six days, silently
conflicting, until a routine scan happened to notice two files with the same name and
different modification dates.

That was a symptom of a deeper problem: **Drive cannot edit a file in place, so every
update had to become a NEW file** — and new files collide, drift, and break any link that
pointed at the old name. The skill library now lives in a git repo, where files ARE
edited in place and history is kept automatically. That removes the root cause for public
skills — but the underlying discipline (work from the current state, don't fork versions,
don't break references, don't leak) still matters, and Drive-based PRIVATE skills still
have the old constraint.

## The core rules

**1. Public-repo skills are edited IN PLACE. Never version-suffix them.**
Every skill and bundle in the public repo is edited directly. Git history IS the version
log — run `git log --follow <path>` to see every change. NEVER create a new `-vN.N` copy
of a repo file, and never add a version suffix to a filename. A renamed file changes its
raw URL, which breaks every project instruction and cross-reference pointing at it — the
exact bug the repo exists to kill.

**2. Before editing, read the CURRENT file from the repo — not a memory of it.**
Fetch/read the live file before changing it. Never edit from a remembered copy, an older
local checkout, or what you think the file said last session. Confirm the real current
content first, then edit in place.

**3. PRIVATE skills still live in Google Drive — the -vN.N convention STILL APPLIES there.**
Drive still cannot edit in place, so a private skill is still updated by creating a new
highest-versioned file. Before creating a new version of a private skill, do a fresh,
live search of the actual Drive folder RIGHT NOW for the current highest version — never
trust a remembered version number. Read that highest version and merge deliberately;
don't fork a parallel copy. (This is the original discipline, unchanged, for the one
place it still applies.)

**4. Before publishing ANY new or edited skill to the public repo, run the privacy triage.**
The repo is PUBLIC and publishing is irreversible (indexed, mirrored, cloned, and scraped
within hours; deleting later recalls nothing). Before a file goes into — or gets updated
in — the repo, test it against all three:
- **TEST 1 — CREDENTIALS:** tokens, API keys, passwords, .env contents, signing keys,
  private URLs with embedded auth, session IDs, connection strings.
- **TEST 2 — BUSINESS INTERNALS:** roster names/handles, revenue, pricing, bonus
  mechanics, vetting criteria, Google Drive folder IDs, server IPs or filesystem paths,
  private repo names, database/table specifics, any business-tied internal identifier.
- **TEST 3 — PERSONAL:** the board-of-directors profile, health, finances, family,
  relationships, real names — anything about the person rather than the work.

ANY single yes ⇒ the file stays PRIVATE in Drive, not in the repo. DEFAULT TO PRIVATE when
uncertain. Re-run this on EVERY edit, not just at creation — a later edit can introduce
something that wasn't there before. State explicitly that the check was done.

## What to do, every time

1. **Editing a public skill/bundle:** read the current repo file first; edit it in place;
   commit (history records the change). Do not rename it, do not add a suffix.
2. **Creating a NEW public skill:** confirm no file of that name already exists in the
   repo; add it with a clean, suffix-free name; run the privacy triage before committing.
3. **Updating a PRIVATE (Drive) skill:** live-search Drive for the current highest
   version; read it; save the next version; never assume the version number from memory.
4. **Publishing anything:** run the three-test triage; default to private on any doubt;
   say the check was done.

## Relationship to other skills

- **skill-library-audit** is the periodic sweep that catches drift, staleness, and broken
  references after the fact. verify-before-versioning is the in-the-moment discipline
  applied the instant before a write or a publish.
- **verify-before-claiming** requires execution proof before claiming something works;
  this skill is about working from the real current state and not leaking on publish.

## Trigger this skill

- Automatically, every time — before creating, editing, or publishing any skill or bundle
  file, in the repo or in Drive.
- No trigger phrase needed; always-on, the same way verify-before-claiming is.
