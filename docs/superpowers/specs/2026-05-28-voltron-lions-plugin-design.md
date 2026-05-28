# Voltron-Lions Plugin — Design Spec

**Date:** 2026-05-28
**Status:** Draft for user review
**Owner:** jarjis@gmail.com

## Summary

A Claude Code plugin that introduces a single mission-orchestration command, `/start-mission`, backed by five agents themed after the classic Voltron lions. Voltron Main (Black Lion) decomposes a mission into a Mission Brief, Lion Assignments, Skills Gap analysis, and Risk Register, then dispatches Red/Blue/Green/Yellow lions to execute after user approval.

## Goals

- Give the user a single command that turns an objective into a structured execution plan with clear ownership and risk.
- Make multi-agent coordination feel cohesive (themed, opinionated) rather than ad-hoc.
- Surface capability gaps explicitly, and let Voltron Main scaffold new skills via the `skill-creator` skill when the user approves.
- Leverage existing `graphify-out/` indexes for fast, high-fidelity codebase recon when available.

## Non-Goals (v1)

- No Actionboard pod integration. Voltron-Lions is a pure Claude Code plugin in v1.
- No persistent mission log or database — mission state lives in the conversation.
- No hooks, no scheduled tasks, no background services.
- No direct Lion-to-Lion messaging — Voltron Main is the sole relay.
- No `/end-mission` or `/mission-status` commands in v1 (can be added later).

## Architecture

One slash command (`/start-mission`) launches Voltron Main. Voltron Main is the only agent the user converses with; it dispatches the four specialist Lions via the `Agent` tool, collects their results, and renders a single mission report. There is no plugin runtime — all agents and commands are static markdown files loaded by Claude Code's plugin loader.

```
User
  │
  ▼
/start-mission <objective>
  │
  ▼
Voltron Main (Black Lion, opus)
  │
  ├──► Green Lion  ── recon ──┐
  │                            │
  ├──► Red Lion    ── build ───┤
  │                            ├──► aggregated mission report
  ├──► Blue Lion   ── data ────┤
  │                            │
  └──► Yellow Lion ── verify ──┘
```

## The Roster

| Lion | Role | Model | Tools |
|------|------|-------|-------|
| **Black — Voltron Main** | Mission commander; produces brief, assignments, skills-gap, risks; relays between Lions | `opus` | `Read, Glob, Grep, Bash, Agent, Skill` |
| **Red Lion** | Rapid execution / builder — writes code, ships features | `sonnet` | `Read, Write, Edit, Bash, Glob, Grep` |
| **Blue Lion** | Data & integrations — APIs, persistence, plumbing | `sonnet` | `Read, Write, Edit, Bash, Glob, Grep, WebFetch` |
| **Green Lion** | Recon, research, codebase analysis; graphify-aware | `sonnet` | `Read, Glob, Grep, WebFetch, WebSearch, Skill` |
| **Yellow Lion** | Quality & defense — tests, security review, validation | `sonnet` | `Read, Edit, Bash, Glob, Grep` |

## Mission Flow

`/start-mission <objective>` launches Voltron Main, which executes the following sequence:

### 1. Mission Brief
Voltron Main parses the objective and produces a structured brief with these sections:
- **Objective** — single sentence
- **Scope** — what is in
- **Out of Scope** — what is explicitly excluded
- **Success Criteria** — observable, testable conditions
- **Constraints** — time, tools, codebase, user-imposed

### 2. Recon (Green Lion)
Voltron Main dispatches Green Lion. Green Lion runs **tiered recon**:

1. **Check for `./graphify-out/` in the project root only.** (Do not walk up parent dirs.)
2. **If found and fresh** → read `GRAPH_REPORT.md` for the code map, query `graph.json` for specific entities the mission needs.
3. **If found but stale** (manifest.json older than recent `git log` activity, e.g. >20 commits since indexing or >7 days behind HEAD) → do NOT use the graph. Recommend re-running `/graphify`, fall back to filesystem recon.
4. **If absent** → use `Glob` + `Grep` + targeted `Read`. In the recon report, suggest running `/graphify` for higher-fidelity recon next time.

Green Lion's report back to Main always includes a header:

```
Recon mode: graph-backed | filesystem-only
Graph source: <path-to-graphify-out> | (none)
Graph freshness: fresh | stale | n/a
Suggestion: <empty> | run /graphify for ~10x faster recon next time | re-run /graphify (index is stale)
```

Voltron Main surfaces the suggestion (if any) in the Mission Brief output as a single advisory line — the mission proceeds either way.

### 3. Lion Assignments
Based on the recon report, Voltron Main assigns concrete tasks to Red, Blue, and Yellow Lions with **file-ownership boundaries** — no two Lions may write the same file. Each assignment includes:
- Task description
- Files owned (exclusive write access)
- Dependencies on other Lions' outputs
- Acceptance criteria

### 4. Skills Gap Analysis
For each capability the mission needs, Voltron Main classifies it as one of:
- `covered` — already available in built-in tools or an installed skill (cite the skill name)
- `use-existing-skill <name>` — exact skill identified, will be invoked by a Lion
- `needs-new-skill` — no existing coverage; Main proposes a new skill scaffold

