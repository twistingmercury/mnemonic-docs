# Documentation Review — Approach B Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Fix factual errors, fill content gaps, resolve duplicates, create a glossary, and standardize navigation/metadata across all architecture, design, reference, and diagram docs so that a new contributor can trace the full path from concept → requirements → architecture → design without hitting dead ends or contradictions.

**Architecture:** Pure documentation edits — no code changes. All work happens in `docs/` within the `mnemonic-docs` repository. Changes are organized in dependency order: factual fixes first (no new content needed), then structural additions (glossary, gap fills), then standardization passes (metadata, navigation, cross-references).

**Tech Stack:** Markdown, Mermaid diagrams

**Scope:** `docs/architecture/`, `docs/design/`, `docs/reference/`, `docs/diagrams/`, `docs/concept.md`, `docs/requirements.md`, `docs/requirements-traceability.md`, `PURPOSE.md`

---

## Task 1: Fix Factual Errors and Stale Content

**Files:**
- Modify: `PURPOSE.md:1`
- Modify: `docs/design/data-storage.md:428`
- Modify: `docs/design/pattern-processing.md:734`
- Modify: `docs/architecture/dbs/08-data-architecture.md` (idx_patterns_embedding row)
- Modify: `docs/architecture/admin/0001-frontend-stack.md:3`
- Modify: `docs/architecture/admin/tech-stack-justification.md:8`

- [ ] **Step 1: Fix PURPOSE.md title typo**

In `PURPOSE.md`, line 1:

Change:
```markdown
# PMNEMONIC DOCS
```
To:
```markdown
# Mnemonic Docs
```

- [ ] **Step 2: Fix stale 1536d dimension references**

In `docs/design/data-storage.md`, line 428:

Change:
```sql
comment on column patterns.embedding is 'Vector embedding (1536d) for semantic similarity search';
```
To:
```sql
comment on column patterns.embedding is 'Vector embedding (2000d, text-embedding-3-large with Matryoshka truncation) for semantic similarity search';
```

In `docs/design/pattern-processing.md`, line 734:

Change:
```markdown
- [ ] PGVector dimensions match configured embedding dimensions (1536)
```
To:
```markdown
- [ ] PGVector dimensions match configured embedding dimensions (2000)
```

- [ ] **Step 3: Mark deprecated index in dbs/08-data-architecture.md**

In `docs/architecture/dbs/08-data-architecture.md`, find the `idx_patterns_embedding` row in the Indexes table. Add a note that this index was superseded by `idx_pattern_chunks_embedding` after migration 000009 moved embeddings to the `pattern_chunks` table. The exact edit depends on the table format — add "(deprecated — superseded by `idx_pattern_chunks_embedding` in migration 000009)" to the description column of that row.

- [ ] **Step 4: Fix macOS file paths in admin docs**

In `docs/architecture/admin/0001-frontend-stack.md`, line 3 — find the source metadata referencing `/Users/doublej/dev/mnemonic-admin/docs/adr/0001-frontend-stack.md` and change it to a relative reference:

Change:
```markdown
> **Source:** `/Users/doublej/dev/mnemonic-admin/docs/adr/0001-frontend-stack.md`
```
To:
```markdown
> **Source:** `mnemonic-admin/docs/adr/0001-frontend-stack.md`
```

In `docs/architecture/admin/tech-stack-justification.md`, line 8 — same pattern. Find the absolute macOS path and change to:
```markdown
> **Related decision record:** [0001-frontend-stack.md](0001-frontend-stack.md)
```

- [ ] **Step 5: Commit**

```bash
git add PURPOSE.md docs/design/data-storage.md docs/design/pattern-processing.md docs/architecture/dbs/08-data-architecture.md docs/architecture/admin/0001-frontend-stack.md docs/architecture/admin/tech-stack-justification.md
git commit -m "docs: fix factual errors — typo, stale dimensions, deprecated index, macOS paths"
```

---

## Task 2: Resolve Migration Tooling Contradiction in data-storage.md

The `data-storage.md` design doc references `golang-migrate` CLI as the migration tool (lines 63, 271, 298), but the project moved to Docker entrypoint–based schema application per `dbs/02-architectural-decisions.md` ADR-001. The doc also shows `.down.sql` files in the migration tree, but the project no longer maintains down migrations (rollback is by pulling a prior image tag).

