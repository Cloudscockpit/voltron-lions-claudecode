# Voltron-Lions

A Claude Code & Claude Cowork plugin for mission-style multi-agent orchestration. One command. Five themed agents. Structured outputs.

## What it does

`/start-mission <objective>` invokes **Voltron Main** (the Black Lion), who:

1. Drafts a **Mission Brief** (objective, scope, success criteria, constraints)
2. Sends **Green Lion** on recon (uses `graphify-out/` if present and fresh; falls back to filesystem)
3. Generates **Lion Assignments** with strict file-ownership boundaries
4. Performs **Skills Gap** analysis using the built-in **skills registry & actions map** — flags missing capabilities, can scaffold new skills via `skill-creator`
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

## Installation

Pick the path that matches you. Both install the exact same plugin.

### 🧑‍💻 For developers (Claude Code)

In any Claude Code session, run these two commands:

```
/plugin marketplace add Cloudscockpit/voltron-lions-claudecode
/plugin install voltron-lions
```

Restart the session (or run `/reload`) and `/start-mission` is available.

### 🙋 For non-technical users (Claude Cowork / Claude Desktop)

No terminal needed — you type two lines into the chat box, same as sending a message:

1. Open **Claude Cowork** (or the Claude desktop app with Claude Code enabled).
2. In the chat box, type exactly this and press Enter:
   `/plugin marketplace add Cloudscockpit/voltron-lions-claudecode`
3. Then type this and press Enter:
   `/plugin install voltron-lions`
4. When asked to confirm, choose **Install / Yes**.
5. Start a new conversation. You're ready — type `/start-mission` followed by what you want done, in plain English. Example:
   `/start-mission organize the files in my project folder and tell me what's outdated`
6. Voltron will show you a mission plan and **wait for your approval** — nothing happens until you reply `go`.

> 💡 If a step fails with "requires a newer version", update your Claude app first, then repeat from step 2.

## Usage

```
/start-mission add a /health endpoint to the express app that returns service version
```

Voltron Main returns a four-part report. Reply `go` to dispatch Lions, `no-go` to revise, or `edit <section>` to change one section.

## Skills registry & actions map

The plugin ships a machine-readable registry of everything the Lions can do:

- **Skill:** `voltron-lions:skills-registry` — invoked by Voltron Main during Skills Gap analysis, or by you ("what can the Lions do?")
- **Data:** [voltron-lions/skills/skills-registry/actions-map.json](voltron-lions/skills/skills-registry/actions-map.json) — each entry maps an action type → responsible Lion → required tools/skills → status

| Action type | Lion | Status |
|-------------|------|--------|
| code-implementation, file-scaffolding | Red | covered |
| api-integration, database-schema, data-pipeline | Blue | covered |
| codebase-recon, web-research | Green | covered |
| test-authoring, security-review | Yellow | covered |
| skill-scaffolding | Voltron Main | covered (asks approval first) |
| remote-pod-action, remote-pod-status, raoara-workflow | Voltron Main | conditional — needs an actionboard.ai pod connection |

To register a new action type, edit `actions-map.json`, bump the plugin version, and reinstall.

## Connect to an actionboard.ai Cloud AI Pod (remote missions)

Voltron Lions can dispatch actions to a remote **Actionboard AI pod** — cloud actionlists, RAOARA six-phase workflows, and Voltron Desktop bridging — through the Actionboard AI plugin.

**What you need:**

1. An account at [actionboard.ai](https://actionboard.ai) with an AI pod (your pod URL and credentials come from the Actionboard AI dashboard).
2. The **Actionboard AI plugin** installed in Claude Code / Cowork (it provides `connect-pod`, `list-actions`, `execute-action`, `voltron-status`, `list-boards`, and `raoara`).

**Then just ask:**

```
connect to my actionboard pod
```

Voltron uses its `actionboard-pod-connect` skill to walk the connection flow: connect → verify health → list boards → list actions. After that, remote pod actions can appear in mission plans like any other Lion assignment — still gated by your go/no-go, and Voltron re-confirms before any irreversible pod action (deploys, sends, deletes).

**Security:** your pod credentials live in the Actionboard AI plugin's own configuration. Voltron Lions never stores, logs, or echoes them.

## Tips

- For large codebases, run `/graphify` first so Green Lion's recon is graph-backed.
- If Voltron Main flags `needs-new-skill` gaps, it will ask before scaffolding via `skill-creator`. New skills become available on the next session.
- Ask "what can the Lions do?" any time — the skills registry answers with the current actions map.

## Design

The design rationale and implementation plan are preserved in the [commit history](https://github.com/Cloudscockpit/voltron-lions-claudecode/commits/main) — see commits `f547fa4` (design spec) and `81adfc2` (implementation plan).
