# Managing Patterns

This guide covers loading, updating, deleting, and monitoring patterns in Mnemonic using the REST API and shell scripts. It assumes you have Mnemonic running (see [Setting Up for Your Team](04-setting-up-for-your-team.md)) and want to manage your knowledge base.

## Overview

The best way to load and maintain patterns is directly through the REST API and the shell scripts in the mnemonic-patterns repository. There is also a web UI (mnemonic-admin at port 3000) that is under active development but not yet fully fleshed out.

## Loading Patterns via the REST API

### Create a pattern

```bash
curl -X POST http://localhost:8080/v1/api/patterns \
  -H 'Content-Type: application/json' \
  -d '{
    "name": "my-pattern",
    "description": "What this pattern is about",
    "entity_type": "best-practice",
    "language": "go",
    "domain": "backend",
    "tags": ["go", "errors"],
    "agent_associations": [
      {"agent_name": "go-software-engineer", "relevance": 0.8}
    ],
    "content": "# My Pattern\n\n## Overview\n\n...\n\n[//]: pattern\n\n## Implementation\n\n..."
  }'
```

Returns: 202 Accepted with Location header. Pattern is queued for enrichment.

### Get a pattern by ID

```bash
curl -s http://localhost:8080/v1/api/patterns/{id} | jq .
```

### List patterns

```bash
curl -s 'http://localhost:8080/v1/api/patterns?limit=20' | jq .
```

Supports query parameters:

- `limit` (1-100, default 20)
- `cursor` (opaque pagination cursor)
- `tags` (comma-separated, e.g. `go,best-practices`)
- `search` (full-text search on name and description)
- `language` (filter by language)
- `domain` (filter by domain)

### Update a pattern

```bash
curl -X PUT http://localhost:8080/v1/api/patterns/{id} \
  -H 'Content-Type: application/json' \
  -d '{ ... updated fields ... }'
```

### Delete a pattern

```bash
curl -X DELETE http://localhost:8080/v1/api/patterns/{id}
```

### Semantic search via API

```bash
curl -s 'http://localhost:8080/v1/api/patterns/search?q=error+handling+in+Go&limit=5&threshold=0.7' | jq .
```

The `threshold` parameter controls the minimum cosine similarity for results. The default is `0.7` for the REST API. Note that the MCP `search_patterns` tool uses a different default threshold of `0.5`, configurable via `MNEMONIC_MCP_DEFAULT_SEARCH_THRESHOLD`.

## Bulk Loading with Shell Scripts

The mnemonic-patterns repository includes scripts for bulk validation and loading.

### Validate all patterns

```bash
cd mnemonic-patterns
./install/validate.sh --dir ./patterns
```

Checks all `.md` files against the pattern schema. Dependencies: yq, jq.

### Load all patterns

```bash
cd mnemonic-patterns
./install/load.sh
```

This script:

1. Runs validation first (aborts if any file fails)
2. Extracts frontmatter and body from each Markdown file
3. Builds a JSON payload including tags and agent associations (agents get default relevance of 0.8)
4. POSTs to `http://localhost:8080/v1/api/patterns`
5. Reports: loaded (202), skipped/already exists (409), or failed

### Custom server target

```bash
./install/load.sh --server http://your-server:8080
```

### Custom patterns directory

```bash
./install/load.sh --dir /path/to/your/patterns
```

Dependencies: curl, yq (mikefarah/yq), jq.

Environment variables:

- `MNEMONIC_BASE_URL` — API base URL (default: `http://localhost:8080`)
- `PATTERNS_DIR` — directory containing pattern Markdown files

## Monitoring Enrichment

After loading patterns, the enrichment worker processes them asynchronously.

### Check enrichment status for all patterns

```bash
curl -s http://localhost:8080/v1/api/patterns?limit=100 | jq '.data[] | {name, enrichment_status}'
```

Enrichment states:

- `pending` — queued, not yet processed
- `enriched` — embeddings generated, graph relationships created, searchable via MCP
- `failed` — enrichment failed (check enricher logs)

### Check enricher logs

```bash
docker compose logs -f dev_enricher
```

### Search availability

Only enriched patterns appear in MCP semantic search results. If a pattern isn't showing up in search, check its enrichment status first.

## Pattern Lifecycle

1. **Author** — write a Markdown pattern file following the schema (see [Authoring Patterns](05-authoring-patterns.md))
2. **Validate** — run `validate.sh` to check schema compliance
3. **Load** — POST via API or `load.sh` script
4. **Enrich** — automatic; generates embeddings and graph relationships
5. **Search** — pattern becomes available via MCP tools
6. **Update** — PUT via API to modify; re-enrichment is triggered automatically
7. **Delete** — DELETE via API to remove

## Error Handling

The API returns RFC 7807 Problem Details for errors:

- **400 Bad Request** — validation failures (includes field-level error details)
- **404 Not Found** — pattern doesn't exist
- **409 Conflict** — pattern with that name already exists (on create)
- **500 Internal Error** — server error

Every response includes an `X-Request-ID` header for correlation with logs.

## API Reference

For the complete API specification, see the OpenAPI document at `mnemonic-api/docs/openapi/mnemonic-v1.yaml`.

---

Copyright (c) 2025-2026 Jeremy K. Johnson. All rights reserved.