If any gap is `needs-new-skill`, Main asks the user for explicit approval before invoking the `skill-creator` skill to scaffold the new skill into `~/.claude/skills/`. New skills do NOT become available mid-mission — they take effect on the next session or after `/reload`. Main notes this in its output.

### 5. Risk Register
A table of mission risks, categorized:

| Category | Risk | Severity | Mitigation |
|----------|------|----------|------------|
| Technical | ... | L/M/H | ... |
| Scope | ... | L/M/H | ... |
| Integration | ... | L/M/H | ... |
| Data-loss | ... | L/M/H | ... |

### 6. Go/No-Go Gate
Voltron Main renders the full four-part report (Brief / Assignments / Skills Gaps / Risks) and **waits for user approval** before any specialist Lion begins execution.

## Skill Creation Capability

When Voltron Main flags a `needs-new-skill` gap and the user approves:
1. Main invokes the `skill-creator` skill.
2. `skill-creator` walks Main through the standard skill-creation workflow (name, description, content, frontmatter).
3. The new skill file is written to `~/.claude/skills/<name>/SKILL.md`.
4. Main updates the gap status from `needs-new-skill` to `covered (pending reload)`.
5. Main re-presents the updated assignments table.

This works because Voltron Main has the `Skill` tool in its tool profile. New skills are static markdown files — once written, they're discoverable on next load.

## Plugin Layout

```
voltron-lions-claudecode/
├── .claude-plugin/
│   └── plugin.json           # plugin manifest
├── agents/
│   ├── voltron-main.md       # Black Lion (commander)
│   ├── red-lion.md
│   ├── blue-lion.md
│   ├── green-lion.md
│   └── yellow-lion.md
├── commands/
│   └── start-mission.md
├── docs/
│   └── superpowers/specs/    # this spec
└── README.md
```

## Component Specs

### `commands/start-mission.md`
- Frontmatter: `description`, `argument-hint: "<mission-objective>"`
- Body: invokes Voltron Main agent via the `Agent` tool, passing `$ARGUMENTS` as the mission objective.

### `agents/voltron-main.md`
- `model: opus`, `color: black`
- System prompt describes the 6-step mission flow above, with explicit instructions to STOP at the Go/No-Go gate.
- Includes mission report template (markdown) for consistent output.

### `agents/green-lion.md`
- `model: sonnet`, `color: green`
- Read-only + Skill. System prompt describes tiered recon (graphify-first), staleness check, fallback, and report header contract.

### `agents/red-lion.md`, `agents/blue-lion.md`, `agents/yellow-lion.md`
- `model: sonnet`, colors red/blue/yellow.
- Each system prompt narrowly scopes the Lion's role and reminds it to respect file-ownership boundaries assigned by Main.

### `.claude-plugin/plugin.json`
```json
{
  "name": "voltron-lions",
  "version": "0.1.0",
  "description": "Mission-style multi-agent orchestration: Voltron Main coordinates four specialist Lions to execute objectives with structured briefs, file-ownership boundaries, skills-gap detection, and risk registers.",
  "author": { "name": "jarjis", "email": "jarjis@gmail.com" },
  "license": "MIT"
}
```

## Error Handling

- **Green Lion finds no project root indicators (no `.git`, no recognizable manifest)** → Green Lion reports "uncertain root" and Main proceeds with filesystem recon scoped to CWD only.
- **A Lion's subagent invocation fails** → Main reports the failure in the mission report, marks that Lion's assignment as `blocked`, and asks the user how to proceed (retry, reassign, abort).
- **`skill-creator` invocation fails or user cancels mid-flow** → Main marks the gap as `needs-new-skill (deferred)` and continues with the mission, surfacing the deferred gap in the final report.
- **User says "no go" at the Go/No-Go gate** → Main asks what to change (scope, assignments, skills, risks) and re-runs the relevant phases.

## Testing Strategy

Since plugin agents are markdown files and not executable code, "testing" means:
1. **Manifest validation** — `plugin.json` parses, all `agents/` files exist, all referenced models are valid.
2. **Smoke mission** — install the plugin locally, run `/start-mission add a hello-world endpoint to a sample express app`, verify the four-part report renders with all sections populated.
3. **Graphify path** — run a smoke mission against a repo that has `graphify-out/` and verify Green Lion's report header says `graph-backed`.
4. **Stale graphify path** — touch `manifest.json` to an old timestamp, verify Green Lion reports `stale` and falls back.
5. **Skills-gap path** — give Main an objective requiring an obviously-missing capability; verify it proposes a new skill and waits for approval.

## Open Questions

None at spec time. Decisions captured:
- 5 classic Voltron lions (Black + Red/Blue/Green/Yellow).
- No Actionboard pod integration in v1.
- Project root only for `graphify-out/` lookup.
- Stale graphify → suggest re-run, fall back to filesystem.
- New-skill creation requires explicit user approval; skills available next session.
