---
name: scaffold
description: Create new building blocks that match our project conventions exactly - API endpoints, frontend pages, forms, components, DB migrations and similar. Use this whenever the user asks to create/add/generate a new endpoint, route, controller, page, screen, form, component, migration, table, model or similar boilerplate ("add a GET /orders/:id endpoint", "create a settings page", "new migration to add a status column"), and whenever the new-feature workflow needs one of these.
---

# Scaffold

The goal is that new code looks like the best existing code in the project, not like a generic
tutorial. The way to get there is to copy a real, approved example from the repo and adapt it,
rather than writing from memory.

## Steps

1. **Identify the kind** of thing being created and read the matching reference:

   | Kind | Reference |
   |------|-----------|
   | API endpoint / route | `references/endpoint.md` |
   | Frontend page / screen | `references/page.md` |
   | Form | `references/form.md` |
   | Component | `references/component.md` |
   | DB migration / schema change | `references/migration.md` |
   | TODO(team): other recurring kinds (background job, email template, cron, ...) | add a reference file |

   If none matches, use the closest one and the general rules below.

2. **Find the golden example.** Look up the kind in the **Golden examples** table of the project's
   `CLAUDE.md` and read that example in full (and its test). If no golden example is listed, find
   2-3 existing instances, pick the cleanest one, show the user which one you'll copy and why,
   and avoid anything in **Do not copy**.

3. **Collect inputs.** Ask only for what you can't infer: names, fields and types, permissions,
   validation rules. Ask once, in one message.

4. **Create the files** following the reference's checklist. Mirror the example's file layout,
   naming, imports, error handling and validation style.

5. **Wire it up.** New code is usually useless until registered somewhere (router, module,
   DI container, navigation menu, index exports, permissions table). The reference lists the
   usual places; check the golden example's commit or its usages to find any others.

6. **Add tests** in the same style as the golden example's tests (see `write-tests` skill).

7. **Verify** with lint + typecheck + the new tests, using the commands in `CLAUDE.md`.

8. **Report**: list created/changed files, which example was copied, and anything the user must
   do manually (env var, permission seeding, running a migration).

## General rules

- Reuse existing shared types, DTOs, hooks, services and UI components instead of creating
  parallel ones.
- Don't add new dependencies without asking.
- Leave no placeholder code (`// TODO implement`) without telling the user.
