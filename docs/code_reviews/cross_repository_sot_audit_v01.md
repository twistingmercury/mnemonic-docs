# Code Review: Cross-Repository Documentation Source-of-Truth Audit

> **Version**: v01
> **Date**: 2026-08-24
> **Notes**: Initial cross-repository audit, updated to record reconciliation of the duplicate-MCP-listener finding.

**Reviewers:** code-reviewer, solutions-architect, go-software-engineer
**Phase:** cross-repository documentation audit

## Files Reviewed

### Source Files

- `mnemonic-docs/docs/` - requirements, architecture, design, reference, guide, and admin documentation.
- `mnemonic-api/src/` - REST routes, configuration, queue publisher, runtime wiring, and generated Swagger contract.
- `mnemonic-mcp/src/` - MCP tool schema, search behavior, configuration, and runtime wiring.
- `mnemonic-enricher/src/` - worker pipeline, OpenAI extraction, configuration, and runtime wiring.
- `mnemonic-dbs/src/` - PostgreSQL and Neo4j migrations.
- `mnemonic-admin/src/` - browser client contract and implemented filters.

### Test Files

- `mnemonic-api/src/internal/**/**/*_test.go` - inspected as contract evidence where relevant.
- `mnemonic-mcp/src/internal/**/**/*_test.go` - inspected as contract evidence where relevant.
- `mnemonic-enricher/src/internal/**/**/*_test.go` - inspected as contract evidence where relevant.

## Validation Results

| Tool | Result |
| ---- | ------ |
| Repository history/status inspection | Documentation repository head is 2026-04-07; API, MCP, Enricher, and Admin heads are 2026-08-19 to 2026-08-23. No pre-existing docs worktree changes. |
| Cross-repository static contract comparison | Completed across documentation, Go source/config/routes, SQL/Cypher migrations, Swagger, and TypeScript client types. |
| Automated tests | Not run: this was a read-only documentation audit; existing test code was used only as supplementary implementation evidence. |

## Design Compliance

Implementation does **not** currently satisfy all behavioral requirements and architectural contracts documented in `mnemonic-docs`. The largest remaining inconsistencies are the removed-but-still-documented agents/skills product model and the operational configuration contract. The runtime service boundary has been reconciled: `mnemonic-mcp` exclusively serves MCP, while `mnemonic-api` retains its REST and operational endpoints.

### Behavioral Requirements Verified

- Patterns are chunked and semantic search uses 2000-dimensional embeddings. ✓
- The MCP surface remains read-only with the three documented tool names. ✓
- Pattern creation is asynchronous and uses RabbitMQ-backed enrichment work. ✓

### Design Doc Divergences (Post-Review)

No fixes were applied in this audit. The following are current discrepancies that require an explicit decision on whether documentation or implementation is authoritative.

#### Naming Divergences (1 active contract rename)

| Old Name (in design docs) | New Name (in implementation) | Reason |
| ------------------------- | ---------------------------- | ------ |
| `server.admin.*` / root `rabbitmq.*` | `server.*`, `mcp.*`, and `queue.rabbitmq.*` | The current service configuration hierarchy changed after the design documentation was written. |

#### Structural Divergences (require reconciliation)

| Divergence | Design Doc | Implementation | Assessment |
| ---------- | ---------- | -------------- | ---------- |
| Pattern indexing | Pattern-level storage and mixed IVFFlat guidance | Decorated chunks with HNSW at 2000 dimensions | Implementation is coherent; documentation must be normalized. |
| Tooling model | Agents, skills, and associations persisted and exposed | Only patterns, chunks, jobs, Patterns and Concepts graph data exist | Documentation conflicts with its own current scope decision. |
| Service topology | MCP serving belongs exclusively to `mnemonic-mcp` | `mnemonic-api` starts only its Admin API; `mnemonic-mcp` retains the MCP listener | Reconciled by the MCP-listener removal. Operational HTTP listeners remain expected health/observability surfaces. |

#### Documents Updated

| Document | Scope | Status |
| -------- | ----- | ------ |
| `docs/code_reviews/cross_repository_sot_audit_v01.md` | Evidence-backed SoT audit | Added |

## Findings

### HIGH Priority

