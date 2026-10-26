---
name: debug
description: Disciplined debugging - reproduce, gather evidence, find the root cause, then make a minimal fix with a regression test and explain it. Use this whenever the user reports a bug, error, exception, failing test, crash, wrong data, something "not working", "broken", "weird behaviour", a stack trace or an error message, even if they just paste the error without asking anything.
---

# Debug coach

The common failure mode is patching the symptom: wrapping something in try/catch, adding a null
check, or changing a test until it passes. That makes the error disappear while the real bug stays,
often somewhere harder to find. This workflow forces finding the cause first.

Explain your reasoning as you go in plain language - the user should learn how the bug was found,
not just get a fix.

## 1. Define the problem

Get these, asking only for what's missing:
- What exactly happens (error message, stack trace, wrong output, screenshot)?
- What was expected?
- Where: local / staging / prod? Which page, endpoint, user, data?
- Since when? Did it ever work? What changed recently (`git log` for the area helps)?
- Always, or only sometimes?

## 2. Reproduce

Find the smallest reliable way to trigger it: a failing test, a curl request, or exact UI steps.
If you can't reproduce, say so and gather more evidence before changing code - a fix you can't
verify is a guess.

Where to look for evidence: see **Logs & debugging** and **Known traps** in `CLAUDE.md`.
TODO(team): add any extra team-wide places (Sentry, Kibana, shared log dashboards).

## 3. Locate and hypothesize

- Follow the stack trace / data flow from where the error surfaces back to where the bad value
  originates.
- Write down 1-3 hypotheses, most likely first, and what evidence would confirm or rule out each.
- Test them with evidence: read the code, add temporary logs, inspect DB data, run with a debugger.
- Check **Known traps** in `CLAUDE.md` early - many bugs are a known gotcha.

## 4. Confirm the root cause

State it in one or two sentences: "The bug happens because X, which causes Y when Z."
If you can't state it this precisely yet, you haven't found it - keep investigating.

Share the root cause with the user before fixing if the fix is non-trivial or touches
**Do not touch without asking** areas.

## 5. Fix

- Fix the cause, at the place it originates, with the smallest sensible change.
- Don't swallow errors, don't add defensive checks that hide the problem, don't weaken or skip
  tests to get green.
- If the same mistake likely exists elsewhere, search for it and mention it (fix only if the user agrees).
- Remove temporary debug logs.

## 6. Prove it

- Add a regression test that fails without the fix and passes with it (see `write-tests`).
  If a test isn't practical, give exact manual verification steps.
- Run the related tests, lint and typecheck.

## 7. Explain

```
## Bug report
Symptom: ...
Root cause: ...
Fix: what changed and why it fixes the cause
Test: regression test added at ...
Related risks / similar spots: ...
How to avoid next time: (one practical lesson)
```

If the bug was caused by a recurring trap, suggest adding it to **Known traps** in `CLAUDE.md`.
