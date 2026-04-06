# Mnemonic Docs

> **Maturity Level**: Emerging - Documentation is in early rough draft; structure and content are actively being organized and will change frequently.
> **Version**: v0.2.0

---

## Table of Contents

- [Usage](#usage)
- [How It Works](#how-it-works)
- [Key Considerations](#key-considerations)
- [Development Considerations](#development-considerations)
- [Versioning](#versioning)

## Usage

This repository is the **single source of truth** for architecture, design, requirements, and reference documentation across the Mnemonic solution. Source repositories reference this repo rather than maintaining their own architecture docs.

For the full vision and problem statement, see [PURPOSE.md](PURPOSE.md).

### Quick Navigation

**First time here?** Read in this order:

1. [Concept](docs/concept.md) - understand the problem and solution
2. [System Architecture Overview](docs/architecture/system/README.md) - see how components fit together
3. [Requirements](docs/requirements.md) - learn what the system must do

## How It Works

Mnemonic is a pattern search system that integrates with Claude Code via MCP (Model Context Protocol). It stores curated engineering patterns in PostgreSQL (PGVector for embeddings) and Neo4j (concept relationships), enabling semantic search and graph traversal. Patterns are created via the admin API, enriched asynchronously via OpenAI embeddings, and served to Claude Code through MCP tools.

The solution spans five source repositories plus this documentation repo:

| Repository                                                              | Role                                                         |
| ----------------------------------------------------------------------- | ------------------------------------------------------------ |
| [mnemonic](https://github.com/twistingmercury/mnemonic)                | MCP service exposing pattern search tools to Claude Code     |
| [mnemonic-api](https://github.com/twistingmercury/mnemonic-api)        | REST API for pattern management (CRUD)                       |
| [mnemonic-enricher](https://github.com/twistingmercury/mnemonic-enricher) | Worker that enriches patterns with embeddings and metadata   |
| [mnemonic-dbs](https://github.com/twistingmercury/mnemonic-dbs)        | PostgreSQL + Neo4j schemas and Docker images                 |
| [mnemonic-admin](https://github.com/twistingmercury/mnemonic-admin)    | Web UI for pattern management and search                     |
| [mnemonic-docs](https://github.com/twistingmercury/mnemonic-docs)      | This repo - consolidated architecture and design docs        |

### Documentation Structure

| Directory                     | Contents                                                        |
| ----------------------------- | --------------------------------------------------------------- |
| `docs/architecture/system/`   | System-wide: decisions, security, communication, deployment     |
| `docs/architecture/mnemonic/` | MCP service: tool definitions and response patterns             |
| `docs/architecture/dbs/`      | Database schemas, migrations, deployment                        |
| `docs/architecture/admin/`    | Frontend stack, ADRs, screen map, use cases                     |
| `docs/design/`                | Implementation-level design shared across Go services           |
| `docs/reference/`             | Pattern file schema and glossary                                |
| `docs/diagrams/`              | System evolution diagrams (MVP 1-5) and flow diagrams           |
| `docs/guides/`                | User-facing guides: introduction, concepts, quickstart, authoring |

## Key Considerations

- **Early rough draft** - docs are still being organized and consolidated from the original monorepo split. Expect gaps, stale references, and structural changes.
- **Source of truth** - despite the draft state, this repo is the canonical location for architecture and design decisions. Source repos should not duplicate this content.
- **No authentication in Phase 1** - the current design assumes a trusted local environment.

## Development Considerations

### Quick Start

Clone this repo alongside the other Mnemonic repositories under a shared parent directory:

```
mnemonic/
  mnemonic/          # MCP server
  mnemonic-api/      # Admin REST API
  mnemonic-enricher/ # Enrichment worker
  mnemonic-dbs/      # Database images
  mnemonic-admin/    # Web UI
  mnemonic-docs/     # This repo
```

A `docker-compose.yaml` is included for running the local documentation preview stack.

### Contributing

When making changes in a source repository:

1. Update the relevant architecture document here to reflect the change
2. If the change affects multiple components, update system-level architecture
3. Create or update an ADR for significant technical decisions

### Versioning

This project follows [Semantic Versioning 2.0.0](https://semver.org/).

Version is determined from git tags:

```bash
git describe --tags --always
```

---

Copyright (c) 2025-2026 Jeremy K. Johnson. All rights reserved.
