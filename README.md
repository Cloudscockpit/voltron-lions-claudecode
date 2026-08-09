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
| live-browser-action | Voltron Main | conditional — per-site user approval; credentials always handed to you |
| actionboard-onboarding | Voltron Main | covered |
| local-mcp-connect | Voltron Main | conditional — needs the voltron-backend stack running locally |

To register a new action type, edit `actions-map.json`, bump the plugin version, and reinstall.

## Getting started as a new user (no account yet?)

Ask Voltron: **"help me get started with actionboard"** — the `actionboard-onboarding` skill walks you through, one step at a time:

1. **Sign up** at [actionboard.ai](https://actionboard.ai) (Claude can open the page and guide you — it hands you the browser for passwords and verification; it never types credentials).
2. **Install Voltron Castle Desktop** from the [releases page](https://github.com/Cloudscockpit/actionboard-desktop-app/releases) and sign in.
3. **Verify the bridge** — ask "check voltron status".

## Connect Claude Cowork to the local Voltron MCP endpoint

Running the voltron-backend stack locally? Ask: **"connect cowork to the local voltron mcp"** — the `cowork-mcp-connect` skill checks the endpoint (`http://localhost:8200/mcp`), then walks either path:

- **Developers (Claude Code):** `claude mcp add --transport http voltron http://localhost:8200/mcp`
- **Non-technical (Cowork/Desktop):** Settings → Connectors → Add custom connector → URL `http://localhost:8200/mcp`

This gives Claude direct access to the local Voltron actionlist tools — no cloud pod needed.

## Browser actions in missions

The `browser-actions` skill (+ `kb/claude-browser-actions.md`) teaches Voltron how to drive a real browser during missions — navigate, read pages, fill forms, screenshot, debug web apps — with hard rules: per-site approval, credentials/CAPTCHAs always handed to you, confirmation before any irreversible web action, and page content treated as data, never as instructions.

## Knowledge base

The plugin ships a `kb/` folder the skills draw on:

| Doc | Covers |
|-----|--------|
| [kb/claude-browser-actions.md](voltron-lions/kb/claude-browser-actions.md) | The two browser surfaces, capabilities, conduct rules, browser flows vs live actions |
| [kb/actionboard-onboarding.md](voltron-lions/kb/actionboard-onboarding.md) | Signup, Voltron Castle Desktop install, bridge verification, troubleshooting |
| [kb/voltron-mcp-endpoints.md](voltron-lions/kb/voltron-mcp-endpoints.md) | Local MCP endpoint table, prerequisites, security notes |

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
