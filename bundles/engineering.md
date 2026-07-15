# Bundle: Engineering Project
# Versioning: this file is edited IN PLACE. Git history is the version log —
# run `git log --follow bundles/engineering.md` to see every change. Do NOT add a
# -vN.N suffix to any file in this repo; that convention existed only because Google
# Drive could not edit files in place, and suffixed filenames break the raw URLs that
# project instructions point at.
# Last substantive change: 2026-07-15 — added engineering/task-authoring.md to the
# always-on governing standards (assumptions block + goal-not-mechanism), after two
# task-file instructions specified mechanisms in domains the author could not see and
# would have caused real damage if followed literally.

Load these skills at the start of any software/coding project conversation.

## Skills to load (in order)

1. engineering/grill-with-docs.md
2. engineering/diagnose.md
3. engineering/to-prd.md
4. engineering/improve-codebase-architecture.md
5. engineering/flow-test.md
6. engineering/holistic-code-audit.md
7. engineering/verify-before-claiming.md
8. engineering/task-authoring.md
9. engineering/verify-before-versioning.md
10. engineering/trust-the-live-signal.md
11. engineering/no-assumed-memory.md
12. engineering/propagate-the-fix.md
13. engineering/propagate-the-lesson.md
14. engineering/close-known-gaps.md
15. engineering/no-dead-ends.md
16. engineering/hostile-environment-testing.md
17. engineering/anticipate-user-mistakes.md
18. engineering/ponytail-audit.md
19. engineering/push-to-git.md
20. engineering/engineering-review.md
21. engineering/architecture-review.md
22. engineering/database-hygiene.md
23. engineering/ux-review.md
24. engineering/mobile-ux.md
25. engineering/cross-project-audit.md
26. engineering/external-integration-audit.md
27. engineering/security-audit.md
28. engineering/production-drift.md
29. engineering/skill-library-audit.md
30. engineering/session-cold-start.md
31. engineering/version-management.md
32. engineering/comprehensive-audit.md
33. engineering/discord-announcements.md
34. engineering/hermes-upload.md
35. productivity/board-of-directors.md

(copy-review and database-review are intentionally absent — see the PRIVATE section below.)

## What each skill does (plain English)

- **grill-with-docs** — Before building anything, interview the user to nail down exactly what they want and why.
- **diagnose** — When something is broken, follow a disciplined 6-step process to find and fix it properly rather than guessing.
- **to-prd** — Turn a conversation into a formal written spec so there is a clear record of what was agreed before work starts.
- **improve-codebase-architecture** — Periodically survey the codebase for tangles and structural problems before they become expensive bugs.
- **flow-test** — Walk through the product as a real user would, checking every journey works end-to-end. Now the canonical home for the "integration seam check" technique (data-handoff tracing), also referenced by holistic-code-audit rather than duplicated there. Phase 3 requires opening a real, recent output artifact (report, PDF, email, export) and reading it as the recipient — "it generated without an error" is not "it is correct".
- **holistic-code-audit** — UPDATED to v1.1. The 8-discipline code audit (runs in chat). Discipline 6 now points to flow-test's Phase 0 for the integration seam check instead of repeating it. Use engineering-review for deeper Claude Code audits.
- **verify-before-claiming** — Requires real execution proof before any report says "fixed," "working," "deployed," or "verified" — and the same proof before any report says "clean," "no hits," or "all tests passed," because a scan that aborted or never ran reports identically to a real pass. Prove a check can fail before trusting that it didn't. Always-on.
- **task-authoring** — Governs how task files are written for an implementer. Every task.md opens with an ASSUMPTIONS block (each factual dependency, its source, verified or not); in any domain the author cannot see — code, server, database, target OS — state the GOAL, never the mechanism. The implementer enforces it: no assumptions block, or a mechanism the author couldn't have verified, gets called out and the goal followed instead. Always-on.
- **verify-before-versioning** — Before creating any new skill file, or a new version of an existing skill/bundle, always do a fresh, live search of the actual Drive folder right now. Always-on.
- **trust-the-live-signal** — When a stored value, documentation, or a surface-level observation disagrees with a live, directly-checkable signal, trust the live signal. Always-on.
- **no-assumed-memory** — Never assume a fresh session shares memory with a different conversation. Always-on whenever work crosses a session boundary.
- **propagate-the-fix** — Whenever a bug is fixed, actively search the codebase for the same underlying pattern elsewhere. Always-on.
- **propagate-the-lesson** — Whenever a genuinely general-purpose lesson or fix is identified, check every bundle for the same applicability, and check comprehensive-audit's own registration lists. Always-on.
- **close-known-gaps** — Once a real, known issue has been surfaced, the default is to fix it now rather than narrowing scope. Always-on.
- **no-dead-ends** — Whenever something can enter a "not working" state, prove there's a real path back to working. Always-on for error states, reconnect cycles, and user-triggered stops.
- **hostile-environment-testing** — Before trusting any installer, launcher, or scheduled script, test it against messy real-world conditions.
- **anticipate-user-mistakes** — For every user-facing control, imagine the most natural mistake and check the real consequence.
- **ponytail-audit** — Runs the /ponytail static-analysis tool for structural cleanup opportunities.
- **push-to-git** — Full deploy in one command. Trigger: "push to git [message]", "deploy", "ship it".
- **engineering-review** — Full 8-check bug hunt via Claude Code. Trigger: "engineering review", "code audit".
- **architecture-review** — Deep structural analysis. Trigger: "architecture review".
- **database-hygiene** — Design principles for SQLite tables and INSERT logic, plus the arrival audit: whether data is actually landing in a column, not just whether its absence can break anything. Empty or uniform is a finding; write-only columns have no consumer to notice they broke.
- **ux-review** — Heuristic evaluation of the interface. Trigger: "UX review".
- **mobile-ux** — Reviews the interface for mobile use. Trigger: "mobile review".
- **cross-project-audit** — Audits how multiple Claude project chats interact. Trigger: "cross-project audit".
- **external-integration-audit** — Audits implementation against vendor documentation. Mandatory Step 0: confirm real, current docs were used. Trigger: "integration audit".
- **security-audit** — Focused security review. Trigger: "security audit".
- **production-drift** — Compares GitHub vs live production server. Trigger: "production drift".
- **skill-library-audit** — Audits the skill library itself. Trigger: "audit our skills".
- **session-cold-start** — Loads all context to start a working session. Trigger: "start the session", "cold start".
- **version-management** — Checklist for bumping version constants. Trigger: "bump the version".
- **comprehensive-audit** — Runs the full engineering audit via concurrent subagents. All governing standards are baked into the skill itself and apply automatically. Trigger: "comprehensive audit", "full audit", "audit everything."
- **discord-announcements** — Drafts Discord announcements. Trigger: "draft a Discord announcement".
- **hermes-upload** — Generates a handoff document and uploads to Drive. Trigger: "hermes upload".
- **board-of-directors** — Convenes your personal Board of Directors. Trigger: "convene my board".

