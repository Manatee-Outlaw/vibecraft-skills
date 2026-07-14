---
name: verify-before-versioning
description: >
  Before creating a new skill file, or creating a new version of an
  existing skill or bundle, always do a fresh, live search of the actual
  Drive folder right now — never rely on conversational memory of what
  version was last known, no matter how recent. Checks whether the target
  filename already exists, what the actual current highest version is, and
  whether anything changed recently that this write would conflict with or
  fail to account for. Trigger automatically, every single time, before any
  skill or bundle file gets created or updated — no trigger phrase needed.
---

# Verify Before Versioning

## Why this skill exists

Two separate sessions, six days apart (July 6 and July 12, 2026), each
independently updated the engineering skill bundle and each named their
new file "engineering-v1.5.md" — identical filenames, completely different
content, neither aware the other existed. One added a real safety
discipline (propagate-the-fix). The other added a real efficiency upgrade
(concurrent subagent audits). Both sat live in the same Drive folder for
six days, silently conflicting, until a routine cleanup scan happened to
notice two files with the same name and different modification dates.

Neither session did anything wrong in isolation — each one correctly
looked at what it believed was the current state (from its own
conversation history) and versioned forward from there. The actual gap was
narrower and more specific: neither session did a fresh, live check of the
real Drive folder in the moment right before writing. Both trusted memory
of "the last version I know about" instead of confirming "the actual
current version, right now, as it exists on disk."

This is the exact same shape of gap propagate-the-fix was built for in the
codebase — a fix or an update happens correctly in isolation, but nobody
checks whether a sibling, parallel version of the same thing already
exists elsewhere. Here, the "codebase" is the skill library itself, and
the risk is worse in one specific way: an outdated or incomplete version of
a skill can get loaded silently in a future session with no error, no
warning — it just quietly does less than it should, or contradicts a
newer version nobody knew to look for.

## The core rule

**Never write a new skill file, or a new version of an existing skill or
bundle, without first doing a live search of the actual current Drive
folder — not a memory of it, not what was true earlier in this
conversation, the real current state, checked right now.**

## What to do, every time, before writing

1. **Before creating a brand-new skill file:** search the target folder
   for that exact filename (and close variants — different capitalization,
   a trailing version number) to confirm nothing with that name already
   exists. A collision here means either genuine duplicate work in
   progress, or a naming choice that needs to change.
2. **Before creating a new version of an existing skill or bundle:**
   search for every file matching that skill/bundle's name pattern (e.g.
   "engineering-v*") and find the actual highest version currently
   present — don't assume it matches what this conversation last touched.
   If a higher version exists than expected, read it before writing
   anything — someone else's work is already there, and it needs to be
   understood and merged, not silently overwritten or duplicated.
3. **If an unexpected file is found:** don't proceed as if nothing
   happened. Read what changed, understand why, and merge deliberately —
   the same way a v1.6 was built to combine two competing v1.5s after the
   fact. Doing this at write-time is strictly better than doing it as
   cleanup later, because a later cleanup only happens if someone thinks
   to look — and in the meantime, two live, conflicting versions sit in
   production with no way to know which one any given session actually
   loaded.
4. **State the check explicitly when reporting the write.** "Searched for
   existing versions of [skill/bundle] before writing — found none" or
   "found version X, read and merged its changes into this version." Make
   the check a visible, verifiable step, not an assumed one.

## Scope

This applies to any skill or bundle file, in any topic area — engineering,
creative, enterprise, productivity, or any future bundle — not just this
project's engineering skills specifically. The failure mode (two
independent writers, no live check, silent version collision) is generic
to the whole skill-library system, not particular to any one bundle's
subject matter.

## Relationship to skill-library-audit

skill-library-audit is the periodic, scheduled sweep — it catches overlaps,
staleness, and gaps on a schedule, after the fact. verify-before-versioning
is the in-the-moment discipline, applied the instant before a write
happens — the same relationship propagate-the-fix has to ponytail-audit in
the engineering skill set. Both matter; this one exists specifically
because six days of silent, undetected conflict is six days too long for
something that a 30-second live search would have caught immediately.

## Trigger this skill

- Automatically, every time — before creating any new skill file
- Automatically, every time — before creating any new version of an
  existing skill or bundle file
- No trigger phrase needed; this is always-on the same way
  verify-before-claiming and propagate-the-fix are always-on
