---
name: new-feature
description: Guided workflow for building a feature or behaviour change in one of our projects - clarify, plan, get approval, implement following project patterns, test, verify, summarize. Use this whenever the user asks to add, build, implement, extend or change functionality ("add a filter to the orders page", "implement PROJ-123", "users should be able to export to CSV", "make the API return X too"), even if they don't say "feature". Not for pure bug fixes (use debug) or adding only tests (use write-tests).
---

# New feature

The people using this skill are often not experienced developers. They may accept whatever you
produce without being able to judge it. So your job is not only to write code, but to keep them
in the loop at the moments that matter, and to make sure the result fits the project instead of
being a plausible-looking one-off.

## 0. Load project context

Read the project's `CLAUDE.md` first, especially **Commands**, **Git & MR**, **Conventions**,
**Golden examples**, **Do not copy** and **Do not touch without asking**.
If `CLAUDE.md` is missing or a section you need is empty, tell the user once and continue by
inferring from the code - but prefer asking over guessing when it affects behaviour.

## 1. Understand the request

Restate the feature in 2-4 bullet points in plain language: what changes for the end user.

Then ask the clarifying questions that would actually change the implementation. Typical gaps:
- Who can do this? (roles / permissions)
- What happens on empty / invalid / error cases?
- Does existing data need to change (migration, backfill)?
- Is there a UI part, an API part, or both?
- TODO(team): add questions your team always ends up asking, e.g. "Does this need to be behind a feature flag?", "Does it need translations?"

If the request references a ticket and you can't read it, ask the user to paste the ticket text.

Ask everything in one message and wait. If the request is already fully clear, say so and skip to the plan.

## 2. Explore before planning

Find where similar functionality already lives. Use the **Golden examples** from `CLAUDE.md` as
the pattern to follow. Avoid anything under **Do not copy**. Look for existing helpers, hooks,
services, components and types you can reuse instead of writing new ones - duplicate helpers are
one of the most common review complaints.

## 3. Plan and wait for approval

Present a short plan:

```
## Plan
Files to change:
- path/to/file.ts - what changes
New files:
- path/to/new.ts - purpose (pattern copied from <golden example>)
DB changes: none | migration: <what>
Tests: what will be tested and where
Risks / open questions: ...
```

If the plan touches anything under **Do not touch without asking**, call it out explicitly and
suggest the user checks with a senior before approving.

**Stop and wait for the user to approve the plan.** This is the user's main chance to catch a
misunderstanding cheaply.

## 4. Branch

If the user is on the base branch, create a branch using the naming format in `CLAUDE.md`
(TODO(team): confirm format, e.g. `feature/PROJ-123-short-name`).

## 5. Implement

- Follow the golden examples closely: same file layout, naming, error handling, validation style.
- Keep the change focused on the plan. If you discover something else that needs fixing, note it
  for the summary instead of fixing it silently.
- For DB changes, use the project's migration command; never edit a migration that may already
  have been applied.
- For anything that needs a scaffold (endpoint, page, migration, component), use the `scaffold` skill.

## 6. Tests

Add or update tests following the `write-tests` skill. A feature without tests is not done unless
the user explicitly agrees.

## 7. Verify

Run, using the exact commands from `CLAUDE.md`:
1. Lint (autofix first if available)
2. Typecheck
3. Tests related to the change, then the full suite if reasonably fast

Fix failures caused by your change. If something was already failing before your change, say so
rather than "fixing" unrelated code.

If there is a UI part, tell the user exactly how to see it working (URL, steps, test user).

## 8. Summarize

End with:

```
## Done
What changed (for the user, plain language): ...
Files: ...
How to try it: ...
Tests: added X, all passing / N failing (why)
Not done / follow-ups: ...
Next step: run /pre-mr before opening the MR.
```

Commit only if the user asks, using the commit format from `CLAUDE.md`.
