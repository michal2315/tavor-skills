---
name: pre-mr
description: Self-review the current branch before opening a merge request - run lint/typecheck/tests, review the diff against the team checklist, report problems by severity, and draft the MR description. Use this whenever the user says they're done, wants to open/create/submit an MR or PR, asks "is this ready?", "review my changes", "check my code before I push", or finishes a feature. Prefer it over a generic code review for our projects.
---

# Pre-MR self-review

The aim is to catch what a senior reviewer would comment on, before they have to. Reviewers' time
is the team's bottleneck, so a clean MR saves everyone a round trip.

Report problems and let the user decide - don't silently rewrite their code. Offer to fix, and fix
on a yes.

## 1. Collect the diff

- Base branch: from **Git & MR** in `CLAUDE.md` (TODO(team): default if missing, e.g. `develop`).
- `git fetch origin <base>` then review `git diff origin/<base>...HEAD` plus uncommitted changes
  (`git status`, `git diff`).
- If the diff is huge (TODO(team): e.g. >800 changed lines) tell the user big MRs get worse reviews
  and suggest splitting, then continue.

## 2. Run the automatic checks

Using the exact commands in `CLAUDE.md` **Commands**: lint, typecheck, tests (affected first,
then full suite if reasonable), build if it's quick (TODO(team): decide).

Record pass/fail with the relevant error lines. A failing check is always a blocker.

## 3. Review the diff

Go through `checklist.md` in this folder. Read changed files in context (not just the hunks) when
needed to judge a point. Check that new code follows the project's **Golden examples** and
**Conventions** in `CLAUDE.md`.

Only report real findings with a file:line and a concrete reason. Skip nitpicks that the linter
would catch or that don't matter.

## 4. Report

Use this format:

```
## Pre-MR report

### Checks
| Check | Result |
|-------|--------|
| Lint | ✅ / ❌ (n errors) |
| Typecheck | ✅ / ❌ |
| Tests | ✅ n passed / ❌ n failed |

### 🔴 Blockers (must fix before MR)
- `path/file.ts:42` - what's wrong - why it matters - suggested fix

### 🟡 Should fix
- ...

### 🟢 Suggestions (optional)
- ...

Verdict: Ready for MR / Fix blockers first
```

Then ask whether to fix the blockers and should-fix items.

## 5. Draft the MR description

When there are no blockers (or the user asks anyway), draft the MR description using the team
template:

TODO(team): paste your MR template here. Default:

```
## What
<1-3 sentences, plain language>

## Why
<ticket link / reason>

## How
<key implementation points a reviewer should know>

## How to test
1. ...

## Checklist
- [ ] Tests added/updated
- [ ] Migration included: yes/no
- [ ] New env vars: none / list
- [ ] Screenshots (UI changes)
```

TODO(team): if the team uses the `glab` CLI and you want Claude to open the MR, add the command
here (e.g. `glab mr create --target-branch develop --title ... --description ...`) and require
the user's explicit OK before running it.
