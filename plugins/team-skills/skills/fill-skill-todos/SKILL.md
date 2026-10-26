---
name: fill-skill-todos
description: Maintainer tool - fill in the TODO(team) placeholders in a project's CLAUDE.md (and optionally the team-skills plugin) by inspecting the actual codebase, then confirm uncertain items with the user. Use when the user asks to set up, fill in, complete or update CLAUDE.md or the team skills for a project, or runs /fill-skill-todos.
disable-model-invocation: true
---

# Fill skill TODOs

Used by the team lead to finish the skill setup inside each project. Most answers are in the code;
the rest only the lead knows. Separate the two clearly so nothing guessed ends up looking like a
confirmed team rule.

## Steps

1. **Locate the targets**
   - The project's `CLAUDE.md`. If it doesn't exist, copy `CLAUDE.md.template` from this skill's
     folder into the project root as `CLAUDE.md`.
     If an existing `CLAUDE.md` has other content, merge the template sections in rather than overwriting.
   - If the user also wants to fill the plugin's TODOs, they'll give you the path to the
     team-skills repo.
   - List every `TODO(team)` occurrence with file and line.

2. **Investigate the codebase** to answer what can be inferred:
   - Stack and versions: package/build files, lockfiles, Dockerfiles
   - Commands: scripts in `package.json` / Makefile / etc., CI config (the CI pipeline shows the
     commands that really run)
   - Run locally: README, `docker-compose*.yml`, `.env.example`, seed scripts
   - Conventions and golden examples: look at several instances of each kind (endpoints, pages,
     forms, components, migrations, tests); propose the cleanest, most recent one as golden and
     note older/messier patterns as "Do not copy" candidates. `git log` dates help tell old from new.
   - Logs, known traps: README, comments, CI config

3. **Fill in confidently-inferred items** directly. Mark anything inferred but uncertain with
   `TODO(team): verify -` followed by your guess, so it's easy to spot.

4. **Ask the user** in one message, grouped, for what only they know: team rules, common review
   comments, what not to touch, where secrets come from, MR template. Offer your best guess for each
   so they can just confirm.

5. **Apply answers**, then re-list remaining `TODO(team)` items (should be none or few).

6. **Smoke-test** the commands in **Commands** that are safe to run (lint, typecheck, tests) and
   fix any that are wrong. Don't run anything that deploys, resets data, or touches remote systems.

Keep `CLAUDE.md` concise - it's loaded into every conversation, so long prose costs context in
every session. Prefer tables and short bullets.
