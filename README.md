# Mnemonic Documentation

This repository is the authoritative source of truth for the Mnemonic solution: consolidated architecture, design, requirements, and reference documentation.

## What is Mnemonic?

Mnemonic is a pattern search system that integrates with Claude Code via MCP (Model Context Protocol) tools. It enables developers to search for reusable patterns in code repositories, retrieve enriched pattern context, and apply those patterns to new code.

For the full vision and problem statement, see [Purpose](PURPOSE.md).

## The Solution: Five Repositories

The Mnemonic solution is split across five repositories. This repository (`mnemonic-docs`) consolidates their architecture and design.

| Repository          | Role                                                                         |
| ------------------- | ---------------------------------------------------------------------------- |
| `mnemonic`          | MCP service that exposes pattern search tools to Claude Code                 |
| `mnemonic-api`      | REST API for pattern management (CRUD operations)                            |
| `mnemonic-dbs`      | Database schemas (PostgreSQL, Neo4j) and Docker images for local development |
| `mnemonic-enricher` | Worker that enriches patterns with embeddings and semantic metadata          |
| `mnemonic-admin`    | Web UI for pattern management, search, and import                            |

Each source repository is read-only in terms of architecture and concept documentation — all updates to those materials come through `mnemonic-docs`.

## Documentation Structure

### Foundation

- **[Concept](docs/concept.md)** — Vision, use cases, and how Mnemonic works end-to-end
- **[Requirements](docs/requirements.md)** — Functional and non-functional requirements
- **[Requirements Traceability](docs/requirements-traceability.md)** — Mapping requirements to architecture decisions and components

### Architecture

Start with [System Architecture Overview](docs/architecture/system/README.md), then explore component-specific architecture:

| Directory                     | Contents                                                                                         |
| ----------------------------- | ------------------------------------------------------------------------------------------------ |
| `docs/architecture/system/`   | System-wide architecture: decisions, security, communication patterns, deployment, observability |
| `docs/architecture/mnemonic/` | MCP service: tool discovery, definitions, response patterns                                      |
| `docs/architecture/dbs/`      | Database schemas, migrations, deployment pipeline                                                |
| `docs/architecture/admin/`    | Frontend stack and architectural decisions                                                       |

### Diagrams

Reference diagrams for conceptual understanding:

- `docs/diagrams/concept.md` — High-level system concept
- `docs/diagrams/01-04-concept-local-iteration.md` — Concept iterations (local deployment)
- `docs/diagrams/05-concept-cloud-iteration.md` — Concept iteration (cloud deployment)
- `docs/diagrams/post-pattern-flow.md` — End-to-end pattern flow after initial match
- `docs/diagrams/example-go-ci-cd-diagram.md` — CI/CD pipeline example

## Quick Navigation

**First time here?** Read in this order:

1. [Concept](docs/concept.md) — understand the problem and solution
2. [Architecture System Overview](docs/architecture/system/README.md) — see how components fit together
3. [Requirements](docs/requirements.md) — learn what the system must do

**Looking for specific component architecture?** Jump to:

- MCP tools: [docs/architecture/mnemonic/](docs/architecture/mnemonic/)
- REST API: See system architecture docs (no separate API docs repo exists; API design lives in mnemonic-api source)
- Databases: [docs/architecture/dbs/](docs/architecture/dbs/)
- Admin UI: [docs/architecture/admin/](docs/architecture/admin/)

**Troubleshooting or decisions?** Check:

- [Architectural Decisions](docs/architecture/system/00-architectural-decisions.md) — ADRs across the system
- [Security Architecture](docs/architecture/system/01-security-architecture.md) — authentication, authorization, and compliance

## Contributing

Architecture and design decisions for the Mnemonic solution are documented here. When you make a change in a source repository:

1. Update the relevant architecture document in `mnemonic-docs` to reflect the change
2. If the change affects multiple components, update the system-level architecture
3. Update or create an ADR if the change involves a significant technical decision

All five source repositories reference this repository for their architecture and concept documentation.

---

Copyright (c) 2025-2026 Jeremy K. Johnson. All rights reserved.
