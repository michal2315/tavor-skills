# Scaffold: DB migration / schema change

Migrations are the riskiest thing a junior can scaffold: they run against shared and production
data and are hard to undo. Be conservative and explain each step to the user.

## Tooling

TODO(team): ORM / migration tool (Prisma / TypeORM / Knex / Flyway / EF / Alembic ...),
the generate command, the apply command, and where migration files live.

## Rules

- **Never edit or delete a migration that may already be applied** (anything merged to the base
  branch). Create a new migration instead. Why: other environments have already run the old one,
  so editing it causes drift that's painful to fix.
- Generate migrations with the tool's command rather than hand-writing, then review the output.
- Make changes backward compatible with the currently deployed code, because migrations usually run
  before the new code is live:
  - Adding a NOT NULL column → add it nullable or with a default, backfill, then tighten later.
  - Renaming/removing a column → add new, migrate, remove old in a later release.
- Add indexes for new foreign keys and for columns used in WHERE/ORDER BY on big tables.
- Data backfills on large tables: TODO(team): your rule (separate script? batch size?).
- Provide a down/rollback if the tool supports it.
- TODO(team): naming convention for tables, columns, migration files.

## Checklist

- [ ] Schema/model file updated - TODO(team): path
- [ ] Migration generated with the project command
- [ ] Generated SQL reviewed and shown to the user
- [ ] Backward compatible (see rules)
- [ ] Indexes considered
- [ ] Applied locally successfully; rollback tested if supported
- [ ] Seed data / fixtures updated if needed
- [ ] Generated client/types regenerated - TODO(team): command (e.g. `prisma generate`)
- [ ] Mention in the MR description that it contains a migration

## Team notes

TODO(team)
