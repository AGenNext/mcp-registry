# MetaMCP — MCP Manager for AGenNext/mcp-registry

> **Role in this registry:** MetaMCP is the designated **lifecycle manager and control plane** for all MCP servers listed here. It aggregates, authenticates, versions, and exposes every server in a unified gateway.

## What is MetaMCP?

[MetaMCP](https://github.com/metatool-ai/metamcp) is a MCP Aggregator, Orchestrator, Middleware, and Gateway packaged in a single Docker container. Think of it as the "runtime layer" that sits in front of all servers in this registry:

```
MCP Client (Claude, Cursor, etc.)
         │
         ▼
   [ MetaMCP Gateway ]  ← manages auth, versions, rate limits, namespaces
         │
    ┌────┴────────────────────────────────────┐
    │                                         │
    ▼                                         ▼
n8n-mcp-server               slack-mcp-server   ... (58+ servers)
```

## Registry Manager Responsibilities

| Responsibility | How MetaMCP handles it |
|---|---|
| **Version management** | Each `registry-entry.json` declares a `version`; MetaMCP tracks which version is running vs. available |
| **Authentication** | Per-server env secrets stored in MetaMCP; API-key or OAuth issued to clients per endpoint |
| **New MCP listing** | Add `registry-entry.json` → open MetaMCP UI → create namespace → assign server → publish endpoint |
| **Enable/disable servers** | Toggle servers and individual tools per namespace in the UI |
| **Rate limiting** | Endpoint-level and per-user rate limits configurable per server |
| **Middleware** | Tool filtering, observability, and custom middleware applied at namespace level |
| **Multi-tenancy** | Public/private scopes; per-user API keys; separate UI and SSO registration controls |

## Quick Start

```bash
git clone https://github.com/metatool-ai/metamcp.git
cd metamcp
cp example.env .env
docker compose up -d
# UI:  http://localhost:12005
# MCP: http://localhost:12008/metamcp/<ENDPOINT_NAME>/sse
```

## Connecting a Registry Server to MetaMCP

1. **Add the server** in MetaMCP UI → *MCP Servers* → *New Server* using the config from its `registry-entry.json`.
2. **Create or open a Namespace** and assign the server.
3. **Create an Endpoint** for the namespace, choose auth method (API key or OAuth).
4. **Connect your MCP client** using the endpoint URL.

Example (adding `n8n-mcp-server` from this registry):
```json
{
  "n8n": {
    "type": "STDIO",
    "command": "docker",
    "args": ["run", "--rm", "-e", "N8N_API_URL=${N8N_API_URL}", "-e", "N8N_API_KEY=${N8N_API_KEY}", "agentnxt/n8n-mcp-server"]
  }
}
```

## Version Management Workflow

```
registry-entry.json (version field)
        │
        ▼
MetaMCP Version Tracker
  ├── running: 1.2.0
  ├── available: 1.3.0  ← registry bumped this
  └── action: update button in UI → pulls new image → restarts server
```

When a server in this registry releases a new version:
1. Update `version` in `registry-entry.json` and open a PR.
2. MetaMCP operators see an "update available" badge in their dashboard.
3. One-click update pulls the new image and restarts the server in-place.

## Authentication Modes

| Mode | Use case |
|---|---|
| `api-key` (Bearer header) | Programmatic clients, CI/CD |
| `oauth-mcp-spec-2025-06-18` | Standard MCP OAuth flow |
| `oidc` | Enterprise SSO (Auth0, Azure AD, Keycloak, Okta, Google) |

## Adding a New MCP to the Registry

1. Create a new folder: `<your-server-name>/`
2. Add `registry-entry.json` following the schema in `schema/`
3. Add `README.md` with server documentation
4. Open a PR — after merge, the server is discoverable in MetaMCP UI

## Transport Support

- **SSE** — `http://localhost:12008/metamcp/<NAME>/sse`
- **Streamable HTTP** — `http://localhost:12008/metamcp/<NAME>/mcp`
- **OpenAPI** — `http://localhost:12008/metamcp/<NAME>/openapi` (for Open WebUI etc.)

## Links

- Repository: https://github.com/metatool-ai/metamcp
- Documentation: https://docs.metamcp.com
- GHCR image: `ghcr.io/metatool-ai/metamcp:latest`
- License: MIT
