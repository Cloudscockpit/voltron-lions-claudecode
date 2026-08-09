# KB — Local Voltron MCP Endpoints

Reference for connecting Claude Code / Claude Cowork to the locally running Voltron backend (the `voltron-backend` docker-compose stack).

## Primary endpoint — ActionList Agent (FastMCP)

| Field | Value |
|-------|-------|
| Name | `ActionList Agent` |
| Transport | Streamable HTTP (FastMCP `http_app`) |
| URL from the host machine | `http://localhost:8200/mcp` |
| Provides | AI-powered action list generation and management (create, list, update actionlists) |
| Served by | `actionlist-service` (docker-compose port mapping `8200:8200`, MCP app mounted at `/mcp`) |

This is the endpoint meant when docs say "the local Voltron MCP endpoint."

## Gateway MCP proxy (REST, not an MCP transport)

The gateway (`http://localhost:8000`) exposes `/api/v1` routes that **proxy** a registry of internal MCP servers. These are for the Castle UI — do NOT point an MCP client at the gateway; connect to the individual servers below if they are port-mapped, or use the ActionList Agent endpoint above.

| Registry name | Internal host:port | Purpose |
|---------------|--------------------|---------|
| `actionboard_core` | `mcp-actionboard-core:3500` | Calendar, action lists, knowledge base, skills |
| `student` | `mcp-student:2000` | Academic research, plagiarism detection |
| `multimedia` | `mcp-multimedia:2200` | Audio transcription, video summary, image gen |
| `twitter` | `mcp-twitter:2500` | Real-time X/Twitter search |
| `docstrange` | `mcp-docstrange:2600` | PDF/image document extraction |

Internal hostnames resolve only inside the docker network. From the host, a server is reachable only if docker-compose maps its port.

## Prerequisites for any local connection

1. The voltron-backend stack is running: `docker compose up` in the `voltron-backend` repo.
2. Verify the MCP endpoint answers before configuring clients:
   ```bash
   curl -s -o /dev/null -w "%{http_code}" http://localhost:8200/mcp
   ```
   Any HTTP response (including 405/406 on a bare GET) means the server is up; connection-refused means the stack isn't running or the port mapping changed.

## Security notes

- These endpoints are localhost-only by default. Do not expose them to the network without auth.
- No credentials are needed for the local endpoint; anything requiring pod credentials goes through the Actionboard AI plugin instead (see the `actionboard-pod-connect` skill).