| ID | Source | Finding | Resolution |
| -- | ------ | ------- | ---------- |
| H1 | architecture + Go review | Reconciled: `mnemonic-mcp` exclusively owns MCP serving; `mnemonic-api` runs only its Admin API and retains its approved operational endpoints. Evidence: `mnemonic-api/src/internal/server/server.go:96`, `mnemonic-api/src/internal/server/server_test.go:16`, `mnemonic-mcp/src/internal/server/server.go:103`. | Complete. The API MCP listener was removed and the system architecture now records `mnemonic-mcp` as the sole MCP owner. |
| H2 | general + Go review | The required configuration contract is stale and cannot configure the implementation. Docs prescribe `server.admin.*`, `server.mcp.*`, and root `rabbitmq.*`; runtime accepts `server.*`, `mcp.*`, and `queue.rabbitmq.*`. Documented variables such as `MNEMONIC_SERVER_ADMIN_PORT` and `MNEMONIC_RABBITMQ_URL` are ignored. Evidence: `docs/design/configuration.md:81`, `docs/design/configuration.md:275`, `mnemonic-api/src/internal/config/config.go:276`, `mnemonic-api/src/internal/config/defaults.go:83`. | Replace the configuration design with per-service, code-derived keys/defaults and add a configuration-contract test or generated reference. |
| H3 | general + architecture review | Agents, skills, skill files, agent associations, and `RELEVANT_FOR` graph edges remain described in system/DB/admin docs even though requirements remove tooling management and the migrations/API do not implement them. Evidence: `docs/requirements.md:40`, `docs/architecture/system/04-data-architecture.md:98`, `mnemonic-dbs/src/postgres/000005_pattern_schema_chunks.up.sql:46`, `mnemonic-dbs/src/neo4j/001_create_constraints.cypher:31`, `mnemonic-api/src/internal/server/routes.go:22`. | Remove these from current architecture, diagrams, traceability, glossary, service design, and UI use cases; retain only as explicitly scoped future work if desired. |
| H4 | general + Go review | The documentation promises embedding-only external AI use, but enrichment calls OpenAI Chat Completions with `gpt-4o-mini` to extract concepts. This is a privacy, cost, and governance boundary mismatch. Evidence: `docs/requirements.md:110`, `docs/architecture/system/02-system-architecture.md:108`, `mnemonic-enricher/src/internal/service/openai/extraction.go:29`. | Either amend requirements/architecture for generative concept extraction and its controls, or remove the extraction implementation. |
| H5 | general + Go review | AD-5 requires an administrator-triggered retry endpoint, but no such API route or Swagger operation exists. Internal retries are not an administrator recovery interface. Evidence: `docs/requirements.md:59`, `mnemonic-api/src/internal/handlers/patterns/patterns.go:53`, `mnemonic-api/src/internal/server/routes.go:25`. | Implement a retry/requeue endpoint with authorization/idempotency semantics, or formally remove AD-5 and its traceability entries. |
| H6 | architecture review | The RabbitMQ message contract and recovery claim are incorrect. Docs describe a pattern ID/action sent through an exchange; the producer sends only `job_id` to the default exchange/queue. On publish failure the job remains pending, but no component republishes it. Evidence: `docs/architecture/system/02-system-architecture.md:221`, `docs/architecture/system/03-communication-patterns.md:154`, `mnemonic-api/src/internal/queue/rabbitmq/publisher.go:92`, `mnemonic-api/src/internal/service/pattern/service.go:196`, `mnemonic-enricher/src/internal/enricher/enrichment.go:243`. | Publish a versioned message contract and assign durable pending-job recovery to an API reconciler, worker reconciler, or separate component. Document cadence, metrics, and operator action. |
| H7 | Go + architecture review | Database documentation lists nonexistent migrations 000007–000010 and stale job/chunk columns. Actual schema ends at 000006; jobs use `attempts`, `max_attempts`, and `last_error`, while `section_title` is required and chunk target exclusivity is enforced. Evidence: `docs/architecture/dbs/05-deployment-architecture.md:49`, `docs/architecture/system/04-data-architecture.md:305`, `mnemonic-dbs/src/postgres/000003_create_enrichment_jobs.up.sql:27`, `mnemonic-dbs/src/postgres/000005_pattern_schema_chunks.up.sql:46`. | Rebuild database architecture and design documentation directly from the migration set; gate migration changes on a docs update. |

