# Pre-MR checklist

Severity: 🔴 blocker, 🟡 should fix, 🟢 suggestion. Adjust severities to taste.

## Team's most common review comments

TODO(team): THIS IS THE MOST VALUABLE SECTION. Write the 5-15 comments you leave on MRs over and
over again, with the severity and, ideally, a "good" vs "bad" example. Examples:

- 🟡 Business logic in the controller instead of the service
- 🟡 New helper duplicates an existing one in `src/utils/`
- 🔴 Endpoint missing permission check
- 🟡 Component fetches data directly instead of using a hook from `src/hooks/api`

## Security
- 🔴 Secrets, tokens, passwords, real customer data in code, config, tests or logs
- 🔴 Missing auth/permission check on new endpoints or actions
- 🔴 SQL built by string concatenation with user input; unsanitized HTML rendering (`dangerouslySetInnerHTML`, `v-html`)
- 🔴 Trusting client-side data for authorization (user id / role taken from the request body)
- 🟡 Sensitive data returned in API responses that the client doesn't need

## Correctness
- 🔴 Obvious logic bugs, wrong conditions, off-by-one, unhandled null/undefined
- 🔴 Errors swallowed (`catch {}` / catch-and-log-only where the caller needs to know)
- 🟡 Missing handling of empty / loading / error states in UI
- 🟡 Async issues: missing `await`, unhandled promise, race conditions on double submit

## Data & performance
- 🔴 Editing a migration that's already merged; non-backward-compatible migration
- 🟡 N+1 queries (DB call inside a loop)
- 🟡 Unbounded queries / list endpoints without pagination
- 🟡 Heavy work inside render or on every request where it could be cached/memoized

## Code quality
- 🟡 Doesn't follow the project's golden example / conventions in CLAUDE.md
- 🟡 Duplicated code or a new helper that already exists
- 🟡 `any` / type assertions hiding real type problems (TODO(team): adjust per language)
- 🟢 Unclear names, functions doing too much
- 🟡 Left-overs: `console.log`, debugger, commented-out code, unrelated formatting changes, TODOs without a ticket

## Tests
- 🔴 New behaviour with no tests (unless agreed)
- 🔴 Tests deleted, skipped (`.skip`, `xit`) or assertions weakened to make them pass
- 🟡 Tests that don't actually assert the behaviour (only "renders without crashing", only checks a mock was called)

## Scope & hygiene
- 🟡 Changes unrelated to the MR's purpose
- 🟡 Files under "Do not touch without asking" (CLAUDE.md) changed - flag for senior review
- 🟡 New dependency added - justify it
- 🟡 New env var not added to `.env.example` / docs
- 🟢 Branch name / commit messages not following the convention