**Files:**
- Modify: `docs/design/data-storage.md:55-75` (Implementation Scope table)
- Modify: `docs/design/data-storage.md:265-310` (Migration File Structure section)
- Modify: `docs/design/data-storage.md:1734` (References section — remove golang-migrate link)

- [ ] **Step 1: Update Implementation Scope table**

In `docs/design/data-storage.md`, the table around line 58-63, change the Migrations row:

Change:
```markdown
| Migrations | golang-migrate CLI | Schema versioning and deployment (run externally, not embedded in Mnemonic) |
```
To:
```markdown
| Migrations | Docker entrypoint scripts | Schema pre-applied at image build time via `docker-entrypoint-initdb.d` (Postgres) and `cypher-shell` (Neo4j); no runtime migration tool |
```

- [ ] **Step 2: Update Migration File Structure section**

In `docs/design/data-storage.md`, lines 269-310:

Replace the paragraph at line 271:
```markdown
All PostgreSQL migrations follow the golang-migrate flat file convention. Migrations are run by the golang-migrate CLI as a deployment step; Mnemonic does not run or manage migrations at runtime.
```
With:
```markdown
All PostgreSQL migrations are `.up.sql` files applied at Docker image build time via `docker-entrypoint-initdb.d`. Mnemonic does not run or manage migrations at runtime. There are no `.down.sql` files — rollback is accomplished by pulling a prior image tag (see [dbs ADR-001](../architecture/dbs/02-architectural-decisions.md#adr-001-pre-seeded-images-over-runtime-migration-tooling)).
```

Remove the `.down.sql` entries from the migration file tree listing (lines 277, 279, 281, etc.) so only `.up.sql` files remain.

Replace the golang-migrate paragraph at line 298:
```markdown
golang-migrate requires a single flat directory with paired `.up.sql` and `.down.sql` files. The 6-digit zero-padded prefix (e.g., `000001`) matches the output of `migrate create -ext sql -dir src/migrations/postgres -seq <name>`. The `-path` flag points to this directory.
```
With:
```markdown
PostgreSQL migration files use a 6-digit zero-padded prefix (e.g., `000001`) to guarantee execution order. The Docker entrypoint runs all `.up.sql` files from `/docker-entrypoint-initdb.d/` in filename-sorted order on first container start.
```

Remove the "Running Migrations" code block that shows `migrate` CLI commands (lines 302-309).

- [ ] **Step 3: Remove golang-migrate reference link**

In `docs/design/data-storage.md`, find the References section near line 1734 and remove the golang-migrate GitHub link.

- [ ] **Step 4: Commit**

```bash
git add docs/design/data-storage.md
git commit -m "docs: update data-storage migration section to reflect Docker entrypoint approach

Replaces stale golang-migrate references with Docker entrypoint-based
schema application, matching dbs/ ADR-001 and current implementation."
```

---

## Task 3: Remove Pre-Pivot Routing Engine References from Observability Doc

The `observability-implementation.md` still documents routing engine, route handlers, and routing rule repository code that was removed during the 2026-02-15 architectural pivot. These sections should be removed or replaced with a brief note pointing to the design changelog.

**Files:**
- Modify: `docs/design/observability-implementation.md`

- [ ] **Step 1: Identify pre-pivot sections**

Read `docs/design/observability-implementation.md` and identify all sections that reference:
- Routing engine
- Route handlers
- Routing rule repository
- ACE CLI

These are flagged in the "Gaps and Additional Implementation" section and scattered throughout handler instrumentation patterns.

- [ ] **Step 2: Remove or replace pre-pivot content**

For each identified section:
- If the section is entirely about pre-pivot code, remove it entirely.
- If only part of a section references pre-pivot code, remove those paragraphs/bullets and leave the rest.
- At the location of the first removal, add a brief note:

```markdown
> **Note:** Routing engine, route handler, and routing rule repository instrumentation was removed during the [2026-02-15 architectural pivot](design-changelog.md#2026-02-15-architectural-pivot). Current instrumentation covers MCP tool handlers and Admin API handlers only.
```

