# Team skills for Claude Code

Shared Claude Code skills for our projects. Your Claude follows the team's workflow and
conventions without you having to know exactly what to ask.

| Skill | What it does | Example trigger |
|-------|--------------|-----------------|
| `new-feature` | Clarify → plan → **your approval** → implement → test → verify → summary | "add CSV export to the orders page" |
| `scaffold` | Creates endpoints, pages, forms, components and migrations by copying our best existing example | "new endpoint GET /orders/:id" |
| `pre-mr` | Runs lint/types/tests, reviews your diff against the team checklist, drafts the MR description | "I'm done, ready for MR?" |
| `debug` | Reproduce → root cause → minimal fix → regression test → explanation | paste an error / "X is broken" |
| `run-local` | Gets a project running locally and fixes setup errors | "how do I run this?" / "can't connect to db" |
| `write-tests` | Meaningful tests in the project's style, checked to really catch bugs | "add tests for OrderService" |
| `fill-skill-todos` | (maintainers) Fills the TODOs in a project's CLAUDE.md from the code | `/team-skills:fill-skill-todos` |

Skills trigger by themselves from what you type. You can also call one directly, e.g.
`/team-skills:pre-mr`.

## How it fits together

```
team-skills repo (this)            each project repo
├─ skills = HOW we work     +      CLAUDE.md = WHAT this project is
   (same for all projects)            (commands, golden examples, traps)
```

The skills are generic. They read project specifics from sections of each project's `CLAUDE.md`
(Commands, Golden examples, Conventions, Run locally, Known traps...). That way one set of skills
works for all our projects.

## Setup (maintainer, once)

1. **Push this folder to an internal GitLab repo**, e.g. `tools/claude-team-skills`.
2. **Fill team-wide TODOs** in the skills: `grep -rn "TODO(team)" plugins/`.
   The highest-value ones:
   - `pre-mr/checklist.md` → your most common review comments
   - `pre-mr/SKILL.md` → MR template, base branch
   - `write-tests/SKILL.md` → testing conventions
   - `scaffold/references/*.md` → file layout per kind
3. **For each project**: open Claude Code in the project and run `/team-skills:fill-skill-todos`.
   It copies `plugins/team-skills/skills/fill-skill-todos/CLAUDE.md.template` into the project as `CLAUDE.md`, fills what it can from
   the code and asks you the rest. Review it, then commit `CLAUDE.md` to the project.
4. Bump `version` in `plugins/team-skills/.claude-plugin/plugin.json` when you change skills.

## Install (every developer)

In Claude Code:

```
/plugin marketplace add <internal-gitlab-url-of-this-repo>
/plugin install team-skills@team-marketplace
```

A local clone path works for `marketplace add` too. To pick up updates later:
`/plugin marketplace update team-marketplace`.

To install it automatically for everyone who opens a project, you can add the marketplace and
plugin to the project's `.claude/settings.json` (`extraKnownMarketplaces` / `enabledPlugins`).
Check the Claude Code plugin docs for your version.

**Fallback without plugins**: copy `plugins/team-skills/skills/*` into `~/.claude/skills/`
(per user) or `<project>/.claude/skills/` (per project). Then you call them without a prefix (`/pre-mr`).

## Recommended daily flow

1. `run-local` when you start on a project
2. `new-feature` to build (it uses `scaffold` and `write-tests` on its own)
3. `debug` when something breaks
4. `pre-mr` before every MR

## Improving the skills

When Claude does something the team doesn't like:
- Project-specific → fix the project's `CLAUDE.md` (Conventions / Do not copy / Known traps).
- Applies to all projects → fix the skill here.
Whenever you write the same MR review comment twice, add it to `pre-mr/checklist.md`.
