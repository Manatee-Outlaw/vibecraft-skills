---
name: render-smoke
description: >
  Drives a real browser against seeded STAGING accounts (one per role), renders every
  view with real data, and asserts the RENDERED DOM is free of the universal broken-render
  signatures (undefined / NaN / $0 / undefinedh / null% / [object Object] / a date sliced to
  one digit) — the class that static contract checks and code review structurally cannot see,
  because it only exists in pixels a logged-in user looks at. Optionally clicks
  success-claiming controls and asserts a mutating request actually fired (kills fake-success).
  Runs as one dedicated subagent in a comprehensive audit.
---

# render-smoke

**Why this exists.** contract_check.py proves the server *could* send a key; live_smoke.py
proves it *is* sent; neither proves the screen is right. The pre-flip audit (2026-07-27)
verified contracts and code but never drove the authenticated UI, and missed four live-only
bugs — coaching reports rendering "$0 / undefinedh", a trends axis showing "3" instead of
"Jul 3", a "Download" that printed the page, a dead hover tooltip. Every one only existed on
a rendered page. This skill closes that gap.

## Prerequisites (report as an audit COVERAGE GAP if missing — do not silently pass)

- A reachable **staging** URL (`SMOKE_BASE`) running the same code against a separate DB —
  see `deploy/staging-setup.md`. Never run the interaction layer against production.
- **Seeded** accounts, one per role (`deploy/seed_test_accounts.py`). An empty state is
  legitimate and HIDES the drift this catches — unseeded staging is not a pass, it is no data.
- `playwright` + chromium where the runner executes.

If staging isn't up or isn't seeded, the honest finding is "render-smoke could not run: no
seeded staging" — a gap in coverage, not ALL CLEAR.

## Steps

1. Confirm staging is reachable and seeded (`curl -sI $SMOKE_BASE/app/auth.html` → 200; the
   seed accounts exist). If not, stop and report the coverage gap.
2. Run the engine:
   ```bash
   export SMOKE_BASE=<staging url>; export SEED_PASSWORD=<seed pw>
   python3 consolidation/render_smoke.py
   ```
   It logs in per role, clicks through every nav view, and scans each view's innerText for the
   signature list. Exit non-zero + `RENDER-SMOKE [role/view] signature '…': …snippet…` lines
   are your raw findings.
3. For the destructive **interaction-backing** layer (fake-success detection — the owner-delete
   / change-password class), populate the `INTERACTIONS` list in `render_smoke.py` with
   `(role, label, selector, must-hit-endpoint)` tuples and re-run **against staging only**. An
   unbacked success ("clicked but NO /remove write fired") is a HIGH finding.
4. Triage each signature to what the USER receives (a manager reading "+0%" for every creator;
   a streamer seeing "$0 / undefinedh"): rate by impact, not "the page didn't crash".

## Governing standards (as for every audit subagent)

- **verify-before-claiming**: the runner IS the execution proof — cite the exact
  `RENDER-SMOKE …` line. An ALL CLEAR must show the run actually happened and could have come
  back dirty (staging was seeded, login succeeded, N views scanned) — a run that skipped
  (no playwright / no SMOKE_BASE / empty staging) reports SKIP, never CLEAN.
- **trust-the-live-signal**: the rendered pixel is the signal; a green contract_check is not.
- **close-known-gaps**: if a signature appears in one view, grep the codebase for the sibling
  render pattern and check every view, not just the one the browser happened to catch.
- **user-impact rating**: "$0"/"undefined" on a data-bearing view is never LOW.

## Output (audit format)

```
[render-smoke] FINDINGS:
HIGH: manager roster shows "+0%" momentum for every creator | FILE: static/app/manager.html | FIX: server payload drift
...
ALL CLEAR: <views scanned per role, and that staging was seeded + login worked so it could have failed>
```

## Project-agnostic core

The signature list in `render_smoke.py` (`undefined`, `NaN`, `$NaN`, `[object Object]`,
`undefinedh`, `null%`, `Invalid Date`, …) and the interaction-backing rule ("a control that
toasts success must be backed by a resolved network write") are universal — only `SMOKE_BASE`,
the seed accounts, and the `INTERACTIONS` tuples are per-project.
