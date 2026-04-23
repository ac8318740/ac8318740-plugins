# ac8318740-plugins Development

## What This Repo Is

A Claude Code plugin marketplace at `ac8318740/ac8318740-plugins`. The main plugin is **SpecHub**, which lives in its own repo (`ac8318740/spechub`) and is referenced here as a git submodule at `plugins/spechub/`.

## Repo Structure

```
.claude-plugin/marketplace.json  – Plugin marketplace registry
.gitmodules                      – Submodule references
plugins/spechub/                 – Submodule: ac8318740/spechub
.claude/skills/commit/           – Dev skill: commit and push to one or both repos
.claude/skills/sync-upstream/    – Dev skill: sync upstream workflow changes into SpecHub (gitignored)
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

### `/sync-upstream` – Sync from upstream

Pulls workflow changes from the upstream project into SpecHub. Diffs, classifies changes as generalizable vs project-specific, presents each for approval. This skill is gitignored – it's a development tool, not part of the published plugin.

## Key Context

- **Author pseudonym**: `ac8318740`

## Releasing open-designer

**Critical**: `open-designer` ships as BOTH a Claude plugin AND an npm package (`open-designer-viewer`, used via `npx open-designer-viewer`). These two MUST stay in lock-step.

**The rule (no exceptions):** any user-visible change under `plugins/open-designer/` – skills, README, briefing docs, `launcher/`, `viewer/`, anything a plugin consumer would notice – requires a plugin version bump AND an npm republish in the same commit. "Docs only" and "skills only" both count as user-visible: the plugin cache on each user's machine only picks up changes on a version bump, so without one the new docs/skills never reach anyone.

- Source of truth for the version is `plugins/open-designer/.claude-plugin/plugin.json`.
- `plugins/open-designer/package.json` is synced from it automatically – never edit its `version` by hand.
- To release: bump `plugin.json`, then from `plugins/open-designer/` run `npm run release`. This syncs the version, builds the viewer, and publishes.
- Full details: `plugins/open-designer/RELEASING.md`.

If you bump plugin.json and forget to republish to npm, users on `npx open-designer-viewer` stay stuck on the old version. If you change skills/docs without bumping plugin.json, users on any device never see the update.

## Commands

No repo-wide build/test/lint yet – skills and agents are markdown. Per-plugin commands:

- `plugins/open-designer/`: `npm run build:viewer`, `npm run sync-version`, `npm run release`.
