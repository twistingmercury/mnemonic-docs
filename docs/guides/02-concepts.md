# Core Concepts

This guide explains the key ideas behind Mnemonic: patterns, semantic search, embeddings, enrichment, and MCP. Understanding these concepts will help you use Mnemonic effectively and make informed decisions about how to structure your team's knowledge.

## Patterns: The Core Unit of Knowledge

A pattern is a Markdown document that captures a piece of team knowledge — a coding convention, architectural decision, best practice, or lesson learned. Patterns are the core unit of knowledge that lives in Mnemonic.

### Structure of a Pattern

Every pattern has two parts: frontmatter (metadata) and content.

**Frontmatter (YAML):**

Patterns begin with YAML metadata that defines searchable properties:

```yaml
---
name: go-error-handling
description: How our team handles errors in Go services
entity_type: pattern
language: go
domain: backend
tags:
  - error-handling
  - reliability
agents:
  - go-software-engineer
---
```

These fields are searchable. When you ask Mnemonic "show me error handling patterns in Go," the system searches both the frontmatter and the content body.

**Content Body:**

The body is regular Markdown. It explains the pattern, includes code examples, rationale, exceptions, and anything else your team needs to know.

### Marking Sections for Search: Pattern Decorators

Not every section of a pattern is meant to be indexed for semantic search. Mnemonic uses decorators to mark which sections should be included in the searchable vector index.

Sections prefixed with `[//]: pattern` are indexed as independent searchable chunks:

```markdown
[//]: pattern
## How We Handle Errors

Our team uses the following approach...
```

Sections without this decorator are stored in Mnemonic but not indexed for semantic search. This means if you add a changelog or disclaimer section, it won't pollute search results.

**Special case:** The `## Overview` section is never indexed, even if it has a pattern decorator. This lets you write a concise summary without it competing in search rankings.

Each decorated section becomes an independently searchable chunk. If your pattern has three decorated sections, all three are indexed separately, allowing more precise matches when Claude Code searches.

### Example Pattern Structure

```markdown
---
name: go-error-handling
description: How our team handles errors in Go services
entity_type: pattern
language: go
domain: backend
tags:
  - error-handling
  - reliability
agents:
  - go-software-engineer
---

## Overview

This pattern describes how we handle errors in Go services.

[//]: pattern
## The Sentinel Error Approach

We use sentinel errors for expected failure modes...

[//]: pattern
## Wrapping Errors

When propagating errors up the stack...

[//]: pattern
## Logging Failures

We log errors at the boundary where we handle them...

## Changelog

- 2026-03: Updated to reflect structured logging changes
```

In this example, three sections are indexed: "The Sentinel Error Approach," "Wrapping Errors," and "Logging Failures." The Overview and Changelog are not indexed.

## Embeddings: Making Patterns Searchable

When you create a pattern, Mnemonic doesn't just store the text. It generates a semantic embedding — a numeric representation of meaning — that makes similarity search fast and relevant.

### What Is an Embedding?

An embedding is a list of 2,000 numbers that represents the semantic content of a piece of text. Think of it as a fingerprint of meaning: similar ideas produce similar embeddings, while unrelated ideas produce very different embeddings.

Mnemonic uses OpenAI's `text-embedding-3-large` model to generate embeddings (with Matryoshka truncation at 2000 dimensions). The model is trained to capture semantic relationships, so "error handling in Go" and "how to propagate failures in a service" produce similar embeddings even though the words are different.

### How Embeddings Enable Search

When Claude Code searches Mnemonic with a query like "how do we handle errors?", here's what happens:

1. Mnemonic embeds your query to a 2,000-dimensional vector
2. It searches PGVector (a Postgres extension) to find patterns with the most similar embeddings
3. Results are ranked by cosine similarity: how closely aligned the vectors are

**Cosine similarity scores range from 0.0 to 1.0:**
- **1.0** = identical meaning
- **0.7–1.0** = highly relevant (typical search threshold)
- **0.5–0.7** = somewhat related
- **Below 0.5** = probably unrelated

The default threshold is 0.5, though it is configurable. This catches both closely related and somewhat related patterns.

### Why This Matters for Pattern Authors

Embeddings work best when your patterns are:

- **Specific**: "Error handling in Go services" ranks better than a vague "error patterns"
- **Clear**: Write naturally; the embedding model understands plain English
- **Complete**: Include examples and rationale; more context produces better embeddings
- **Chunked**: Use multiple decorated sections if your pattern covers different subtopics

Embeddings don't care about keyword matching. You don't need exact words to find what you need. A query about "propagating failures" will find your error-handling pattern because the concepts are similar, not because the words match exactly.

