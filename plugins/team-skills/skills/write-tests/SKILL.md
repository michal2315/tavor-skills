---
name: write-tests
description: Write meaningful tests that follow the project's existing test patterns - decide what to test, copy the style of the golden test example, run them, and make sure they really catch bugs. Use this whenever the user asks to add, write, fix or improve tests, increase coverage, test a function/endpoint/component/page, or when another workflow (new-feature, debug, scaffold) needs tests.
---

# Write tests

A test is only worth having if it would fail when the behaviour breaks. Tests that just render a
component or assert that a mock was called pass forever and give false confidence. Aim for fewer,
meaningful tests over many trivial ones.

## 1. Learn the project's testing setup

From `CLAUDE.md`: test commands (**Commands**) and the test **Golden examples**. Read the golden
test fully: file location and naming, setup/teardown, factories/fixtures, how the DB and HTTP are
handled, assertion style.

TODO(team): fill in team-wide testing conventions:
- Unit vs integration split: TODO(team) (e.g. services get unit tests, endpoints get integration tests against a test DB)
- What to mock: TODO(team) (e.g. mock external HTTP APIs, never mock our own DB layer in integration tests)
- Test data: TODO(team) (factories in `test/factories`? fixtures?)
- Frontend: TODO(team) (Testing Library queries by role/label, no snapshot tests?)
- Coverage expectations: TODO(team)

## 2. Decide what to test

For the code under test, list the behaviours that matter, then pick tests for:
- The main happy path
- Validation / bad input
- Permissions (unauthorized, wrong role) for endpoints and protected UI
- Edge cases: empty lists, null/missing values, boundaries, duplicates
- Error paths: what happens when a dependency fails
- For bug fixes: a regression test reproducing the exact bug

Show the list to the user briefly before writing many tests, so they can add cases they know matter.

## 3. Write them

- Put them where the golden example puts them, with the same naming.
- Test behaviour through the public interface (function output, HTTP response, what the user sees),
  not internal implementation details.
- Each test name describes the behaviour: `returns 403 when user is not the order owner`.
- Arrange / act / assert, one behaviour per test.
- Reuse existing factories/helpers; add new ones in the shared place if needed.
- No real external services, no reliance on test order, no sleeps for timing.

## 4. Run and validate

1. Run the new tests with the single-file command from `CLAUDE.md`. They must pass.
2. **Check they can fail**: temporarily break the code under test (e.g. invert a condition, return
   early) and confirm at least the relevant test goes red, then restore the code. If nothing fails,
   the test isn't testing the behaviour - fix it.
3. Run the related suite to make sure nothing else broke.

Never make tests pass by weakening assertions, adding `.skip`, or changing the code's behaviour to
match a wrong test without asking.

## 5. Report

List the tests added (by behaviour), where they live, results, and any behaviour you chose not to
test and why.