- [ ] **Step 3: Commit**

```bash
git add docs/design/observability-implementation.md
git commit -m "docs: remove pre-pivot routing engine references from observability design"
```

---

## Task 4: Resolve Diagram Duplication (concept.md vs 02-concept-local-iteration.md)

`docs/diagrams/concept.md` and `docs/diagrams/02-concept-local-iteration.md` are nearly identical (both show MVP 2). The numbered series (01 through 05) should be the canonical sequence. `concept.md` should either be removed or repurposed.

**Files:**
- Remove: `docs/diagrams/concept.md`
- Modify: `README.md` (root — update any reference to `docs/diagrams/concept.md`)

- [ ] **Step 1: Verify no other doc links to docs/diagrams/concept.md**

Search all `.md` files for references to `diagrams/concept.md`. If any exist, update them to point to `diagrams/02-concept-local-iteration.md` instead.

- [ ] **Step 2: Remove the duplicate**

Delete `docs/diagrams/concept.md`.

- [ ] **Step 3: Update root README.md**

In `README.md`, the Diagrams section references `docs/diagrams/concept.md`. Remove that bullet or replace it with a note that the numbered series (01-05) shows the full evolution.

- [ ] **Step 4: Commit**

```bash
git rm docs/diagrams/concept.md
git add README.md
git commit -m "docs: remove duplicate concept.md diagram — 02-concept-local-iteration.md is canonical"
```

---

## Task 5: Create Glossary Document

No glossary exists. Key terms like "enrichment," "chunk," "decorator," "MCP," "agent," "pattern," and "knowledge graph" are used across all docs but defined inconsistently or not at all.

**Files:**
- Create: `docs/reference/glossary.md`
- Modify: `README.md` (add glossary to Reference section)

- [ ] **Step 1: Write the glossary**

Create `docs/reference/glossary.md` with definitions for all key terms used across the documentation. Each term should have a one-paragraph definition and optionally a "See also" cross-reference.

Terms to define (alphabetical):

- **Agent** — A named entity in Mnemonic representing a Claude Code agent role (e.g., `go-engineer`, `devops-engineer`). Patterns are associated with agents via relevance scores (0.0–1.0) so that search results can be filtered by agent context. Agent definitions are managed externally (version-controlled repos); Mnemonic stores only the name and a JSONB definition blob.

- **Chunk** — A decorated section of a pattern file that is independently vectorized and indexed for semantic search. Chunks are created by splitting pattern content at H2/H3 headings marked with the `[//]: pattern` decorator. Each chunk gets its own embedding in the `pattern_chunks` table. See also: Decorator, Embedding.

- **Cosine Similarity** — A measure of similarity between two vectors, calculated as the cosine of the angle between them. Values range from 0.0 (orthogonal/unrelated) to 1.0 (identical direction). Mnemonic uses PGVector's `<=>` operator to compute cosine similarity between pattern chunk embeddings and query embeddings. See also: Embedding.

- **Decorator** — The exact string `[//]: pattern` placed on its own line immediately before an H2 or H3 heading in a pattern file. Decorators mark sections for chunking and vectorization. Sections without decorators are discarded during indexing. The `## Overview` section must not be decorated. See also: Chunk, Pattern File.

- **Embedding** — A numeric vector (array of 2000 floating-point numbers) that represents the semantic meaning of text. Generated by OpenAI's `text-embedding-3-large` model with Matryoshka truncation to 2000 dimensions. Similar texts produce similar embeddings, enabling semantic search via cosine similarity. See also: Cosine Similarity, PGVector.

- **Enrichment** — The asynchronous pipeline that transforms a raw pattern into a searchable, interconnected knowledge entry. Five steps: (1) validate and store metadata, (2) generate embeddings for each chunk via OpenAI, (3) extract concepts via OpenAI, (4) create Neo4j relationships, (5) update status. Enrichment status tracks as `pending`, `enriched`, or `failed`. Only enriched patterns appear in search results. See also: Chunk, Knowledge Graph.

- **Frontmatter** — The YAML metadata block at the top of a pattern file, delimited by `---`. Contains required fields (`name`, `entity_type`, `language`, `domain`, `description`) and optional fields (`agents`, `tags`, `version`, `related_patterns`). See also: Pattern File.

