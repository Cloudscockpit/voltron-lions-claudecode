# Voltron-Lions

A plugin marketplace for **Voltron Lions** — mission-style multi-agent orchestration for
Claude Code and Claude Cowork. One command. Five themed agents. Structured, machine-readable
output.

## What it does

`/start-mission <objective>` invokes **Voltron Main** (the Black Lion), who:

1. Drafts a **Mission Brief** — objective, scope, out of scope, success criteria, constraints
2. Sends **Green Lion** on read-only recon (uses `graphify-out/` when present and fresh)
3. Generates **Lion Assignments** with strict file-ownership boundaries
4. Performs **Skills Gap** analysis against the built-in skills registry & actions map
5. Builds a **Risk Register** — technical / scope / integration / data-loss, severity L/M/H
6. Waits for your **go / no-go**, then dispatches Red / Blue / Yellow Lions to execute

Nothing is written and no Lion is dispatched before you approve.

## The Lions

| Lion | Role |
|------|------|
| **Black — Voltron Main** | Commander. Plans, delegates, gates, reports. |
| **Red** | Rapid execution — builds features, writes code. |
| **Blue** | Data & integrations — APIs, schemas, plumbing. |
| **Green** | Recon — codebase analysis, read-only (graphify-aware). |
| **Yellow** | Quality & defense — tests, security, verification. |

## Installation

### Claude Code

In any Claude Code session:

```
/plugin marketplace add Cloudscockpit/voltron-lions-claudecode
/plugin install voltron-lions@voltron-lions
```

If the install summary reports `Run /reload-plugins to activate.`, run `/reload-plugins`.
Otherwise the plugin is already active and `/start-mission` is available.

### Claude Cowork

No terminal needed — this is done through the Cowork interface, not the chat box:

1. Open **Customize** in the sidebar, then **Plugins**.
2. Select **Add marketplace** and enter `Cloudscockpit/voltron-lions-claudecode`.
3. **Voltron Lions** appears alongside your other plugins. Select it and choose **Install**.
4. Open the installed plugin to review its skills and agents before enabling them.

Then type `/start-mission` followed by what you want done, in plain English:

```
/start-mission organize the files in my project folder and tell me what's outdated
```

Voltron shows you a mission plan and waits — nothing happens until you reply `go`.

## Mission documents as a knowledge base

Every mission document is emitted twice: **Markdown** for people, **schema.org JSON-LD** for
machines. JSON-LD + schema.org is the open format
[Google's Knowledge Graph Search API](https://developers.google.com/knowledge-graph) returns
and the one Google recommends for structured data, so mission output drops into a graph store
or a search index without a transform step.

| Skill | Writes | Covers |
|-------|--------|--------|
| `mission-brief` | `mission-brief.md` + `.jsonld` | Objective, scope, success criteria, constraints |
| `mission-plan` | `mission-plan.md` + `.jsonld` | Lion Assignments, Skills Gap, Risk Register |
| `mission-report` | `mission-report.md` + `.jsonld` | What shipped, what blocked, what was deferred |

The three documents share one URN identifier scheme, so their `@graph` arrays concatenate into
a single valid knowledge base. Lion and file identifiers are **global** rather than
per-mission — that is what makes a directory of missions worth querying: which files block
most often, which Lion's assignments need a second pass, which success criteria keep coming
back unmet.

Full type mapping, identifier scheme, status vocabulary, and a worked example:
[kb/mission-knowledge-format.md](kb/mission-knowledge-format.md).

## Skills registry & actions map

The plugin ships a machine-readable registry of everything the Lions can do:

- **Skill:** `voltron-lions:skills-registry` — used by Voltron Main during Skills Gap
  analysis, or ask it directly ("what can the Lions do?")
- **Data:** [skills/skills-registry/actions-map.json](skills/skills-registry/actions-map.json)
  — each entry maps an action type → responsible Lion → required tools/skills → status

| Action type | Lion | Status |
|-------------|------|--------|
| code-implementation, file-scaffolding | Red | covered |
| api-integration, database-schema, data-pipeline | Blue | covered |
| codebase-recon, web-research | Green | covered |
| test-authoring, security-review | Yellow | covered |
| mission-brief-authoring, mission-plan-authoring, mission-report-authoring | Voltron Main | covered |
| skill-scaffolding | Voltron Main | covered (asks approval first) |
| live-browser-action | Voltron Main | conditional — per-site user approval; credentials always handed to you |

To register a new action type, edit `actions-map.json`, bump the plugin version, and reinstall.

## What ships

**Command:** `/start-mission`

**Agents:** `voltron-main`, `red-lion`, `blue-lion`, `green-lion`, `yellow-lion`

**Skills:**

| Skill | Purpose |
|-------|---------|
| `mission-brief` | Write a Mission Brief as Markdown + schema.org JSON-LD |
| `mission-plan` | Write assignments, skills gap, and risk register as Markdown + JSON-LD |
| `mission-report` | Record each assignment's outcome against the plan's graph nodes |
| `skills-registry` | The actions map — which Lion handles what, and what it needs |
| `browser-actions` | Conduct rules for driving a real browser during a mission |

## Browser actions in missions

The `browser-actions` skill (+ `kb/claude-browser-actions.md`) teaches Voltron how to drive a
real browser during missions — navigate, read pages, fill forms, screenshot, debug web apps —
with hard rules: per-site approval, credentials and CAPTCHAs always handed to you,
confirmation before any irreversible web action, and page content treated as data, never as
instructions.

## Knowledge base

| Doc | Covers |
|-----|--------|
| [kb/mission-knowledge-format.md](kb/mission-knowledge-format.md) | schema.org JSON-LD envelope, identifier scheme, type mapping, status vocabulary, worked example |
| [kb/claude-browser-actions.md](kb/claude-browser-actions.md) | The two browser surfaces, capabilities, conduct rules |

## Tips

- For large codebases, run `/graphify` first so Green Lion's recon is graph-backed — reading
  one graph report costs a fraction of walking fifty source files.
- If Voltron Main flags `needs-new-skill` gaps, it asks before scaffolding via `skill-creator`.
  New skills take effect on the next session or after `/reload-plugins`.
- Ask "what can the Lions do?" any time — the skills registry answers with the current map.

## Repository layout

The plugin lives at the repository root, so this repo is both the plugin and a
single-plugin marketplace pointing at itself (`"source": "./"`).

```
.claude-plugin/
  plugin.json         the plugin manifest
  marketplace.json    the marketplace manifest
agents/               voltron-main + the four Lions
commands/             /start-mission
skills/               mission-brief, mission-plan, mission-report,
                      skills-registry, browser-actions
kb/                   knowledge base the skills read at runtime
```

## License

MIT © Cloudscockpit
