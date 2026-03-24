# ac8318740-plugins Development

## What This Repo Is

A Claude Code plugin marketplace at `ac8318740/ac8318740-plugins`. The main plugin is **SpecHub**, which lives in its own repo (`ac8318740/spechub`) and is referenced here as a git submodule at `plugins/spechub/`.

## Repo Structure

```
.claude-plugin/marketplace.json  – Plugin marketplace registry
.gitmodules                      – Submodule references
plugins/spechub/                 – Submodule: ac8318740/spechub
.claude/skills/commit/           – Dev skill: commit and push to one or both repos
.claude/skills/sync-upstream/    – Dev skill: sync upstream project workflow changes into SpecHub (gitignored)
```

## Writing Standards

All prose in this project – commit messages, READMEs, skill docs, comments – must follow these rules:

- **En dashes (–) only.** Never em dashes.
- **Simple language.** Short sentences. Plain words over jargon.
- **Concise.** No filler, no repetition, no marketing tone.
- **Active voice.** "Add feature" not "Feature was added."

## Working on SpecHub

The plugin code at `plugins/spechub/` is a **git submodule**. It has its own repo, its own commits, and its own CLAUDE.md (which is the plugin's orchestrator instructions for end users – not development instructions).

To make changes to SpecHub:
1. Work inside `plugins/spechub/` – it's a full git repo
2. Use `/commit` to commit and push (handles both repos automatically)

## Dev Skills

### `/commit` – Commit and push

Commits to spechub, the marketplace parent, or both. Auto-detects which repos have changes. Enforces writing standards on commit messages. See `.claude/skills/commit/SKILL.md`.

### `/sync-upstream` – Sync from upstream project

Pulls workflow changes from `~/upstream project/.claude/` into SpecHub. Diffs, classifies changes as generalizable vs upstream project-specific, presents each for approval. This skill is gitignored – it's a development tool, not part of the published plugin.

## Key Context

- **Author pseudonym**: `ac8318740`
- **upstream project** (`~/upstream project`) is the upstream source – workflow innovations happen there first, then get generalized into SpecHub via sync-upstream
- **OpenSpec CLI** is being forked/rebranded as **SpecHub CLI** (MIT license allows this). The rename from `openspec` to `spechub` is in progress.
- Both repos are **private** for now

## Commands

No build/test/lint commands yet – the plugin is all markdown skills and agent definitions. The CLI (when built) will have its own test/build setup.