- **HNSW** — Hierarchical Navigable Small World. A vector index algorithm used by PGVector for approximate nearest-neighbor search on `pattern_chunks.embedding`. Chosen over IVFFlat for chunk-level embeddings because it provides better recall without requiring periodic retraining. See also: IVFFlat, PGVector.

- **IVFFlat** — Inverted File with Flat compression. A vector index algorithm that partitions embeddings into clusters via k-means, then searches only the nearest clusters at query time. Previously used for pattern-level embeddings (now deprecated in favor of HNSW on chunks). See also: HNSW, PGVector.

- **Knowledge Graph** — The combination of Neo4j graph relationships and PostgreSQL vector data that represents the semantic structure of the pattern library. Nodes include `Pattern`, `Agent`, and `Concept`. Relationships include `RELEVANT_FOR` (Pattern → Agent), `RELATED_TO` (Pattern → Pattern), and `HAS_CONCEPT` (Pattern → Concept). See also: Enrichment.

- **MCP (Model Context Protocol)** — The protocol Claude Code uses to communicate with external tool servers. Mnemonic exposes three read-only MCP tools (`search_patterns`, `find_related_patterns`, `get_pattern`) via Streamable HTTP on port 8081. Claude Code discovers and invokes these tools automatically during conversation. See also: Streamable HTTP.

- **Pattern** — A Markdown document with YAML frontmatter that captures a piece of team knowledge — a coding convention, architectural decision, best practice, or design pattern. Patterns are the core unit of knowledge in Mnemonic. They are stored in PostgreSQL, chunked and vectorized for semantic search, and projected into Neo4j for graph traversal. See also: Pattern File, Chunk, Enrichment.

- **Pattern File** — The on-disk Markdown format for patterns. Consists of YAML frontmatter followed by an `## Overview` section (excluded from indexing) and one or more decorated sections (indexed as chunks). Validated by `install/validate.sh`. See also: Frontmatter, Decorator, Chunk.

- **PGVector** — A PostgreSQL extension that adds vector data types and similarity search operators. Mnemonic uses PGVector to store embeddings as `vector(2000)` columns and to perform cosine similarity searches using the `<=>` operator with HNSW indexes. See also: Embedding, HNSW.

- **Streamable HTTP** — The MCP transport protocol used by Mnemonic. Claude Code connects to `http://localhost:8081/mcp` and sends JSON-RPC 2.0 messages. Mnemonic runs in stateless mode (no session persistence). See also: MCP.

- [ ] **Step 2: Add glossary to root README.md**

In `README.md`, under the Reference section, add:
```markdown
- **[Glossary](docs/reference/glossary.md)** — Definitions of key terms used throughout the documentation
```

- [ ] **Step 3: Commit**

```bash
git add docs/reference/glossary.md README.md
git commit -m "docs: add glossary of key terms used across documentation"
```

---

## Task 6: Fill Content Gap — pattern_chunks Table in data-storage.md

The `pattern_chunks` table is referenced in `pattern-processing.md` and `service-layer.md` but never defined in `data-storage.md`. The `service-layer.md` itself notes this as "Required Changes to data-storage.md."

**Files:**
- Modify: `docs/design/data-storage.md` (add Migration 009 and Migration 010 sections after Migration 008)

- [ ] **Step 1: Read service-layer.md and pattern-processing.md for pattern_chunks details**

Read:
- `docs/design/service-layer.md` — look for the "Required Changes to data-storage.md" section and the `pattern_chunks` table definition
- `docs/design/pattern-processing.md` — look for chunk embedding details (2000 dimensions, HNSW index)
- `docs/architecture/dbs/08-data-architecture.md` — look for the pattern_chunks table and indexes already documented there

Use these sources to write accurate migration content.

- [ ] **Step 2: Add Migration 009 section**

After the Migration 008 section in `docs/design/data-storage.md`, add a new section:

```markdown
### Migration 009: Create Pattern Chunks Table

**Purpose:** Store semantic chunks of patterns with per-chunk embeddings, replacing pattern-level embeddings. Each chunk corresponds to a decorated section in the pattern file.

> **Architecture Reference:** [Data Architecture - Pattern Chunks](../architecture/dbs/08-data-architecture.md#tables)
```