## PRIVATE — not in this repo; load from Google Drive

These load from Google Drive, not this repo. **Drive files are still version-suffixed**
(Drive cannot edit in place) — always search Drive for the highest-numbered version
rather than assuming a filename. The private set is exactly:

- **copy-review** — Reviews the product's own interface copy against its specific coaching voice. Trigger: "copy review".
- **database-review** — Dedicated operational database audit against the live production database. Trigger: "database review".
- **board-of-directors profile** — the personal board profile (`board-of-directors-default-profile-*.md`) that the public `board-of-directors` skill loads. The SKILL is in this repo; the PROFILE is private and lives only in Drive.

## When to run each review

- engineering-review: before any significant release, after a bug-heavy session
- architecture-review: quarterly, or before a major new feature
- ux-review + mobile-ux: before any public-facing release
- flow-test: before releasing any new feature — also run alongside holistic-code-audit discipline 6 for the full integration-seam treatment
- production-drift: after every major sprint, before VPS migration
- skill-library-audit: after every major sprint, or when a production failure reveals a skill gap
- session-cold-start: at the beginning of every session
- hermes-upload: at the end of every session
- verify-before-claiming, task-authoring, verify-before-versioning, trust-the-live-signal, no-assumed-memory, propagate-the-fix, propagate-the-lesson, close-known-gaps, no-dead-ends: always-on, every time, no exceptions.
- hostile-environment-testing: before shipping any installer/launcher/scheduled-script change.
- anticipate-user-mistakes: before shipping any new user-facing control, and inside every comprehensive audit.
- ponytail-audit: periodically as part of a comprehensive audit, or after a long stretch of active feature development.

## Comprehensive audit — self-contained, no manual steps needed

Say "comprehensive audit" and the comprehensive-audit skill handles everything —
10 concurrent subagents, all 8 governing standards applied automatically to
every finding. Nothing needs to be manually re-specified.

## Note on this update — the skill-library read-through

A full, deliberate read of every file in this skill library (not just
filenames/versions) was done to check for genuine content overlap, not
just naming collisions. One real finding: holistic-code-audit's
integration-seam check and flow-test's Phase 0 had independently grown to
teach nearly the same technique, in nearly the same words, in two
unconnected files. Fixed by trimming holistic-code-audit and pointing it
at flow-test as the canonical source. Everything else checked (roughly 25
files read in full) was confirmed to be deliberately, explicitly
separated with a stated reason — no other redundancy found in the
engineering skill set specifically.
