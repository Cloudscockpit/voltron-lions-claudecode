# Voltron-Lions

A Claude Code plugin for mission-style multi-agent orchestration. One command. Five themed agents. Structured outputs.

## What it does

`/start-mission <objective>` invokes **Voltron Main** (the Black Lion), who:

1. Drafts a **Mission Brief** (objective, scope, success criteria, constraints)
2. Sends **Green Lion** on recon (uses `graphify-out/` if present and fresh; falls back to filesystem)
3. Generates **Lion Assignments** with strict file-ownership boundaries
4. Performs **Skills Gap** analysis — flags missing capabilities, can scaffold new skills via `skill-creator`
5. Builds a **Risk Register** (technical / scope / integration / data-loss, severity L/M/H)
6. Waits for your **go / no-go**, then dispatches Red / Blue / Yellow Lions to execute

## The Lions

| Lion | Role |
|------|------|
| **Black — Voltron Main** | Commander. Plans, delegates, gates, reports. |
| **Red** | Rapid execution — builds features, writes code. |
| **Blue** | Data & integrations — APIs, schemas, plumbing. |
| **Green** | Recon — codebase analysis (graphify-aware). |
| **Yellow** | Quality & defense — tests, security, verification. |

## Install

From the project root, add this plugin to Claude Code via `/plugin` and point it at this directory, or copy the repo into your plugin marketplace.

## Usage

```
/start-mission add a /health endpoint to the express app that returns service version
```

Voltron Main returns a four-part report. Reply `go` to dispatch Lions, `no-go` to revise, or `edit <section>` to change one section.

## Tips

- For large codebases, run `/graphify` first so Green Lion's recon is graph-backed.
- If Voltron Main flags `needs-new-skill` gaps, it will ask before scaffolding via `skill-creator`. New skills become available on the next session.

## Design

See [docs/superpowers/specs/2026-05-28-voltron-lions-plugin-design.md](docs/superpowers/specs/2026-05-28-voltron-lions-plugin-design.md).