Include the SQL for creating the `pattern_chunks` table with columns: `id` (UUID PK), `pattern_id` (FK to patterns), `chunk_index` (integer), `heading` (text), `content` (text), `embedding` (vector), `created_at`, `updated_at`. Include the HNSW index on `embedding`.

Note that this migration also removes the `embedding` column from the `patterns` table and drops the old `idx_patterns_embedding` IVFFlat index.

- [ ] **Step 3: Add Migration 010 section**

Add a section for Migration 010 that updates `pattern_chunks.embedding` to `vector(2000)` (the Matryoshka-truncated dimension for `text-embedding-3-large`).

- [ ] **Step 4: Commit**

```bash
git add docs/design/data-storage.md
git commit -m "docs: add pattern_chunks migrations (009, 010) to data-storage design"
```

---

## Task 7: Fill Content Gap — Neo4j Coverage in concept.md

The concept doc focuses heavily on embeddings/PGVector but barely mentions Neo4j, even though graph relationships are a core requirement (KM-3). Add a section explaining the knowledge graph's role.

**Files:**
- Modify: `docs/concept.md` (add section after "Embeddings explained" and before "What lives where")

- [ ] **Step 1: Add "How patterns connect to each other" section**

Before the "What lives where" section in `docs/concept.md`, add a new H2 section that explains:

1. Embeddings find patterns that are semantically similar, but Mnemonic also tracks explicit relationships between patterns via a knowledge graph (Neo4j).
2. During enrichment, Mnemonic extracts concepts (technologies, practices, domains) from each pattern and creates graph relationships: patterns that share concepts are connected.
3. This enables the `find_related_patterns` MCP tool — given a pattern, traverse the graph to find related patterns rather than relying solely on vector similarity.
4. Include a simple Mermaid diagram showing 2-3 pattern nodes connected via concept nodes.
5. Keep the tone consistent with the rest of concept.md (conversational, analogy-driven).

- [ ] **Step 2: Commit**

```bash
git add docs/concept.md
git commit -m "docs: add knowledge graph explanation to concept doc"
```

---

## Task 8: Standardize Header Metadata Across All Docs

Currently: architecture docs use source/reconciliation metadata, design docs use back-link breadcrumbs, reference docs use neither, concept.md mixes both. Standardize to a single format.

**Standard format for all docs:**

```markdown
# [Title]

> Back to [Architecture Overview](path) | [Project README](path)
```

Rules:
- **Back-links**: Every doc gets a back-link line pointing to its parent index and the project README.
- **Source metadata**: Remove "Source:" and "Last reconciled:" lines. These are internal bookkeeping — not useful to contributors and they go stale. The git history tracks provenance.
- **Table of Contents**: Required for any doc with 4+ H2 sections.

**Files:**
- Modify: Every `.md` file under `docs/architecture/`, `docs/design/`, `docs/reference/`, `docs/diagrams/`
- Modify: `docs/concept.md`, `docs/requirements.md`, `docs/requirements-traceability.md`

- [ ] **Step 1: Standardize architecture/system/ docs**

For each file in `docs/architecture/system/`:
1. Remove "Source:" and "Last reconciled:" metadata lines.
2. Add or update back-link: `> Back to [System Architecture](README.md) | [Project README](../../../README.md)`
3. If the doc has 4+ H2 sections and no ToC, add one after the back-link.

Files: `00-architectural-decisions.md`, `01-security-architecture.md`, `02-system-architecture.md`, `03-communication-patterns.md`, `04-data-architecture.md`, `05-database-integration-flow.md`, `06-deployment-architecture.md`, `07-observability-architecture.md`

- [ ] **Step 2: Standardize architecture/dbs/ docs**

Same pattern. Back-link: `> Back to [Database Architecture](README.md) | [Project README](../../../README.md)`

Files: `00-overview.md`, `02-architectural-decisions.md`, `05-deployment-architecture.md`, `08-data-architecture.md`

- [ ] **Step 3: Standardize architecture/mnemonic/ docs**

Same pattern. Back-link: `> Back to [MCP Service Architecture](README.md) | [Project README](../../../README.md)`

