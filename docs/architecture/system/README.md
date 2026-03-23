# Architecture

> **Source:** Reconciled from `mnemonic/docs/architecture/README.md`, `mnemonic-api/docs/architecture/README.md`, `mnemonic-enricher/docs/architecture/README.md`
> **Component-specific divergences:** None — all three versions are identical
> **Last reconciled:** 2026-03-23

Start here:

1. [What is Mnemonic?](../../concept.md) — the problem, the vision, how it fits together
2. [Requirements](../../requirements.md) — what Mnemonic must do
3. The architecture docs below — how those requirements are realized

## Architecture Documents

| Document                                                       | Description                                    |
| -------------------------------------------------------------- | ---------------------------------------------- |
| [Architectural Decisions](00-architectural-decisions.md)       | Key decision records (ADR-001 through ADR-009) |
| [Security Architecture](01-security-architecture.md)           | Authentication and authorization (Post-MVP)    |
| [System Architecture](02-system-architecture.md)               | Component breakdown and data flow              |
| [Communication Patterns](03-communication-patterns.md)         | MCP and REST protocol patterns                 |
| [Data Architecture](04-data-architecture.md)                   | Database schemas and data management           |
| [Database Integration Flow](05-database-integration-flow.md)   | End-to-end database interaction patterns       |
| [Deployment Architecture](06-deployment-architecture.md)       | Deployment topology and operations             |
| [Observability Architecture](07-observability-architecture.md) | Monitoring, logging, and tracing               |
| [MCP Tools](../mnemonic/08-mcp-tools.md)                      | Tool discovery, definitions, and response patterns |

## Design Documents

Architecture docs describe **what** and **why**. Design docs describe **how** and remain in each component repo.

<!-- design docs remain in each component repo's docs/design/ -->