## Semantic Search: Finding Patterns by Concept

Semantic search in Mnemonic is concept-driven, not keyword-driven. This is what makes it powerful.

### Vector Similarity Search

When you ask Mnemonic "how do we handle timeouts in our API services?", the system:

1. Generates an embedding of your query
2. Searches the PGVector index to find patterns with similar embeddings
3. Returns the closest matches, ranked by similarity score

This works across languages and different wording. A pattern titled "timeout strategies in REST APIs" will match your query even though you used different words.

### Search Filters and Tags

Beyond semantic similarity, you can filter results using:

- **Tags**: Match patterns that have specific tags (AND logic: all specified tags required)
- **Language**: Narrow to patterns for a specific language (go, python, typescript, etc.)
- **Domain**: Filter by domain (backend, testing, frontend, devops, etc.)
- **Agent**: Show only patterns associated with a specific agent or skill

These filters work together with semantic search. Your query is still embedded and ranked by similarity, but only patterns matching all your filters are considered.

### Similarity Threshold

The default threshold is 0.5, though it is configurable. When Claude Code calls `search_patterns`, it can adjust the threshold:

- Higher threshold (0.7–0.9): Stricter, only highly relevant patterns
- Lower threshold (0.3–0.5): Broader, catches more tangential ideas

The default strikes a balance between relevance and coverage. You can tune it based on your team's pattern library size and specificity.

## Enrichment: The Background Pipeline

When a pattern is posted to Mnemonic, it doesn't instantly become searchable. It goes through an enrichment pipeline that generates embeddings and builds knowledge graph relationships.

### The Enrichment Lifecycle

**1. Pattern arrives via REST API**

Someone (an admin, CI pipeline, or sync script) POSTs a new pattern to Mnemonic's admin API. The pattern is stored in PostgreSQL with status `pending`.

**2. Enrichment worker claims the job**

The enrichment worker (mnemonic-enricher) subscribes to a RabbitMQ queue. When the API creates a pattern, it publishes an enrichment job to the queue. The worker picks up the message and begins processing.

**3. Embedding is generated**

The worker sends the pattern text to OpenAI's embedding API, which returns a 2,000-dimensional vector. This embedding is stored in PostgreSQL alongside the pattern, ready for vector search.

**4. Knowledge graph is built**

The worker analyzes the pattern's concepts and relationships. It creates nodes in Neo4j representing key concepts and edges representing how patterns relate to each other. For example, if two patterns share concepts like "error handling," the enrichment worker creates a `RELATED_TO` edge between them.

**5. Pattern is marked as enriched**

The worker updates the pattern status to `enriched`. Only enriched patterns appear in semantic search results.

### Why Async?

Enrichment happens asynchronously (not during the HTTP POST) because calling the embedding API takes time. Making the user wait for OpenAI would be slow. Instead, Mnemonic accepts the pattern immediately (HTTP 202 Accepted) and enriches it in the background. By the time you use Claude Code a few moments later, the pattern is ready to search.

### Requirements

Enrichment requires an OpenAI API key. Set the `MNEMONIC_OPENAI_API_KEY` environment variable before running the enrichment worker. Without this key, patterns will remain in `pending` status and never become searchable.

### Failure Handling

If enrichment fails (network timeout, API error, etc.), the job is retried automatically. The retry strategy is exponential backoff — failed jobs get retried sooner at first, then increasingly delayed intervals.

Failed patterns don't stay in `pending` forever. Eventually, an admin will notice and either fix the issue (e.g., rotate an expired API key) or delete the pattern.

## Knowledge Graphs: Relationships Between Patterns

Beyond semantic embeddings, Mnemonic builds a Neo4j knowledge graph that captures relationships between patterns.

### What the Graph Contains

The enrichment worker analyzes patterns and creates:

- **Nodes**: Key concepts extracted from each pattern (e.g., "error handling," "database migration," "HTTP middleware")
- **Edges**: Relationships between concepts (e.g., two patterns that both discuss "error handling" are related)

The edge type is `RELATED_TO`, indicating general conceptual similarity.

### Why the Graph Matters

The knowledge graph serves two purposes:

1. **Related pattern discovery**: Given one pattern, find others that discuss similar concepts. The `find_related_patterns` MCP tool uses the graph to traverse and return related patterns.

2. **Future ranking improvements**: In later phases, Mnemonic will combine vector similarity with graph context for blended ranking. A pattern that is both semantically similar AND connected via the knowledge graph will rank higher.

For now (MVP), graph relationships are visible in the `get_pattern` and `find_related_patterns` tools, but vector search is purely embedding-based.

