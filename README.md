# Retired 2026-10-03 — the skills moved to skill-stack

This repo is no longer a skill source. Every skill that lived here was merged into
**[Manatee-Outlaw/skill-stack](https://github.com/Manatee-Outlaw/skill-stack)** (commit `fd926d4`),
which is now the one record of truth. The files were removed rather than left in place on purpose:
a stale copy that keeps answering is worse than a 404 that says where to go.

Old URL → new URL:

    https://raw.githubusercontent.com/Manatee-Outlaw/vibecraft-skills/main/engineering/<name>.md
    https://raw.githubusercontent.com/Manatee-Outlaw/skill-stack/main/plugins/<plugin>/skills/<name>/SKILL.md

`<plugin>` is `skill-engineering` for the audits, `skill-core` for the always-on disciplines
(verify-before-claiming, trust-the-live-signal, ...). The history of every file is still in this
repo's git log.

Not carried, deliberately: `session-cold-start` (retired in skill-stack `5b492c4`), the three
creative skills (Anthropic's official versions are used instead; see skill-stack `EXTERNALS.md`),
and `bundles/` (the retired load-list system).
