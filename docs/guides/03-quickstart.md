# Quick Start for Developers

Get your AI coding assistant connected to Mnemonic and searching team patterns in about 5 minutes.

## Prerequisites

Before you start, you need:

- Docker and Docker Compose installed and running
- An OpenAI API key (for embedding generation via text-embedding-3-large)
- Claude Code, Claude Web, or another MCP-compatible AI coding assistant
- curl (for loading a test pattern)

## Step 1: Start the Stack

Export your OpenAI API key and start all services:

```bash
export MNEMONIC_OPENAI_API_KEY="sk-your-key-here"
docker compose up -d
```

This starts seven services:

- PostgreSQL (port 5432) — pattern storage and embeddings
- Neo4j (port 7474/7687) — pattern relationship graph
- RabbitMQ (port 5672/15672) — job queue
- MCP server (port 8081) — reads patterns via MCP tools
- Admin API (port 8080) — REST API for pattern management
- Enricher (port 8082) — generates embeddings asynchronously
- Admin UI (port 3000) — web UI for pattern browsing

Wait about 30 seconds for all services to be healthy.

## Step 2: Verify the Stack is Healthy

Check that all containers are running:

```bash
docker compose ps
```

All services should show `Up`. Verify the API and MCP server are responding:

```bash
curl -s http://localhost:8080/health | jq .
curl -s http://localhost:8081/health | jq .
```

Both should return HTTP 200 with a status object.

## Step 3: Load a Test Pattern

POST a single test pattern to the API to populate the knowledge base:

```bash
curl -X POST http://localhost:8080/v1/api/patterns \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "go-error-handling",
    "description": "Best practices for error handling in Go",
    "entity_type": "best-practice",
    "language": "go",
    "domain": "backend",
    "tags": ["go", "errors", "best-practices"],
    "content": "# Go Error Handling\n\n## Overview\n\nThis pattern covers Go error handling best practices.\n\n[//]: pattern\n\n## Wrapping Errors\n\nUse fmt.Errorf with %w to wrap errors and preserve the error chain.\n\n```go\nif err != nil {\n    return fmt.Errorf(\"failed to connect: %w\", err)\n}\n```\n\nThis allows callers to use errors.Is and errors.As for inspection."
  }'
```

The API returns HTTP 202 Accepted. The pattern is now queued for enrichment.

## Step 4: Wait for Enrichment

Enrichment adds semantic embeddings to the pattern so it can be found via natural language search. This takes a few seconds. Check the enrichment status:

```bash
curl -s http://localhost:8080/v1/api/patterns | jq '.data[].enrichment_status'
```

Wait until you see `"enriched"` (not `"pending"` or `"failed"`).

## Step 5: Configure Your AI Coding Assistant

### Claude Code

Add the Mnemonic MCP server to your MCP settings. You can configure this in three ways:

**Option A: CLI command (quickest)**

Run this from any directory to add Mnemonic to your user-level MCP config:

```bash
claude mcp add --transport http mnemonic http://localhost:8081/mcp
```

To scope it to a single project instead, add `-s project`:

```bash
claude mcp add -s project --transport http mnemonic http://localhost:8081/mcp
```

**Option B: Project-level config file (recommended for team repos)**

Create or edit `.mcp.json` in the project root:

```json
{
  "mcpServers": {
    "mnemonic": {
      "type": "streamable-http",
      "url": "http://localhost:8081/mcp"
    }
  }
}
```

This file can be committed so the whole team gets the configuration automatically.

**Option C: User-level config file (all projects)**

Add the same JSON configuration to `~/.claude.json` under the `"mcpServers"` key, or use the CLI:

```bash
claude mcp add -s user --transport http mnemonic http://localhost:8081/mcp
```

Restart Claude Code after making changes to pick up the new MCP server.

### Other MCP-Compatible Assistants

If you're using a different AI assistant that supports MCP:

1. Locate its MCP configuration file (varies by tool)
2. Add the Mnemonic server endpoint: `http://localhost:8081/mcp`
3. Specify transport as HTTP Streamable
4. Restart the assistant

## Step 6: Verify It Works

In Claude Code or your AI assistant, ask a question that matches the test pattern you loaded:

> "Search Mnemonic for Go error handling best practices."

or

> "Search mnemonic for information about wrapping errors in Go."

The assistant should call the `search_patterns` MCP tool and return the pattern you loaded. If it does, you're ready to go.

## What's Next

**[Setting Up for Your Team](04-setting-up-for-your-team.md)** — Move beyond the test pattern. Deploy Mnemonic for your team's full pattern library, manage access, and set up automated pattern loading.

**[Authoring Patterns](05-authoring-patterns.md)** — Learn to write patterns that work well with semantic search. Covers frontmatter conventions, section structure, and tips for clarity.

**[Managing Patterns](06-managing-patterns.md)** — Keep your knowledge base organized and fresh. Strategies for bulk loading, updates, versioning, and deprecation.

---

Copyright (c) 2025-2026 Jeremy K. Johnson. All rights reserved.
