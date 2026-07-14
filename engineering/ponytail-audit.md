---
name: ponytail-audit
description: >
  Run the /ponytail static-analysis tool (github.com/DietrichGebert/ponytail)
  against the codebase to find structural cleanup opportunities: duplicated
  logic that's drifted apart across files, dead or superseded files nobody
  deleted, and configuration inconsistencies. Use periodically as part of a
  comprehensive audit, or whenever the codebase has grown significantly
  since the last run. Trigger phrases: "ponytail audit", "run ponytail",
  "code hygiene audit".
---

# Ponytail Audit

## What this is

Ponytail (github.com/DietrichGebert/ponytail) is a third-party static
analysis tool, available in this environment as the Claude Code slash
command /ponytail. It scans a codebase and surfaces structural findings —
duplicated logic that's drifted apart, dead or superseded files, and
configuration drift. It finds a different category of problem than a
security audit or a bug hunt: not "is this broken," but "has this
accumulated mess that will cause a bug later."

This was previously run once against this codebase, on June 21, 2026,
directly as a one-off tool invocation — the run itself and its findings
are real and documented, but it had never been formalized as a repeatable
skill until now.

## When to use

- As part of a comprehensive audit, alongside the other engineering skills
- Whenever the codebase has grown significantly since the last run (many
  commits, several new features) and hasn't had a structural cleanup pass
- When something feels harder to maintain than it should, and duplicated
  or drifted code is suspected as the reason

## How to run it

Generate a task.md for Claude Code with this instruction:

```
Run the /ponytail slash command against this repository. Report every
finding it produces, in full — do not summarize or filter the list before
showing it back.
```

/ponytail is an existing Claude Code capability, not something Claude Code
needs to be taught how to do — the task.md's only job is to trigger it and
capture the output faithfully.

## How to triage the findings

Ponytail findings are typically structural cleanup opportunities, not
urgent bugs. For each finding, land on one of three outcomes — the June
21, 2026 run against this exact codebase is the model for how this
triage should look:

- **RESOLVED** — fix it now if it's small and safe. Examples from the
  June 21 run: consolidating a helper function that had drifted into 14+3
  separate inline copies across two files into one shared version;
  replacing a duplicated function definition with a single import alias;
  deleting a genuinely dead deploy script that had been fully superseded.
  Per verify-before-claiming, prove the fix didn't change behavior for any
  call site that depended on a quirk of the pre-fix duplication — a
  consolidation that silently breaks one caller is worse than the mess it
  replaced.
- **KEPT AS-IS, WITH REASONING** — sometimes a finding that looks like a
  problem is actually deliberate. Example from June 21: ponytail flagged
  some audit documents as redundant; they were intentionally kept as a
  historical record and the finding was declined with that reasoning
  stated, not silently ignored.
- **DEFERRED** — a genuine finding, not worth the complexity or risk to
  fix immediately. Example from June 21: three near-identical report
  builders were flagged as shareable, but each had different enough
  layouts that unifying them would add complexity without enough payoff —
  deferred with a stated trigger condition ("revisit if a fourth similar
  builder is ever added").

Every finding gets one of these three outcomes explicitly stated — never
leave a finding unaddressed with no reasoning given either way.

## Trigger this skill

- "ponytail audit", "run ponytail", "code hygiene audit"
- As part of a comprehensive audit run, when a full structural pass is
  warranted in addition to the standard checks
- After a long stretch of active feature development, before the codebase
  accumulates too much drift to clean up cheaply
