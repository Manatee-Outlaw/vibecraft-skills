---
name: ponytail-audit
description: >
  Run the real ponytail over-engineering audit against the codebase to find
  structural cleanup opportunities: reinvented standard-library code, unneeded
  dependencies, speculative abstractions, duplicated logic that's drifted apart
  across files, and dead or superseded files nobody deleted. Use periodically as
  part of a comprehensive audit, or whenever the codebase has grown significantly
  since the last run. Trigger phrases: "ponytail audit", "run ponytail",
  "code hygiene audit", "audit for over-engineering", "what can I delete".
---

# Ponytail Audit

## What this is

`ponytail` is the Anthropic-authored over-engineering skill family that ships
with Claude Code — NOT a third-party tool. In this environment it is available
as real, registered skills you invoke directly:

- **ponytail:ponytail-audit** — whole-repo scan for over-engineering: a ranked
  list of what to delete, simplify, or replace with a stdlib/native equivalent.
  This is the one this skill drives.
- **ponytail:ponytail-review** — the same lens focused on a diff rather than the
  whole repo (use during review of a specific change).
- **ponytail:ponytail-debt** — harvests every `# ponytail:` marker comment in the
  codebase into a debt ledger, so the deliberate shortcuts ponytail left behind
  get tracked instead of rotting into "later means never".

It finds a different category of problem than a security audit or a bug hunt:
not "is this broken," but "has this accumulated mess — a reinvented helper, a
dependency doing what five lines would, an abstraction with one caller, a dead
file beside its thriving replacement — that will cause a bug or a maintenance
tax later."

## When to use

- As part of a comprehensive audit, alongside the other engineering skills
- Whenever the codebase has grown significantly since the last run (many
  commits, several new features) and hasn't had a structural cleanup pass
- When something feels harder to maintain than it should, and duplicated or
  drifted code is suspected as the reason

## How to run it

Invoke the real skill directly — say "run a ponytail audit" (or `/ponytail-audit`),
which triggers **ponytail:ponytail-audit** against the repository. It is an
existing capability, not something Claude Code needs to be taught; your job is to
run it and capture the full output. If you are generating a task.md for Claude
Code, the instruction is simply:

```
Run the ponytail:ponytail-audit skill against this repository. Report every
finding it produces, in full — do not summarize or filter the list before
showing it back.
```

Do NOT describe ponytail as an external binary or a URL to fetch; it is a skill
in this session. (An earlier version of this file wrongly described it as a
third-party tool at a made-up GitHub URL — that provenance was fictitious.)

## How to triage the findings

Ponytail findings are structural cleanup opportunities, not urgent bugs. Land
every finding on exactly one of three outcomes — never leave one unaddressed
with no reasoning either way:

- **RESOLVED** — fix it now if it's small and safe (consolidate a helper that
  drifted into several inline copies; replace a duplicated definition with a
  single import; delete a genuinely dead, fully-superseded file). Per
  verify-before-claiming, prove the fix didn't change behavior for any call site
  that depended on a quirk of the pre-fix duplication — a consolidation that
  silently breaks one caller is worse than the mess it replaced.
- **KEPT AS-IS, WITH REASONING** — a finding that looks like a problem but is
  deliberate (e.g. audit documents flagged as redundant that are intentionally
  kept as historical record). Decline it with the reasoning stated, not silently.
- **DEFERRED** — a genuine finding not worth the complexity or risk to fix now
  (e.g. near-identical builders that are cheaper left separate). Defer with a
  stated trigger condition to revisit ("revisit if a fourth similar builder is
  added").

## Trigger this skill

- "ponytail audit", "run ponytail", "code hygiene audit", "audit for over-engineering"
- As part of a comprehensive audit run, when a full structural pass is warranted
  in addition to the standard checks
- After a long stretch of active feature development, before the codebase
  accumulates too much drift to clean up cheaply
