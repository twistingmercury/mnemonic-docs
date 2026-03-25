# Setting Up Mnemonic for Your Team

This guide walks through a complete, production-ready setup of Mnemonic for a team. It assumes you've read the Quick Start or want a thorough foundation from scratch.

## Prerequisites

Before starting, ensure you have:

- **Docker and Docker Compose** installed and running
- **An OpenAI API key** with access to the text-embedding-3-large model (https://platform.openai.com/api-keys)
- **curl, jq, and yq** (mikefarah/yq) for API interaction and pattern loading scripts
- **Git** to clone repositories
- The mnemonic-docs and mnemonic-patterns repositories (see next section)

Verify Docker is running:

```bash
docker --version
docker compose version
```

Verify you have required tools:

```bash
curl --version
jq --version
yq --version
```

If any are missing, install them using your system's package manager.

## Step 1: Clone the Repositories

You need two repositories to run Mnemonic: the orchestration repo (docker-compose.yaml) and the pattern library.

```bash
git clone https://github.com/twistingmercury/mnemonic-docs.git
git clone https://github.com/twistingmercury/mnemonic-patterns.git
```

The mnemonic-docs repo contains the docker-compose.yaml that starts all seven services. The mnemonic-patterns repo contains validation and loading scripts, plus a library of curated patterns your team can build on.

You don't need to clone the individual service repos (mnemonic-mcp, mnemonic-api, mnemonic-enricher, mnemonic-admin, mnemonic-dbs) — Docker Compose pulls pre-built images automatically.

## Step 2: Configure the Environment

Mnemonic services use layered configuration: built-in defaults, then config files, then environment variables. For team setup, you only need to set one environment variable.

Export your OpenAI API key:

```bash
export MNEMONIC_OPENAI_API_KEY="sk-your-key-here"
```

Replace `sk-your-key-here` with your actual API key.

All other configuration uses development defaults in the docker-compose.yaml:

| Service | Default Port | Default Credentials |
|---------|--------------|-------------------|
| PostgreSQL | 5432 | mnemonic / mnemonic_dev |
| Neo4j | 7474 (HTTP), 7687 (Bolt) | neo4j / mnemonic_dev |
| RabbitMQ | 5672 (AMQP), 15672 (Management) | mnemonic / mnemonic_dev |
| MCP Server | 8081 | — (read-only, no auth) |
| Admin API | 8080 | — (no auth in Phase 1) |
| Enricher | 8082 | — (internal use only) |
| Admin UI | 3000 | — (no auth in Phase 1) |

These defaults are appropriate for local and trusted team networks. For production deployments, override credentials by setting environment variables before running `docker compose up` (see the docker-compose.yaml for the full list of `MNEMONIC_*` variables).

## Step 3: Start the Stack

Navigate to the mnemonic-docs directory and bring up all services:

```bash
cd mnemonic-docs
docker compose up -d
```

Docker Compose starts seven containers:

- PostgreSQL — pattern storage and vector embeddings
- Neo4j — pattern relationship graph
- RabbitMQ — asynchronous job queue
- MCP Server — MCP tools for Claude Code (port 8081)
- Admin API — REST API for pattern management (port 8080)
- Enricher — embedding generation worker
- Admin UI — web interface for pattern browsing and management (port 3000)

Wait about 30 seconds for all services to become healthy. Check the status:

```bash
docker compose ps
```

All containers should show `Up` and (after a moment) `healthy` in the Status column.

Verify the API and MCP server are responding:

```bash
curl -s http://localhost:8080/health | jq .
curl -s http://localhost:8081/health | jq .
```

Both should return HTTP 200 with a status object. If either returns an error, check logs:

```bash
docker compose logs dev_api
docker compose logs dev_mcp
```

## Step 4: Load the Pattern Library

The mnemonic-patterns repo includes a shell script that validates all patterns against the schema and loads them into Mnemonic via the REST API.

Navigate to the mnemonic-patterns directory:

```bash
cd ../mnemonic-patterns
```

Run the load script:

```bash
./install/load.sh
```

This script:

1. Validates all pattern files in the patterns/ directory (frontmatter, required sections, decorators)
2. POSTs each valid pattern to the Admin API at `http://localhost:8080/v1/api/patterns`
3. Reports results: loaded, skipped (409 conflict — pattern already exists), and failed
4. Exits with status 0 if all patterns loaded successfully

Example output:

```
Loading patterns from ./patterns/
  ✓ go-error-handling
  ✓ go-testing-strategies
  ✓ database-migrations
  ✗ api-versioning (failed: 400 Bad Request)
  ⊘ git-workflow (skipped: 409 Conflict)

Loaded: 2, Skipped: 1, Failed: 1
```

If you want to point the script at a different Mnemonic server (e.g., a shared team instance), pass the server URL:

```bash
./install/load.sh --server http://your-server:8080
```

The script uses curl, yq, and jq to interact with the API. If loading fails, ensure:
- The Admin API is running and healthy (`curl -s http://localhost:8080/health`)
- Your OpenAI API key is set and valid (the enricher will fail silently if the key is wrong)
- Each pattern's frontmatter is valid YAML and contains required fields (name, description, entity_type, language, domain)

## Step 5: Verify Enrichment

After patterns are loaded, the enricher worker generates semantic embeddings asynchronously. This process typically takes seconds to minutes, depending on the number of patterns and OpenAI API rate limits.

Check enrichment status:

```bash
curl -s http://localhost:8080/v1/api/patterns?limit=10 | jq '.data[] | {name, enrichment_status}'
```

Look for output like:

```json
{
  "name": "go-error-handling",
  "enrichment_status": "enriched"
}
```

Patterns should eventually show `"enriched"` (not `"pending"` or `"failed"`). If a pattern stays in `"pending"` for more than a few minutes, check the enricher logs:

```bash
docker compose logs dev_enricher -f
```

Common causes of enrichment failure:

- Invalid OpenAI API key (check the MNEMONIC_OPENAI_API_KEY environment variable)
- OpenAI API rate limit reached (wait and retry)
- Pattern content is malformed or too large (check the Admin API logs for details)

Once all patterns show `"enriched"`, your knowledge base is ready for search.

## Step 6: Configure Developers' AI Coding Assistants

Each team member configures their AI assistant to connect to Mnemonic as an MCP server. The MCP interface is read-only — developers can search patterns but cannot create or modify them through the assistant.

### Claude Code Configuration

Claude Code supports two levels of MCP configuration: project-level and user-level.

**Project-level (recommended for teams):**

Create a `.mcp.json` file in the project root (the same directory where you cloned mnemonic-docs). This file is shared with the team:

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

**User-level (personal setup):**

If you prefer not to commit .mcp.json to your repo, add the same configuration to Claude Code's user settings. The exact location varies by setup; check Claude Code's documentation or settings interface.

After adding the configuration, **restart Claude Code** to pick up the new MCP server. You should see "mnemonic" appear in the available MCP servers.

**For a remote team instance:**

If Mnemonic is running on a shared server instead of localhost, replace `http://localhost:8081` with the server's address and port:

```json
{
  "mcpServers": {
    "mnemonic": {
      "type": "streamable-http",
      "url": "http://mnemonic.team.example.com:8081/mcp"
    }
  }
}
```

### Other MCP-Compatible Assistants

If your team uses a different AI assistant that supports MCP (e.g., Anthropic's Claude Web, or third-party tools), follow these steps:

1. Locate the assistant's MCP configuration file (check documentation for exact path)
2. Add a new MCP server entry with these settings:
   - **Name:** mnemonic
   - **Transport:** HTTP Streamable
   - **URL:** http://localhost:8081/mcp (or your server's address)
3. Restart the assistant

## Step 7: Test the Integration

Verify that your developers can search Mnemonic from their AI assistant.

In Claude Code or your chosen assistant, ask a natural-language question that matches one of your loaded patterns:

> Search Mnemonic for our error handling pattern in Go.

or

> Find information about database migration strategies in Mnemonic.

The assistant should invoke the `search_patterns` MCP tool and return relevant patterns from your knowledge base. If the search works, your team is ready to use Mnemonic.

## Operational Basics

### Health Checks

Monitor the health of the stack:

```bash
curl -s http://localhost:8080/health | jq .
curl -s http://localhost:8081/health | jq .
```

Both endpoints return HTTP 200 when the service is healthy.

### Logs

All services emit structured JSON logs via zerolog. View logs for a specific service:

```bash
docker compose logs -f dev_api
docker compose logs -f dev_mcp
docker compose logs -f dev_enricher
```

View logs for all services:

```bash
docker compose logs -f
```

### Database Tools

**Neo4j Browser** (pattern relationship graph):

Open http://localhost:7474 in a web browser. Log in with credentials: neo4j / mnemonic_dev.

From the Neo4j browser, you can explore the pattern relationship graph, run Cypher queries, and troubleshoot pattern connections.

**RabbitMQ Management UI** (job queue):

Open http://localhost:15672 in a web browser. Log in with credentials: mnemonic / mnemonic_dev.

Use the management UI to inspect queues, monitor job throughput, and troubleshoot queue issues. The main queue is named `mnemonic.enrichment`.

**Admin UI** (web-based pattern browser):

Open http://localhost:3000 in a web browser. No login required in Phase 1.

The Admin UI provides a web interface for searching patterns, viewing pattern details, and managing the pattern library. This is useful for admins and pattern authors.

### Stopping and Restarting

**Stop all services:**

```bash
docker compose down
```

**Stop all services and remove data:**

```bash
docker compose down -v
```

The `-v` flag removes Docker volumes, which deletes all data in PostgreSQL, Neo4j, and RabbitMQ. Use this to reset your local environment. Do not use `-v` in production unless you intend to delete all data.

**Restart the stack:**

```bash
docker compose up -d
```

Data is preserved between restarts unless you used `down -v`.

### Troubleshooting

**A service won't start:**

Check the logs for that service:

```bash
docker compose logs dev_api  # or dev_mcp, dev_enricher, etc.
```

Common causes:
- Port already in use (change the port in docker-compose.yaml)
- Insufficient disk space (free up space and retry)
- Missing or invalid OpenAI API key (check MNEMONIC_OPENAI_API_KEY)

**Patterns load but enrichment fails:**

Check the enricher logs:

```bash
docker compose logs dev_enricher
```

Common causes:
- Invalid OpenAI API key
- OpenAI API rate limit (wait and retry)
- Pattern content exceeds size limits (shorten the pattern or split it)

**MCP server is not responding:**

Verify the MCP server is running:

```bash
docker compose ps dev_mcp
curl -s http://localhost:8081/health | jq .
```

If the container is running but the health check fails, check logs:

```bash
docker compose logs dev_mcp
```

**Search results are empty:**

Verify patterns are enriched:

```bash
curl -s http://localhost:8080/v1/api/patterns?limit=5 | jq '.data[] | {name, enrichment_status}'
```

All patterns should show `"enriched"`. If they show `"pending"`, wait a moment and retry. If they show `"failed"`, check the enricher logs.

## Security Considerations

**Phase 1 limitations:**

- No authentication or authorization — Mnemonic runs without credentials checks. The MCP interface is read-only, and the REST API is open. This is appropriate only for trusted local networks or team-internal deployments.
- No audit trail — there is no record of who accessed or modified patterns.
- No encryption in transit — use Mnemonic only within a trusted network (local machine or corporate LAN).

For production deployments:

- Run Mnemonic only on internal networks, not exposed to the internet
- Restrict database and queue access to the Docker network only
- Use strong credentials for RabbitMQ, PostgreSQL, and Neo4j (override defaults in docker-compose.yaml)
- Monitor logs regularly for unusual activity
- Plan for authentication and authorization in Phase 2

## Customization

### Changing Ports

If a default port conflicts with another service, edit the docker-compose.yaml file and change the port mapping:

```yaml
dev_api:
  ports:
    - "8080:8080"  # Change first 8080 to your desired port
```

Then restart:

```bash
docker compose down
docker compose up -d
```

### Using a Different OpenAI Embedding Model

The default is text-embedding-3-large. To use a different model, set the environment variable before starting:

```bash
export MNEMONIC_OPENAI_EMBEDDING_MODEL="text-embedding-3-large"
docker compose up -d
```

Check OpenAI's documentation for supported models.

### Persisting Data Across Restarts

By default, data persists in Docker volumes. To verify volumes are not cleaned up on shutdown, always use `down` without `-v`:

```bash
docker compose down  # Keeps volumes and data
```

To permanently delete all data and start fresh:

```bash
docker compose down -v  # Removes volumes and data
docker compose up -d
```

### Scaling for a Larger Team

For larger teams or more complex use cases:

- Deploy Mnemonic on a shared server (e.g., a team VM or Kubernetes cluster)
- Adjust resource limits in docker-compose.yaml (memory, CPU)
- Configure external PostgreSQL and Neo4j databases instead of using the Docker Compose ones
- Set up a reverse proxy (nginx, Caddy) in front of the services for load balancing and SSL termination
- Use environment variables to override default credentials

See the project CLAUDE.md file for details on the service architecture and configuration options.

## What's Next

**[Authoring Patterns](05-authoring-patterns.md)** — Learn how to write patterns that work well with semantic search. Covers frontmatter conventions, section structure, and best practices for clarity.

**[Managing Patterns](06-managing-patterns.md)** — Keep your knowledge base organized and up-to-date. Strategies for bulk loading, versioning, deprecation, and maintaining pattern quality.

**[Architecture Documentation](../architecture/)** — Deep dives into system design, data architecture, deployment architecture, and observability.

---

Copyright (c) 2025-2026 Jeremy K. Johnson. All rights reserved.
