---
name: verify-before-claiming
description: >
  Require real execution evidence before reporting anything as fixed, working,
  deployed, or verified. Use whenever a report is about to claim something
  works, is live, or is already resolved — the claim must be backed by an
  actual run, not just a read of the code. Trigger automatically before
  writing "fixed", "deployed", "verified", "already resolved", "works
  correctly", or similar language in any report. Also trigger when a prior
  report or an earlier session claims something is already true and there is
  no logged execution proof attached — re-verify rather than repeating the
  claim as-is.
---

# Verify Before Claiming

## Why this skill exists
On June 30, 2026 (Build 1.6 rollout), two things happened on the same night:
a code review reported a bug as "already fixed" — it wasn't; the code that
read correctly had a hidden character-parsing bug (stray parentheses read as
end-of-block by cmd.exe) that only broke when actually run. Separately, a
build was reported "pushed, deployed, active" before it had genuinely reached
the production server. Both cost real troubleshooting time on a live streamer
account, at the worst possible moment — hours after a mandatory rollout.
Reading code well is not the same as knowing what the code does when it runs.

## The core rule
**Code review tells you what should happen. Only running it tells you what
does happen.** Never write "fixed," "working," "deployed," or "verified" in
a report unless there is an actual execution trail behind that word.

## Before writing any status claim, ask
1. Did I actually run this, or did I read it and conclude it should work?
2. If I ran it — what exact command, what exact output? Could I paste it?
3. If I could NOT run it (no safe test environment, would affect a live
   user, no way to replicate the exact machine/condition) — have I said so
   explicitly, in the same sentence as the claim, not buried later?
4. Am I about to repeat a claim from an earlier report or session without
   re-checking it myself right now? A prior "verified" is not proof — verify
   again if it materially matters to what's being decided.

## Required report format
Every fix/deploy/audit report must separate these two categories explicitly:
- **Proven by execution** — what was actually run, and the actual result
  (paste the real output where possible, not a paraphrase of it)
- **Believed correct by code review only** — what looks right but was not
  executed, and why (no safe test environment, would disrupt a live user,
  environment couldn't be replicated, etc.)

Never blend these into one undifferentiated "done." A person reading the
report needs to know at a glance which claims to fully trust and which are
still open.

## Specific traps to watch for
- **The "should work" report.** Code that reads correctly can still fail at
  runtime for reasons invisible on a read-through — a stray character a
  command interpreter misreads, a wrong working directory, a silently-caught
  exception, a dependency that resolves differently at runtime than assumed.
  If it hasn't run, it hasn't been proven — no matter how confident the read
  felt.
- **The stale "already fixed" claim.** A previous commit or an earlier
  session's report claiming something is resolved is a starting hypothesis,
  not a fact. Confirm it against the actual current state (the live server,
  the live file, a real run) before repeating it in a new report.
- **The "deployed" claim without checking the actual target.** "Pushed"
  means it left the local machine. "Deployed" means it is genuinely running
  on the real target right now. Confirm the live target's actual state (its
  real git HEAD, its actual running process) — don't infer deployment from
  a successful local push or an earlier assumption.
- **The happy-path-only test.** Running the success case once is not the
  same as testing the failure case, the edge case, or the specific scenario
  a real user actually hit. If a real user reported a specific symptom,
  reproduce that exact symptom as closely as possible — don't settle for a
  nearby, easier-to-run scenario and call it equivalent.
- **Skipping the branch that's inconvenient to test.** If a code path is
  hard to test (needs a machine with no Python, needs a fresh clean install,
  needs a live streaming session), that difficulty is not a reason to skip
  it — it's usually a sign that path is exactly the one that's never been
  exercised for real and most likely to hide a bug. Find a safe way to
  simulate it rather than skipping it.

## When execution genuinely isn't possible
Sometimes real execution isn't available — no safe way to test on a live
production system, no second machine to replicate a real user's exact
environment. That's a legitimate limitation, not a failure. But say so
plainly, in plain language, right next to the claim: "not verified by
execution — reasoned from code review only, because [specific reason]."
Never let an unverified claim read the same as a verified one in a report.

## Trigger this skill
- Before finalizing any report that uses the words "fixed," "resolved,"
  "deployed," "working," "verified," or "complete"
- Whenever repeating a claim made in an earlier session or an earlier task
  report, rather than treating it as already-settled
- After any holistic-code-audit, engineering-review, or comprehensive-audit
  finding is marked resolved — confirm the fix was actually run, not just
  written and read back
- Immediately after a live production incident, before writing the
  after-action summary — this is exactly when the temptation to round up
  "should be fine now" to "fixed" is highest
