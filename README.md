# Agent-Tools

Agent-Tools is the canonical tool catalog for the AGenNext platform.

This is a cloud marketplace-style catalog problem.

A tool is an invocable catalog object that can be discovered, governed, versioned, permissioned, reviewed, priced, installed, and used by agents, workflows, teams, and runtimes.

## Core model

```text
Tool
  = primary catalog object
  = invocable capability/product

Tool metadata includes:
  - capabilities
  - author
  - publisher
  - versions
  - provider offers
  - license
  - pricing
  - risk
  - approval requirements
  - invocation contracts
```

The primary catalog object is always the **Tool**.

Capability, author, publisher, version, and provider are metadata attached to the tool record.

## Role definitions

```text
Capability
  = standardized semantic ability implemented by the tool

Author
  = creator or maintainer of the tool

Publisher
  = party that lists, curates, verifies, or publishes the tool into the catalog

Version
  = version metadata and compatibility contract for the tool

Provider Offer
  = one access path for a specific tool version

Provider
  = organization, package, service, MCP server, CLI, SDK, HTTP API, marketplace, or runtime endpoint offering that version
```

## Marketplace rule

```text
One Tool
  can implement many Capabilities

One Tool
  can have many Versions

One Tool Version
  can have many Provider Offers

One Provider
  can offer many Tools

One open-source Tool
  can have many Providers

One MCP Server
  is a Provider type

One MCP Server
  can expose many Tools
```

## Example: tool with capability, author, publisher, version, and providers

```yaml
id: filesystem.read_file
type: tool
name: Read File

capabilities:
  - file.read

author:
  name: AGenNext
  type: Organization

publisher:
  name: AGenNext
  type: Organization

versions:
  - version: 1.0.0
    status: active
    contract:
      inputs:
        path: string
      outputs:
        content: string
    providers:
      - provider_id: filesystem-mcp-server
        provider_type: mcp_server
        protocol: mcp
        package_path: packages/filesystem-mcp-server
        offer_status: active

      - provider_id: internal-filesystem-http
        provider_type: http_api
        protocol: https
        offer_status: active

risk: medium
approval_required: false
```

## Example: open-source tool with multiple providers

```yaml
id: surrealkit
type: tool
name: SurrealKit

capabilities:
  - database.schema.generate
  - database.migration.run
  - database.seed.load

author:
  name: SurrealDB
  type: Organization

publisher:
  name: AGenNext
  type: Organization
  role: catalog_curator

versions:
  - version: 2.2.1
    status: supported
    providers:
      - provider_id: surrealkit-github-release
        provider_type: github_release
        install_strategy: pinned_binary
        version_policy: pinned

      - provider_id: surrealkit-npm
        provider_type: npm_package
        install_strategy: package_manager
        version_policy: pinned

      - provider_id: agennext-ci-wrapper
        provider_type: ci_wrapper
        install_strategy: workflow_script
        version_policy: pinned
```

## Repository responsibilities

Agent-Tools owns:

```text
- tool catalog entries
- capability references used by tools
- author and publisher metadata
- version metadata and compatibility contracts
- provider offer metadata
- external tool definitions
- MCP provider/package metadata where needed
- tool invocation contracts
- tool security, approval, pricing, and governance metadata
```

Agent-Tools does not own:

```text
- runtime execution of agents
- skill definitions
- agent blueprints
- graph schema
- grammar rules
- seed-data ownership
```

## Current folders

### `catalog/`

Canonical catalog entries for tools and external tools.

Example:

```text
catalog/surrealkit.tool.yaml
```

Each file should describe a tool first.

Capabilities, author, publisher, versions, and provider offers belong inside the tool entry as metadata.

### `packages/`

Existing folders under `packages/` are provider implementation packages.

Most current packages are MCP server implementations. They should be migrated conceptually from:

```text
MCP server = catalog item
```

to:

```text
Tool = catalog item
MCP server = provider offer metadata under a tool version
```

Recommended package structure:

```text
packages/<provider-name>/
  provider.yaml          # provider/package metadata
  tools/                 # tool catalog entries exposed through this provider
    <tool-id>.tool.yaml
  src/                   # provider implementation
  package.json
  tsconfig.json
```

## MCP provider model

An MCP server package should declare which tools it provides:

```yaml
provider_id: filesystem-mcp-server
provider_type: mcp_server
provides:
  - tool_id: filesystem.read_file
    version: 1.0.0
  - tool_id: filesystem.write_file
    version: 1.0.0
  - tool_id: filesystem.list_directory
    version: 1.0.0
```

Each exposed tool should have a catalog entry with provider offer metadata:

```yaml
id: filesystem.read_file
type: tool
capabilities:
  - file.read
versions:
  - version: 1.0.0
    providers:
      - provider_id: filesystem-mcp-server
        provider_type: mcp_server
        protocol: mcp
risk: medium
approval_required: false
```

## Capability standardization

Capabilities should be stable semantic identifiers.

Examples:

```text
file.read
file.write
filesystem.list
database.query
database.schema.generate
database.migration.run
database.seed.load
cloud.compute.create
cloud.storage.object.read
```

Capabilities allow the platform to compare tools, substitute providers, reason about risk, and match skills to tools.

## Build existing provider packages

Existing packages can still be built as Node.js workspaces:

```bash
npm install
npm run build
npm run build:changed
npm run build --workspace=packages/<provider-name>
```

Run a built MCP provider directly:

```bash
node packages/<provider-name>/dist/index.js
```

## Environment variables

Provider packages may require credentials or service URLs.

Common examples:

```bash
SERVICE_URL="https://your-service.com"
SERVICE_API_KEY="your-api-key"
ALLOWED_DIRECTORIES="/data"
```

Never commit secrets into this repository. Use environment variables, Docker secrets, or runtime secret stores.

## Migration rule for existing packages

Do not delete existing provider packages immediately.

Migrate them in place:

```text
1. Identify the tools exposed by each provider package.
2. Identify capabilities implemented by each tool.
3. Add one .tool.yaml catalog entry per tool.
4. Add capability, author, publisher, and version metadata under each tool.
5. Add provider offers under each version.
6. Add provider.yaml only for package/build/runtime metadata.
7. Mark tool risk, approval requirements, auth, pricing, license, and support metadata where known.
8. Keep implementation under src/.
9. Build and test provider package.
```

## Relationship to other repos

```text
Agent-Skills
  uses tools as executable capabilities

Agent-Graph
  maps tools, capabilities, authors, publishers, versions, providers, permissions, and invocations

Agent-Grammar
  validates tool metadata

Agent-Seed
  seeds default platform tool records

Agent-Review
  reviews high-risk tools before publication/use
```

## Rule

```text
Tool
  = primary catalog object

Capability
  = semantic ability metadata

Author
  = creator/maintainer metadata

Publisher
  = catalog publishing/curation metadata

Version
  = compatibility metadata

Provider Offer
  = access path metadata for a version
```