Files: `08-mcp-tools.md`

- [ ] **Step 4: Standardize architecture/admin/ docs**

Same pattern. Back-link: `> Back to [Admin UI Architecture](README.md) | [Project README](../../../README.md)`

Files: `0001-frontend-stack.md`, `screen-map.md`, `tech-stack-justification.md`, `use-cases.md`

- [ ] **Step 5: Standardize design/ docs**

For design docs, the back-link should point to the design index:
`> Back to [Design Documents](README.md) | [Project README](../../README.md)`

Add Table of Contents to files missing them:
- `pattern-processing.md` — has 4+ H2 sections, needs ToC
- `design-changelog.md` — has 3 H2 sections (Format, Change Log), borderline but add for consistency since the Change Log section is long

Files: all 7 non-README files in `docs/design/`

- [ ] **Step 6: Standardize reference/ and diagram docs**

Reference docs back-link: `> Back to [Project README](../../README.md)`

Diagram docs back-link: `> Back to [Project README](../../README.md)`

Files: `docs/reference/pattern-schema.md`, `docs/reference/glossary.md`, all files in `docs/diagrams/`

- [ ] **Step 7: Standardize foundation docs**

For `docs/concept.md`, `docs/requirements.md`, `docs/requirements-traceability.md`:
Back-link: `> Back to [Project README](../README.md)`

Remove source/reconciliation metadata from concept.md and requirements.md.

- [ ] **Step 8: Commit**

```bash
git add docs/
git commit -m "docs: standardize header metadata — consistent back-links, remove stale source provenance, add missing ToCs"
```

---

## Task 9: Standardize Copyright Footers

Some docs have copyright footers (with inconsistent years: 2025 vs 2026), some don't. Standardize: every doc gets a footer.

**Files:**
- Modify: All `.md` files under `docs/` that are missing footers

- [ ] **Step 1: Add/update copyright footers**

For every `.md` file under `docs/architecture/`, `docs/design/`, `docs/reference/`, `docs/diagrams/`, and the foundation docs (`concept.md`, `requirements.md`, `requirements-traceability.md`):

Ensure the file ends with:
```markdown

---

Copyright (c) 2025-2026 Jeremy K. Johnson. All rights reserved.
```

For files that already have a footer with only "2025" or only "2026", update to "2025-2026" for consistency.

- [ ] **Step 2: Commit**

```bash
git add docs/
git commit -m "docs: standardize copyright footers across all documentation"
```

---

## Task 10: Add Reading Order to design/README.md

The design README is a bare table with no guidance. Add a recommended reading order so new contributors know where to start.

**Files:**
- Modify: `docs/design/README.md`

- [ ] **Step 1: Add reading order and context**

Replace the current content of `docs/design/README.md` with an expanded version that keeps the existing table but adds:

1. A brief introduction explaining that design docs describe "how" (implementation) vs. architecture docs which describe "what" and "why."
2. A recommended reading order:
   - Start with **configuration.md** — understand how services are configured
   - Then **data-storage.md** — understand the database schema and repository pattern
   - Then **mcp-server.md** — understand how MCP tools are implemented
   - Then **service-layer.md** — understand the business logic layer that ties handlers to repositories
   - Then **pattern-processing.md** — understand the enrichment pipeline
   - Then **observability-implementation.md** — understand tracing, metrics, and logging
   - Finally **design-changelog.md** — understand how the design has evolved
3. A note that all design docs are implementation-level and assume familiarity with the architecture docs.

- [ ] **Step 2: Commit**

```bash
git add docs/design/README.md
git commit -m "docs: add reading order and context to design docs index"
```

---

## Task 11: Fix Cross-References Post-Consolidation

Architecture docs still reference "design docs in each component repo" or use stale relative paths. Now that design docs are consolidated in mnemonic-docs, update these references.

**Files:**
- Modify: Various files in `docs/architecture/` and `docs/design/`

- [ ] **Step 1: Search for stale cross-references**

Search all `.md` files under `docs/` for:
- `mnemonic/docs/design/` — stale references to design docs in source repos
- `mnemonic-api/docs/` — stale references to API docs in source repo
- `mnemonic-enricher/docs/` — stale references to enricher docs in source repo
- `../../../` or deep relative paths that may be broken after consolidation
- Any link containing `#` anchors — verify the target heading exists