### MEDIUM Priority

| ID | Source | Finding | Resolution |
| -- | ------ | ------- | ---------- |
| M1 | general + Go review | MCP `search_patterns` documents a nonexistent `agent` filter and omits implemented `language` and `domain` filters; it also describes whole-pattern results whereas implementation formats matching chunks. Evidence: `docs/architecture/mnemonic/08-mcp-tools.md:50`, `mnemonic-mcp/src/internal/mcpserver/input.go:13`, `mnemonic-mcp/src/internal/mcpserver/format.go:23`. | Regenerate/tool-test the MCP reference from the SDK input/output schema and state chunk/graph-result semantics. |
| M2 | Go review | KM-1 still caps content at 10 KB, but migration 000005 removes the constraint and API validation permits 100,000 bytes. Evidence: `docs/requirements.md:25`, `mnemonic-dbs/src/postgres/000005_pattern_schema_chunks.up.sql:18`, `mnemonic-api/src/internal/handlers/patterns/patterns.go:296`. | Decide the intended limit; enforce it consistently or update requirement, schema, and user guidance to the 100 KB bound. |
| M3 | architecture review | MCP search documentation calls graph expansion post-MVP, but the MCP service already expands vector seeds through Neo4j. Evidence: `docs/architecture/system/02-system-architecture.md:166`, `mnemonic-mcp/src/internal/service/search/service.go:164`. | Document current ranking, result blending, failure behavior, and graph-read dependency. |
| M4 | general + architecture review | Admin UI documentation promises agent filters/association display and labels an asynchronous import response “202 Created”; the UI/API provide neither agent data nor a created lifecycle. Evidence: `docs/architecture/admin/screen-map.md:55`, `docs/architecture/admin/screen-map.md:174`, `mnemonic-admin/src/features/patterns/components/filterTypes.ts:1`, `mnemonic-api/src/internal/handlers/patterns/patterns.go:351`. | Remove the unscheduled UI promises and call the state “accepted for enrichment,” including pending/enriched/failed refresh behavior. |
| M5 | Go review | DP-3 requires startup schema-compatibility verification, but the services open connections without a schema/version/constraint check. Evidence: `docs/requirements.md:100`, `mnemonic-api/src/internal/server/server.go:63`, `mnemonic-dbs/src/neo4j/003_create_indexes.cypher:52`. | Implement a checked compatibility contract, or mark DP-3 unimplemented and remove it from verified requirements. |

### LOW Priority

| ID | Source | Finding | Resolution |
| -- | ------ | ------- | ---------- |
| L1 | general + Go review | Data architecture recommends IVFFlat, while deployed migrations intentionally create HNSW because it initializes safely on an empty table and recreate HNSW after the 2000-dimension migration. Evidence: `docs/architecture/system/04-data-architecture.md:374`, `mnemonic-dbs/src/postgres/000005_pattern_schema_chunks.up.sql:64`, `mnemonic-dbs/src/postgres/000006_update_embedding_dimensions.up.sql:11`. | Replace IVFFlat guidance with HNSW operations and capacity guidance. |
| L2 | main review | The docs repository is four months behind active implementation heads, with no visible synchronization control despite being declared the SoT. Evidence: `mnemonic-docs` HEAD 2026-04-07; implementation heads 2026-08-19 through 2026-08-23. | Add a cross-repository documentation-change gate and release/audit cadence. |

## Patterns to Document

Patterns identified that should be added to the project's guidance for installed subagents.

1. Every externally consumable route, MCP tool, queue message, configuration key, and database migration must have one owned contract and an automated docs/contract check.
2. A repository split must have an explicit runtime-boundary matrix: process, listeners, ports, write permissions, dependencies, and deployable image for each service.
3. Requirements traceability must distinguish implemented, planned, and removed capabilities; removed features must be deleted from downstream architecture and UX documents in the same change.

## Notes for Future Phases

**Phase documentation-reconciliation**: H1 is reconciled: `mnemonic-mcp` owns MCP serving and `mnemonic-api` owns the REST and operational API surface. Next, remove the obsolete agents/skills model (H3), regenerate DB/config/tool references (H2, H7, M1), and decide whether H4, H5, and M5 are implementation work or requirement changes.
