# Session Handoff

## What This Repo Is

A Claude Code plugin marketplace at `ac8318740/ac8318740-plugins`. Contains one plugin so far: **SpecHub** (`plugins/spechub/`).

## What SpecHub Is

A generalized version of the workflow developed in `/home/acoote/upstream project/.claude/`. It's a Claude Code plugin that combines:

- **OpenSpec CLI** for spec-driven development (proposals, designs, tasks, living specs, archiving)
- **Three-phase TDD pipeline** with structurally enforced isolation (test-writer can't see impl plans, executor can't modify tests, checker verifies everything)
- **Orchestrator pattern** where Claude delegates all code work to subagents
- **Living specifications** that auto-sync on every commit
- **Playwright-based frontend visual verification** (conditional on frontend config)
- **Quality gates**: mock skepticism, test baseline enforcement, regression checks, TDD isolation audits

## How the Plugin Works

- **Skills** (14 total) in `plugins/spechub/skills/` — the main workflow: init, propose, clarify, design, tasks, implement, implement-quick, commit, archive, bootstrap, sync, verify, explore, sync-upstream
- **Agents** (3) in `plugins/spechub/agents/` — test-writer, task-executor, task-checker
- **CLAUDE.md** at `plugins/spechub/CLAUDE.md` — orchestrator instructions that get `@import`ed into the user's project CLAUDE.md. The `/spechub:init` skill adds this import line automatically.
- **Profiles** in `plugins/spechub/profiles/` — preset configs for python, node-typescript, fullstack-python. Used by init to generate `openspec/project.yaml`.
- **Hooks** in `plugins/spechub/hooks/` — SessionStart hook that checks for project config.

## Key Design Decisions Made

1. **No `settings.json` agent override.** We explored using `settings.json` with `"agent": "specflow-orchestrator"` to activate a main-thread agent, but rejected it. The docs say it replaces the Claude system prompt entirely, and it's unclear how multiple plugins interact. Too aggressive for a distributed plugin. Instead, orchestrator instructions live in CLAUDE.md and get `@import`ed — additive, explicit, plays nice with other plugins.

2. **`@import` added by init skill.** Users run `/spechub:init`, which detects project type, writes `openspec/project.yaml`, and adds `@<plugin-path>/CLAUDE.md` to their project's CLAUDE.md. One-time setup, single line, removable.

3. **All commands parameterized via `openspec/project.yaml`.** No hardcoded test/build/lint commands. Everything comes from the project config. Profiles provide sensible defaults.

4. **upstream project-specific tools removed.** LEANN, Serena, Agno, Langfuse, shadcn, K8s references all stripped. Generic alternatives used (Grep/Glob, Explore subagents).

5. **`/spechub:sync-upstream` skill** for keeping the plugin in sync with ongoing upstream project workflow changes. Diffs upstream project's `.claude/` against the plugin, classifies changes as generalizable vs project-specific, proposes adaptations, checks for upstream project-specific leakage.

6. **Naming**: Was "specflow" but that name is taken (github.com/specstoryai/specflow). Renamed to SpecHub (code: `spechub`).

7. **Author pseudonym**: `ac8318740` — not real name.

## Credits in README

- **OpenSpec** (github.com/Fission-AI/OpenSpec) — credited as core spec engine, not just "built on"
- **Taskmaster AI** (github.com/eyaltoledano/claude-task-master) — "also inspired by"
- Also mentions Superpowers, GSD, Spec Kit as alternatives worth looking at

## What's Left To Do

- **Test the plugin locally** with `claude --plugin-dir ./plugins/spechub` — nothing has been tested yet
- **Git init and push** to GitHub as `ac8318740/ac8318740-plugins`
- **Validate** with `claude plugin validate .` or `/plugin validate .`
- **The skills reference `openspec` CLI commands** — need to verify they work with current OpenSpec CLI version
- **Skill content review** — all skills were generalized from upstream project but haven't been tested in a clean project
- **The `profiles/` directory** is not a standard plugin component — verify the init skill can actually read from `${CLAUDE_PLUGIN_ROOT}/profiles/`
- **Consider submitting to official Anthropic marketplace** later (optional, requires review)

## Source Material

The upstream project originals that were generalized:
- `/home/acoote/upstream project/CLAUDE.md` — the original orchestrator instructions
- `/home/acoote/upstream project/.claude/skills/` — original skills
- `/home/acoote/upstream project/.claude/agents/` — original agents
