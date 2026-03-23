# ac8318740-plugins Development

## What This Repo Is

A Claude Code plugin marketplace at `ac8318740/ac8318740-plugins`. The main plugin is **SpecHub**, which lives in its own repo (`ac8318740/spechub`) and is referenced here as a git submodule at `plugins/spechub/`.

## Repo Structure

```
.claude-plugin/marketplace.json  — Plugin marketplace registry
.gitmodules                      — Submodule references
plugins/spechub/                 — Submodule: ac8318740/spechub (DO NOT edit directly here for plugin changes)
.claude/skills/sync-upstream/    — Dev tool: sync upstream project workflow changes into SpecHub (gitignored)
HANDOFF.md                       — Historical context and design decisions
```

## Working on SpecHub

The plugin code at `plugins/spechub/` is a **git submodule**. It has its own repo, its own commits, and its own CLAUDE.md (which is the plugin's orchestrator instructions for end users — not development instructions).

To make changes to SpecHub:
1. Work inside `plugins/spechub/` — it's a full git repo
2. Commit and push there (to `ac8318740/spechub`)
3. Then in this parent repo, commit the updated submodule pointer

## Dev Tools (gitignored)

### `/sync-upstream` — Sync from upstream project

The sync-upstream skill at `.claude/skills/sync-upstream/` pulls workflow changes from `~/upstream project/.claude/` into SpecHub. It:
- Diffs upstream project's `.claude/` against `plugins/spechub/`
- Classifies changes as generalizable vs upstream project-specific
- Presents each change for approval (apply/skip/modify)
- Applies approved changes with upstream project-specific references stripped

This skill is gitignored — it's a development tool, not part of the published plugin.

## Key Context

- **Author pseudonym**: `ac8318740`
- **upstream project** (`~/upstream project`) is the upstream source — workflow innovations happen there first, then get generalized into SpecHub via sync-upstream
- **OpenSpec CLI** is being forked/rebranded as **SpecHub CLI** (MIT license allows this). The rename from `openspec` → `spechub` is in progress.
- Both repos are **private** for now

## Commands

No build/test/lint commands yet — the plugin is all markdown skills and agent definitions. The CLI (when built) will have its own test/build setup.
