---
name: run-local
description: Get a project running on the developer's machine and fix local setup problems - dependencies, env vars, Docker, database, migrations, seeding, ports. Use this whenever the user wants to run, start, set up or install a project locally, just cloned a repo, switched to a project they haven't worked on, or hits errors like "can't connect to database", "port already in use", "module not found", "env variable missing", "docker won't start", or "it works for X but not for me".
---

# Run locally

People switch between many projects, and each has its own setup quirks. Walk the user through
it step by step, checking each step works before moving on, so that when something fails you know
exactly which step broke.

## Safety rules

- **Never invent secrets or credentials.** If an env var needs a real value, tell the user where
  to get it (per `CLAUDE.md`) and let them fill it in. Don't print secret values back in chat.
- **Never point local config at production** (prod DB URLs, prod API keys), even if it "would work".
- Ask before anything destructive: dropping/resetting the local DB, deleting Docker volumes,
  `rm -rf node_modules`, killing processes the user may need.

## Steps

1. **Read `CLAUDE.md` → Run locally** and **Commands**. If missing, infer from README,
   `package.json`/build files, `docker-compose*.yml`, `.env.example`, and tell the user the
   project should get a **Run locally** section (offer to draft it once the app is running).

2. **Check prerequisites** and report what's missing, with how to install it:
   - Runtime versions vs. what the project expects (`.nvmrc`, `engines`, `.tool-versions`, etc.)
   - Docker running, if required
   - TODO(team): other team-wide tools (VPN, internal registry login, certificates...)

3. **Environment variables**: compare `.env.example` (or equivalent) with the user's `.env` and
   list missing keys by name only. For each, say where the value comes from.
   TODO(team): team-wide rule for getting secrets (vault, password manager, ask the tech lead...).

4. **Dependencies**: install with the project's command. If it fails, read the error, don't just retry.
   TODO(team): private registry setup if relevant.

5. **Services**: start DB/cache/etc. (e.g. `docker compose up -d <services>`), confirm they're healthy.

6. **Database**: run migrations, then seed. Confirm with a quick query or the app's health check.

7. **Start the app** and confirm it actually responds (health endpoint, open the URL, log in with
   the seed user from `CLAUDE.md`).

8. **Summarize** what was done and, if any step needed a workaround, offer to add it to the
   project's **Common setup errors** table in `CLAUDE.md` so the next person doesn't hit it.

## When something fails

First look up the error in **Common setup errors** in `CLAUDE.md`, then the general table below.
Diagnose from the actual error text instead of trying random fixes.

| Symptom | Usual cause | What to check |
|---------|-------------|---------------|
| Port already in use | Another instance / another project running | Find the process on the port; ask before killing it |
| Can't connect to DB | Container not running, wrong host/port in `.env`, DB not created yet | `docker ps`, `.env` values vs compose file |
| Module not found / wrong version | Deps out of date after pull, wrong runtime version | Reinstall deps, check runtime version |
| Missing env var / undefined config | `.env` missing new keys from a recent change | Diff `.env.example` vs `.env` |
| Migration fails | Local DB in a state from another branch | Ask before resetting the local DB |
| Works for teammate, not for me | Different runtime version, stale `.env`, OS differences (Windows paths/line endings) | Compare versions and env |
| TODO(team) | | |
