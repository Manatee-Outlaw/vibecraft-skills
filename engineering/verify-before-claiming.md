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
  claim as-is. Applies equally to negative results: "clean", "no hits", "no
  errors found", "nothing to report", "no issues", "all tests passed" are
  claims that need the same proof as "fixed", because a check that aborted,
  matched nothing, or never ran produces output identical to a genuine pass.
  Trigger before reporting any clean scan, audit, grep, health check, or test
  result — prove the check could fail (exit code, positive control) before
  trusting that it didn't.
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
- **The check that cannot fail.** A scan that aborts, a grep with a rejected
  pattern, a test suite that never ran — all report "clean" identically to a
  real pass. Before trusting a negative result, prove the check was live:
  check the exit code, and feed it a positive control it must catch. See
  "The check that cannot fail" below.

## The check that cannot fail
A negative result is only meaningful if the check was capable of returning a
positive one. "No errors found" and "the error-finder didn't run" produce
identical output — and the second one is a lie that reads exactly like the
first.

Six instances of this pattern surfaced across two days, and the split
matters. In the product:
- An installer logged "seeded" without checking whether the seeding command
  had actually succeeded.
- A script wrote its own success flag without ever capturing the return code
  of the operation the flag was describing.
- A version marker advanced even when the command that should have set it
  failed silently.

In the tooling built to catch bugs like those:
- A fix for the first item raised a NON-TERMINATING parse error on corrupt
  input, so the corrupt case exited 0, wrote an empty value, and logged
  "seeded ()" — the fix reproduced the bug it was fixing. Only executing the
  malformed case caught it; code review passed it.
- A credential scan used a regex syntax the search tool rejected outright
  (rc=128). It reported "clean" while matching nothing at all — it could not
  have found the credentials it was written to find.
- A blocklist scan printed "0 hits" while the underlying search aborted on
  every single term.

An audit has the same shape when it asks the wrong question: it can confirm
that a write-only telemetry field is HARMLESS (nothing reads it, so nulls
can't break anything) while never asking whether anything is ARRIVING in it.
Both answers are "fine." Only one of them is true. Live-signal checks are
only as good as the question asked of them.

**Prove the check can fail before trusting that it didn't.** Every time a
scan, a grep, a validator, a health check, or a test suite returns a clean
result, ask:
1. What is the exit code? Not the output — the exit code. A tool that aborts
   often prints nothing and returns non-zero, which a loop will happily
   discard.
2. Did I feed it a POSITIVE CONTROL — something it MUST catch? If a scan for
   secrets returns zero hits, plant a known string and confirm it's found.
   If a test suite passes, break something and confirm it goes red. A suite
   that passes on broken code is worse than no suite.
3. Am I checking that something is HARMLESS when I should be checking that
   it WORKS? Those are different questions with the same reassuring answer.
4. Did the thing I'm reporting on actually EXECUTE, or did I only observe
   that nothing complained? Silence is not success.

## Applied to writing code, not just running checks
The same discipline applies when writing the code itself: any operation whose
failure is discarded (`>nul 2>&1`, a bare `except:`, an unchecked return
code, a non-terminating error mode) must not be followed by an unconditional
success log or flag. Either capture the result and branch on it, or don't
claim success.

"Write the marker, log success, discard the evidence" is a habit, not an
accident. It has appeared six times. Assume it is present wherever a status
is recorded and the operation that produced it is not checked.

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
- Before reporting ANY clean/negative result from a scan, audit, or test —
  "found nothing" is a claim like any other and needs the same proof as
  "fixed"