## MCP: How Claude Code Talks to Mnemonic

MCP (Model Context Protocol) is the standard way Claude Code integrates with external services. Mnemonic exposes itself as an MCP server that Claude Code can call.

### What MCP Is

MCP is a protocol for exchanging information between AI assistants and external tools. It's not HTTP or gRPC — it's a message-based protocol designed specifically for Claude Code to call remote functions reliably.

### The Connection

Claude Code connects to Mnemonic via MCP over Streamable HTTP:

```
Claude Code → http://localhost:8081/mcp → Mnemonic MCP Server
```

This is configured in your Claude Code settings. See the Quick Start guide for setup instructions.

### The Three Tools

Mnemonic exposes three read-only MCP tools:

#### `search_patterns`

Semantic search over the team knowledge graph.

**Parameters:**
- `query` (required, string): Your natural language question
- `limit` (optional, default 10, max 50): How many results to return
- `threshold` (optional, default 0.5, range 0.0–1.0): Minimum similarity score
- `tags` (optional, string array): Filter by tags (AND logic)
- `agent` (optional, string): Filter by agent association

**Returns:** Markdown text with ranked results, similarity scores, tags, and full pattern content.

Example: Claude Code might call this when you ask "how do we test database migrations?" and it determines that team knowledge is relevant.

#### `find_related_patterns`

Find patterns related to a given pattern via the knowledge graph.

**Parameters:**
- `pattern_id` (required, UUID): ID of the starting pattern
- `limit` (optional, default 5, max 20): How many related patterns to return

**Returns:** Markdown with related patterns, shared concepts, and similarity scores.

Example: After retrieving a pattern about database migrations, Claude Code might call this to find related patterns about schema management or rollback strategies.

#### `get_pattern`

Retrieve a specific pattern by UUID.

**Parameters:**
- `id` (required, UUID): The pattern's ID

**Returns:** Markdown with full pattern content, metadata, relationships, and extracted concepts.

Example: When you ask "show me the migration pattern," Claude Code calls this tool with the ID.

### Read-Only Access

All MCP tools are read-only. Claude Code can search and retrieve patterns, but it cannot create, modify, or delete them. Pattern management happens via the REST API, not through MCP. This keeps the team's knowledge base controlled and authoritative.

### Error Handling

If a tool call fails (pattern not found, database unavailable, etc.), Mnemonic returns a clear error message in the tool result. Claude Code sees the error and can inform you or retry. For protocol-level errors (malformed requests, unknown tools), MCP returns standard JSON-RPC errors.

See [MCP Tools](../architecture/mnemonic/08-mcp-tools.md) for complete tool specifications and error codes.

## System Components: A Quick Tour

Here's where everything runs:

| Component | Port | Purpose |
|-----------|------|---------|
| **mnemonic-mcp** | 8081 | MCP server for Claude Code |
| **mnemonic-api** | 8080 | Admin REST API for pattern CRUD |
| **mnemonic-enricher** | 8082 | Async enrichment worker |
| **PostgreSQL** | 5432 | Pattern storage and PGVector indexes |
| **Neo4j** | 7687 | Knowledge graph |
| **RabbitMQ** | 5672 | Message queue between API and enricher |
| **mnemonic-admin** | 3000 | Web UI for pattern management |

Developers interact with the MCP server (port 8081). Admins use the REST API (port 8080) and the web UI (port 3000). The enrichment worker (port 8082) runs silently in the background.

For deployment details, see [System Architecture](../architecture/system/02-system-architecture.md).

## What Mnemonic Does NOT Do

Mnemonic is focused and purposeful. It deliberately does not:

- **Perform LLM inference**: Mnemonic calls the embedding API only. It does not generate text, write code, or make decisions. That's Claude Code's job.
- **Store user credentials**: Mnemonic never receives or stores passwords, API keys, or authentication tokens.
- **Execute tools or file operations**: Mnemonic cannot run commands, write files, or take actions on your system.
- **Make workflow decisions**: Mnemonic is a memory system, not a workflow engine. You decide when and how to use the knowledge it provides.

This narrow scope keeps Mnemonic simple, secure, and focused on its core job: being your team's searchable knowledge base.

## Next Steps

Now that you understand the concepts, you're ready to:

- **Use Mnemonic**: See [Quick Start for Developers](03-quickstart.md) to configure Claude Code
- **Dive deeper**: See [System Architecture](../architecture/system/02-system-architecture.md) for technical details and component interactions

---

Copyright (c) 2025-2026 Jeremy K. Johnson. All rights reserved.