Record all findings with file paths and line numbers.

- [ ] **Step 2: Update stale references**

For each stale reference found:
- If the target doc now exists in mnemonic-docs, update the path to the correct relative location.
- If the target doc doesn't exist in mnemonic-docs (e.g., OpenAPI specs that remain in source repos), keep the reference but add a note: "(in source repository)".
- Fix any broken anchor links by updating to match actual heading text.

- [ ] **Step 3: Verify architecture/system/README.md MCP tools link**

The system README references `../mnemonic/08-mcp-tools.md`. Verify this relative path is correct from `docs/architecture/system/README.md` to `docs/architecture/mnemonic/08-mcp-tools.md`.

- [ ] **Step 4: Verify requirements-traceability.md anchor links**

The traceability matrix uses many anchor links like `02-system-architecture.md#data-flow`. Read the target files and verify that the referenced section headings actually exist. Fix any mismatches.

- [ ] **Step 5: Commit**

```bash
git add docs/
git commit -m "docs: fix cross-references for consolidated documentation structure"
```

---

## Task 12: Update Diagram Section in Root README

The root README's diagram section lists `docs/diagrams/concept.md` (now deleted in Task 4) and uses inconsistent naming. Update to clearly present the MVP evolution sequence.

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Rewrite the Diagrams section**

Replace the current Diagrams section in `README.md` with a clearer presentation of the evolution sequence:

```markdown
### Diagrams

The system evolves through five iterations from local monolith to cloud-native deployment:

| Iteration | Document | Description |
|-----------|----------|-------------|
| MVP 1 | [01-concept-local-iteration.md](docs/diagrams/01-concept-local-iteration.md) | Monolithic local service with PostgreSQL + Neo4j |
| MVP 2 | [02-concept-local-iteration.md](docs/diagrams/02-concept-local-iteration.md) | Service decomposition (MCP, Admin API, Enricher) + RabbitMQ |
| MVP 3 | [03-concept-local-iteration.md](docs/diagrams/03-concept-local-iteration.md) | Local services with AWS-managed infrastructure |
| MVP 4 | [04-concept-local-iteration.md](docs/diagrams/04-concept-local-iteration.md) | Local Kubernetes with Envoy + OPA authentication |
| MVP 5 | [05-concept-cloud-iteration.md](docs/diagrams/05-concept-cloud-iteration.md) | AWS Fargate deployment with Terraform automation |

Additional diagrams:

- [POST /v1/api/patterns Flow](docs/diagrams/post-pattern-flow.md) — End-to-end pattern creation activity diagram
- [Go CI/CD Pipeline](docs/diagrams/example-go-ci-cd-diagram.md) — Docker-first build pipeline for Go services
```

- [ ] **Step 2: Commit**

```bash
git add README.md
git commit -m "docs: update README diagrams section with clear MVP evolution sequence"
```

---

## Task 13: Final Verification Pass

Read through the entire documentation set one more time to verify all changes are consistent and no new issues were introduced.

- [ ] **Step 1: Verify all internal links**

Run a link check across all `.md` files under `docs/`. For each relative link `[text](path)` or `[text](path#anchor)`, verify the target file exists and (if anchored) the heading exists.

Use grep to find all markdown links, then spot-check the top 20 most-referenced targets.

- [ ] **Step 2: Verify glossary completeness**

Read through `docs/architecture/mnemonic/08-mcp-tools.md` and `docs/design/pattern-processing.md` and verify that every technical term used is either:
- Defined in the glossary, OR
- A standard industry term that needs no definition (e.g., PostgreSQL, Docker, HTTP)

- [ ] **Step 3: Verify reading path coherence**

Simulate the new-contributor reading path:
1. `README.md` → `docs/concept.md` → `docs/requirements.md`
2. `docs/architecture/system/README.md` → system architecture docs
3. `docs/design/README.md` → design docs in recommended order

At each transition, verify that back-links work and the reader isn't left without navigation.

- [ ] **Step 4: Final commit if any fixes needed**

```bash
git add docs/
git commit -m "docs: final verification fixes"
```
