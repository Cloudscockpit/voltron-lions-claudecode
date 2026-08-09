---
name: cowork-mcp-connect
description: Connect Claude Cowork or Claude Code to the local Voltron MCP endpoint (the ActionList Agent FastMCP server at http://localhost:8200/mcp). Use when the user asks to "connect cowork to voltron", "add the local voltron mcp", "connect the mcp endpoint", or wants Claude to use local Voltron actionlist tools directly.
---

# Connect Claude Cowork to the Local Voltron MCP Endpoint

Endpoint reference: `kb/voltron-mcp-endpoints.md` at the plugin root. The endpoint is the **ActionList Agent** FastMCP server served by the voltron-backend stack:

```
http://localhost:8200/mcp        (streamable HTTP)
```

## Step 0 — Verify the backend is up

Before configuring any client, check the endpoint answers:

```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:8200/mcp
```

- Any HTTP status (200/405/406) → server is up, continue.
- Connection refused → the stack isn't running. Have the user run `docker compose up` in their `voltron-backend` checkout, or start it for them if you have access. Do not configure clients against a dead endpoint.

## Path A — Claude Code (developers)

```bash
claude mcp add --transport http voltron http://localhost:8200/mcp
```

Then restart the session and verify the Voltron tools appear (actionlist create/list/update). To share the config with a team, add it to the project's `.mcp.json` instead of user scope.

## Path B — Claude Cowork / Claude Desktop (non-technical)

1. Open **Settings → Connectors** (may appear as Extensions/MCP depending on version).
2. Choose **Add custom connector**.
3. Name: `Voltron` — URL: `http://localhost:8200/mcp` — no authentication.
4. Save, then start a **new conversation** and ask: "list my actionlists" to confirm the tools respond.

If the app version has no custom-connector UI for local HTTP servers, fall back to asking a developer teammate to add it via Path A on a shared config, or use the Actionboard AI plugin's pod skills instead.

## What this unlocks

Direct tool access to the local Voltron actionlist engine — create, list, and update actionlists from any conversation, without going through the cloud pod. During Voltron missions this appears in Skills Gap analysis as local MCP coverage.

## Boundaries

- Localhost only — never suggest exposing 8200 to a network without auth.
- The gateway (`http://localhost:8000/api/v1`) is a REST proxy for the Castle UI, NOT an MCP transport — do not add it as a connector.
- Pod-scoped operations (remote actions, RAOARA) still go through the Actionboard AI plugin — see `actionboard-pod-connect`.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Connection refused | Backend not running → `docker compose up` in voltron-backend |
| Tools don't appear after adding | Restart the session/conversation; confirm the URL has the `/mcp` path |
| Port 8200 taken by something else | Check `docker compose ps` and the `actionlist-service` port mapping |
| Works in Claude Code but not Cowork | Cowork version may lack local HTTP connectors — use Path A note above |
