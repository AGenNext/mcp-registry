# MCPHub by AgentNxt

**Production-ready MCP servers for your entire stack. 58 servers, 1000+ tools.**

MCPHub is a registry of Model Context Protocol servers maintained as a Node.js workspace monorepo. Each server lives under `packages/` and can be built, run, packaged, or deployed independently.

---

## Quick Start: Build from source

```bash
# Clone the repo
git clone https://github.com/AGenNext/mcp-registry.git
cd mcp-registry

# Install workspace dependencies
npm install

# Build every server
npm run build
```

If some packages are still scaffolded or do not yet expose a build script, use the safer workspace build:

```bash
npm run build:changed
```

---

## Build a single MCP server

```bash
# Example: build the n8n MCP server
npm run build --workspace=packages/n8n-mcp-server
```

Run the built server directly:

```bash
node packages/n8n-mcp-server/dist/index.js
```

Most packages compile TypeScript into `dist/index.js` and expose that file as the MCP server entry point.

---

## Quick Start: Docker

```bash
# Pull and run any published server from Docker Hub
docker run -d \
  -e SERVICE_URL="https://your-service.com" \
  -e SERVICE_API_KEY="your-api-key" \
  agentnxt/<server-name>

# Example: filesystem server
docker run -d \
  -e ALLOWED_DIRECTORIES="/data" \
  agentnxt/filesystem-mcp-server
```

All published images are available on Docker Hub: https://hub.docker.com/u/agentnxt

---

## Claude Desktop configuration

After building a server, add it to Claude Desktop or another MCP-compatible client.

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "node",
      "args": ["/absolute/path/to/mcp-registry/packages/filesystem-mcp-server/dist/index.js"],
      "env": {
        "ALLOWED_DIRECTORIES": "/data"
      }
    }
  }
}
```

Use an absolute path in `args` so the MCP client can find the server reliably.

---

## Available Servers

Each package lives under `packages/` and follows the standard layout:

```text
packages/<server-name>/
  src/
    index.ts        # MCP server entry point
    tools/          # Tool definitions
    api/            # API client
  package.json
  tsconfig.json
```

Common commands:

```bash
# Install dependencies
npm install

# Build every workspace
npm run build

# Build only workspaces with build scripts
npm run build:changed

# Build one workspace
npm run build --workspace=packages/<server-name>

# Run one built server
node packages/<server-name>/dist/index.js
```

---

## Environment variables

Each MCP server may require different credentials or service URLs. Check the specific package README or source before deployment.

Common examples:

```bash
SERVICE_URL="https://your-service.com"
SERVICE_API_KEY="your-api-key"
ALLOWED_DIRECTORIES="/data"
```

Never commit secrets into this repository. Use environment variables, Docker secrets, or your deployment platform secret store.

---

## Deployment model

This repository is not a single web app. It is a registry of independent MCP servers.

Recommended deployment flow:

1. Choose the target server under `packages/`.
2. Install dependencies from the repo root with `npm install`.
3. Build the selected package with `npm run build --workspace=packages/<server-name>`.
4. Run the compiled entry point from `dist/index.js`.
5. Provide required service credentials through environment variables.

---

## Unboxd Platform Scaffolds

Added scaffold servers for the first platform wave:

- `microcloud-mcp-server`
- `lxc-mcp-server`
- `ansible-mcp-server`
- `terraform-mcp-server`
- `gcloud-run-mcp-server`

See `docs/unboxd-platform-mcp-roadmap.md` for implementation direction.

---

Copyright 2026. All rights reserved AgentNxt. An Autonomyx Platform.
