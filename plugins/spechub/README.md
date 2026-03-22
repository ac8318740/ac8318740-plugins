# SpecHub

A Claude Code plugin for spec-driven TDD development.

## Credits

- **[OpenSpec](https://github.com/Fission-AI/OpenSpec)** — SpecHub uses OpenSpec as its core spec engine. The entire spec-driven workflow (proposals, designs, tasks, living specs, change management, archiving) runs on the OpenSpec CLI. If you're interested in spec-driven development without the TDD and orchestration layers that SpecHub adds, OpenSpec on its own is great.
- **[Taskmaster AI](https://github.com/eyaltoledano/claude-task-master)** — The orchestrator pattern and agent coordination approach in SpecHub were also inspired by Taskmaster's task management model.

Also worth looking at if you're exploring this space: [Superpowers](https://github.com/obra/superpowers), [GSD](https://github.com/gsd-build/get-shit-done), and [Spec Kit](https://github.com/github/spec-kit). They all take different angles on the same general problem.

## How is this different?

Every rule here exists because something went wrong without it. Built over months of actual product development with Claude Code, not designed upfront.

- **TDD is structural, not aspirational.** Test-writer can't see the implementation plan. Executor can't touch test files. Tests stay independent of the code they verify. (Influenced by OpenSpec's proposal/design separation.)
- **Specs auto-sync.** Every commit updates the living specs. Agents fix inaccuracies on sight. Specs converge toward reality over time instead of drifting. (Powered by OpenSpec's delta model.)
- **4x more effort on planning than coding.** Three parallel explorers before any code is written. Mock audits, mutation checks, regression suites, integration wiring. Most bugs came from not understanding existing code, not from writing bad new code.
- **Strict defaults, easy to relax.** Orchestrator mode, TDD pipeline, spec workflow — all configurable. But adding discipline later is harder than loosening it.

## What It Does

1. **Spec-driven development** — Features start as proposals, get designed, broken into tasks, then implemented. This workflow is powered by [OpenSpec](https://github.com/Fission-AI/OpenSpec).
2. **Three-phase TDD pipeline** — test-writer (writes failing tests) -> task-executor (makes them pass) -> task-checker (verifies everything)
3. **Living specifications** — Cumulative specs that stay in sync with your codebase, using OpenSpec's delta-based change management
4. **Orchestrator pattern** — Claude coordinates specialized agents rather than doing everything itself
5. **Quality gates** — Mock skepticism, test baseline enforcement, regression checking, TDD isolation audits
6. **Frontend visual verification** — Playwright-based UI verification when a frontend is present

## Prerequisites

- [Claude Code](https://claude.com/claude-code) CLI
- Node.js >= 20 (for the SpecHub CLI)

## Installation

```
/plugin marketplace add ac8318740/ac8318740-plugins
/plugin install spechub@ac8318740-plugins
```

Then in your project:

```
/spechub:init
```

This detects your project type, generates `openspec/project.yaml`, and adds an `@import` line to your CLAUDE.md that activates the orchestrator.

## Skills

### Spec Workflow (Full Path)

| Skill | Description |
|-------|-------------|
| `/spechub:propose` | Create a feature proposal with user stories |
| `/spechub:clarify` | Resolve ambiguities in the proposal |
| `/spechub:design` | Generate implementation design |
| `/spechub:tasks` | Generate dependency-ordered task list |
| `/spechub:implement` | Execute tasks via TDD pipeline |
| `/spechub:archive` | Archive change, update living specs |

### Fast Path

| Skill | Description |
|-------|-------------|
| `/spechub:implement-quick` | Quick implementation with deep analysis |
| `/spechub:commit` | Git commit with automatic spec sync |

### Supporting

| Skill | Description |
|-------|-------------|
| `/spechub:init` | Initialize SpecHub in a project |
| `/spechub:bootstrap` | Generate initial living specs from code |
| `/spechub:sync` | Update specs from code changes |
| `/spechub:verify` | Cross-artifact consistency analysis |
| `/spechub:explore` | Thinking partner mode (read-only) |

## Agents

| Agent | Role |
|-------|------|
| `test-writer` | TDD Phase 1: writes failing tests from requirements only |
| `task-executor` | TDD Phase 2: makes tests pass, cannot modify tests |
| `task-checker` | TDD Phase 3: verifies everything (mock audit, regression, visual) |

## Language Profiles

- **python** — pytest, ruff, mypy
- **node-typescript** — npm test, eslint, tsc
- **fullstack-python** — Python backend + Node/TS frontend

## The Orchestrator Pattern

By default, SpecHub enforces a strict orchestrator pattern where Claude delegates all code work to specialized subagents. Set `orchestrator.strict: false` in `openspec/project.yaml` to relax this for smaller projects.
