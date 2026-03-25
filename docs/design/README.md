# Design Documents

Implementation-level design documents covering cross-cutting concerns shared by the Go services (mnemonic, mnemonic-api, mnemonic-enricher).

These complement the architecture documents in `../architecture/` — architecture describes _what_ and _why_, design describes _how_.

## Documents

| Document | Covers |
|---|---|
| [Configuration](configuration.md) | Layered configuration via Viper: defaults, config files, environment variables |
| [Data Storage](data-storage.md) | JSONB document model, PostgreSQL migrations, PGVector indexes, Neo4j schema, repository interfaces |
| [MCP Server](mcp-server.md) | MCP server setup, StreamableHTTP handler, tool registration, error handling |
| [Observability Implementation](observability-implementation.md) | OpenTelemetry integration, tracing, metrics, structured logging via zerolog |
| [Pattern Processing](pattern-processing.md) | Enrichment pipeline: chunking, embeddings, entity extraction, Neo4j relationships, job queue |
| [Service Layer](service-layer.md) | Service interfaces, dependency injection, business logic organization |
| [Design Changelog](design-changelog.md) | History of design decisions and iterations |
