# Voltron-Lions (pi package)

Mission-style multi-agent orchestration for the **pi** coding agent. One prompt forms Voltron — a commander (Black Lion) who plans, delegates, gates, and reports while playing four specialist Lion roles.

## Install

```bash
pi install npm:voltron-lions
# or from git:
pi install git:github.com/Cloudscockpit/voltron-lions-claudecode
```

## Use

Run the `start-mission` prompt with an objective:

```
start voltron mission <objective>
```

Voltron Main will:

1. Draft a **Mission Brief** (objective, scope, success criteria, constraints)
2. Send **Green Lion** on read-only recon
3. Generate **Lion Assignments** with strict file-ownership boundaries
4. Perform **Skills Gap** analysis against the shipped skills registry & actions map
5. Build a **Risk Register** (technical / scope / integration / data-loss, severity L/M/H)
6. Wait for your **go / no-go**, then execute as Red / Blue / Yellow Lions

## What ships in this package

- **Prompts:** `start-mission`
- **Skills:** `skills-registry`, `actionboard-onboarding`, `actionboard-pod-connect`, `browser-actions`, `cowork-mcp-connect`

> Note: pi loads the `skills/` and `prompts/` directories. The `agents/` and `commands/` folders target Claude Code / Cowork and are ignored by pi (this is a dual-target plugin).

## License

MIT © Cloudscockpit
