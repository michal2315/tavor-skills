# Scaffold: API endpoint

## Files usually involved

TODO(team): replace with your real layout. Example for a NestJS-style API:

| Layer | Location | Notes |
|-------|----------|-------|
| Route / controller | `src/modules/<module>/<module>.controller.ts` | thin: parse input, call service, return |
| Input validation | `src/modules/<module>/dto/<action>.dto.ts` | |
| Business logic | `src/modules/<module>/<module>.service.ts` | |
| Data access | TODO(team) repository / ORM model | |
| Response type | TODO(team) | |
| Test | TODO(team) | |

## Checklist

- [ ] Route path and HTTP method follow existing naming (TODO(team): e.g. plural nouns, kebab-case)
- [ ] Auth guard / permission check applied - TODO(team): how a route is protected here
- [ ] Input validated with TODO(team): validation library, all fields typed
- [ ] Business logic in the service, not the controller
- [ ] Errors use the project's error types - TODO(team): e.g. `NotFoundException`, custom `AppError`
- [ ] No N+1 queries: related data loaded with TODO(team): include / join / dataloader
- [ ] Pagination for list endpoints - TODO(team): the standard pagination params/shape
- [ ] Response shape matches existing endpoints (envelope? camelCase?)
- [ ] Registered: TODO(team): module imports, router file, API docs/OpenAPI decorators
- [ ] Frontend API client / types updated if the project has a shared client - TODO(team): where
- [ ] Tests: happy path, validation error, unauthorized, not found

## Team notes

TODO(team): anything specific, e.g. "All endpoints under /admin need the AdminGuard",
"Never return the raw DB entity - map to a response DTO".
