# Documentation Consolidation Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Consolidate all architecture, design, and diagram documentation into mnemonic-docs and delete duplicates from source repos.

**Architecture:** Each source repo (`mnemonic-mcp`, `mnemonic-api`, `mnemonic-enricher`, `mnemonic-dbs`, `mnemonic-admin`, `mnemonic-patterns`) currently has its own `docs/` tree with architecture, design, and diagram files. Most are duplicates of what's already in `mnemonic-docs`. Unique design docs live only in `mnemonic-mcp` (the other Go services have identical copies). We move unique content to `mnemonic-docs`, delete all duplicate docs trees from source repos, and update the mnemonic-docs README to reflect the new `design/` and `reference/` sections.

**Tech Stack:** Shell (cp, rm), git

**Workspace root:** `/home/jeremy/dev/mnemonic/`

**Repos involved:** `mnemonic-docs`, `mnemonic-mcp`, `mnemonic-api`, `mnemonic-enricher`, `mnemonic-dbs`, `mnemonic-admin`, `mnemonic-patterns`

---

## File Structure

New directories and files in `mnemonic-docs`:

```
mnemonic-docs/docs/
├── design/                              # NEW directory
│   ├── README.md                        # NEW — navigation for design docs
│   ├── configuration.md                 # MOVE from mnemonic-mcp/docs/design/
│   ├── data-storage.md                  # MOVE from mnemonic-mcp/docs/design/
│   ├── design-changelog.md              # MOVE from mnemonic-mcp/docs/design/
│   ├── mcp-server.md                    # MOVE from mnemonic-mcp/docs/design/
│   ├── observability-implementation.md  # MOVE from mnemonic-mcp/docs/design/
│   ├── pattern-processing.md            # MOVE from mnemonic-mcp/docs/design/
│   └── service-layer.md                 # MOVE from mnemonic-mcp/docs/design/
├── reference/                           # NEW directory
│   └── pattern-schema.md               # MOVE from mnemonic-patterns/docs/pattern-file-schema.md
├── architecture/
│   └── admin/
│       ├── screen-map.md               # MOVE from mnemonic-admin/docs/design/pattern-ui-screen-map.md
│       ├── use-cases.md                # MOVE from mnemonic-admin/docs/design/pattern-ui-usecase-diagram.md
│       └── tech-stack-justification.md # MOVE from mnemonic-admin/docs/design/tech-stack-justification.md
└── (existing files unchanged)
```

Deletions from source repos — entire `docs/` trees except for `plans/`, `code-reviews/`, and `superpowers/` subdirectories (historical artifacts kept in source repos).

---

## Task 1: Move unique design docs from mnemonic-mcp to mnemonic-docs

**Files:**
- Create: `mnemonic-docs/docs/design/` directory
- Copy: 7 files from `mnemonic-mcp/docs/design/` → `mnemonic-docs/docs/design/`
- Create: `mnemonic-docs/docs/design/README.md`

- [ ] **Step 1: Create design directory and copy files**

```bash
cd /home/jeremy/dev/mnemonic
mkdir -p mnemonic-docs/docs/design
cp mnemonic-mcp/docs/design/configuration.md mnemonic-docs/docs/design/
cp mnemonic-mcp/docs/design/data-storage.md mnemonic-docs/docs/design/
cp mnemonic-mcp/docs/design/design-changelog.md mnemonic-docs/docs/design/
cp mnemonic-mcp/docs/design/mcp-server.md mnemonic-docs/docs/design/
cp mnemonic-mcp/docs/design/observability-implementation.md mnemonic-docs/docs/design/
cp mnemonic-mcp/docs/design/pattern-processing.md mnemonic-docs/docs/design/
cp mnemonic-mcp/docs/design/service-layer.md mnemonic-docs/docs/design/
```

- [ ] **Step 2: Create design README**

Create `mnemonic-docs/docs/design/README.md`:

```markdown
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
```

- [ ] **Step 3: Verify all 8 files exist in mnemonic-docs**

```bash
ls -la mnemonic-docs/docs/design/
```

Expected: 8 files (7 design docs + README.md).

- [ ] **Step 4: Commit in mnemonic-docs**

```bash
cd /home/jeremy/dev/mnemonic/mnemonic-docs
git add docs/design/
git commit -m "docs: add design documents consolidated from source repos

Move 7 cross-cutting design documents from mnemonic-mcp/docs/design/ to
establish mnemonic-docs as the single source of truth. These cover
configuration, data storage, MCP server implementation, observability,
pattern processing, and service layer patterns shared by all Go services."
```

---

## Task 2: Move unique admin design docs to mnemonic-docs

**Files:**
- Copy: 3 files from `mnemonic-admin/docs/design/` → `mnemonic-docs/docs/architecture/admin/`

- [ ] **Step 1: Copy admin design files**

```bash
cd /home/jeremy/dev/mnemonic
cp mnemonic-admin/docs/design/pattern-ui-screen-map.md mnemonic-docs/docs/architecture/admin/screen-map.md
cp mnemonic-admin/docs/design/pattern-ui-usecase-diagram.md mnemonic-docs/docs/architecture/admin/use-cases.md
cp mnemonic-admin/docs/design/tech-stack-justification.md mnemonic-docs/docs/architecture/admin/tech-stack-justification.md
```

- [ ] **Step 2: Update admin architecture README**

Edit `mnemonic-docs/docs/architecture/admin/README.md` to add the new files to the document listing. Add rows for `screen-map.md`, `use-cases.md`, and `tech-stack-justification.md`.

- [ ] **Step 3: Verify**

```bash
ls -la mnemonic-docs/docs/architecture/admin/
```

Expected: 5 files (README.md, 0001-frontend-stack.md, screen-map.md, use-cases.md, tech-stack-justification.md).

- [ ] **Step 4: Commit in mnemonic-docs**

```bash
cd /home/jeremy/dev/mnemonic/mnemonic-docs
git add docs/architecture/admin/
git commit -m "docs: add admin UI design docs (screen map, use cases, tech stack)

Move wireframes, use case diagrams, and tech stack justification from
mnemonic-admin/docs/design/ into the admin architecture section."
```

---

## Task 3: Move pattern schema reference to mnemonic-docs

**Files:**
- Create: `mnemonic-docs/docs/reference/` directory
- Copy: `mnemonic-patterns/docs/pattern-file-schema.md` → `mnemonic-docs/docs/reference/pattern-schema.md`

- [ ] **Step 1: Create reference directory and copy file**

```bash
cd /home/jeremy/dev/mnemonic
mkdir -p mnemonic-docs/docs/reference
cp mnemonic-patterns/docs/pattern-file-schema.md mnemonic-docs/docs/reference/pattern-schema.md
```

- [ ] **Step 2: Verify**

```bash
ls -la mnemonic-docs/docs/reference/
```

Expected: 1 file (pattern-schema.md).

- [ ] **Step 3: Commit in mnemonic-docs**

```bash
cd /home/jeremy/dev/mnemonic/mnemonic-docs
git add docs/reference/
git commit -m "docs: add pattern file schema reference

Move canonical pattern schema spec from mnemonic-patterns/docs/ into
a new reference section. This is the normative spec for pattern file
frontmatter, body structure, chunking rules, and validation."
```

---

## Task 4: Update mnemonic-docs README

**Files:**
- Modify: `mnemonic-docs/README.md`

- [ ] **Step 1: Add Design and Reference sections to README**

Edit `mnemonic-docs/README.md` to add entries for the new sections. After the existing "### Architecture" section, add:

```markdown
### Design

Implementation-level design documents shared across Go services:

- **[Design Overview](docs/design/README.md)** — Configuration, data storage, MCP server, observability, pattern processing, service layer

### Reference

- **[Pattern File Schema](docs/reference/pattern-schema.md)** — Normative specification for pattern file structure, frontmatter, and chunking
```

Also update the "### Architecture" table to mention the new admin docs:

```markdown
| `docs/architecture/admin/`    | Frontend stack, ADRs, screen map, use cases, tech stack justification        |
```

- [ ] **Step 2: Commit in mnemonic-docs**

```bash
cd /home/jeremy/dev/mnemonic/mnemonic-docs
git add README.md
git commit -m "docs: update README with design and reference sections"
```

---

## Task 5: Delete duplicate docs from mnemonic-mcp

**Files:**
- Delete: `mnemonic-mcp/docs/architecture/` (00-08, README.md — 10 files; keep `review-*.md`)
- Delete: `mnemonic-mcp/docs/design/` (all 7 files — now in mnemonic-docs)
- Delete: `mnemonic-mcp/docs/diagrams/` (all 7 files)
- Delete: `mnemonic-mcp/docs/mnemonic-concept.md`
- Delete: `mnemonic-mcp/docs/mnemonic-requirements.md`
- Delete: `mnemonic-mcp/docs/requirements-traceability.md`

- [ ] **Step 1: Delete architecture duplicates (keep review files)**

```bash
cd /home/jeremy/dev/mnemonic/mnemonic-mcp
rm docs/architecture/00-architectural-decisions.md
rm docs/architecture/01-security-architecture.md
rm docs/architecture/02-system-architecture.md
rm docs/architecture/03-communication-patterns.md
rm docs/architecture/04-data-architecture.md
rm docs/architecture/05-database-integration-flow.md
rm docs/architecture/06-deployment-architecture.md
rm docs/architecture/07-observability-architecture.md
rm docs/architecture/08-mcp-tools.md
rm docs/architecture/README.md
```

- [ ] **Step 2: Delete design docs (now in mnemonic-docs)**

```bash
rm -r docs/design/
```

- [ ] **Step 3: Delete diagram duplicates**

```bash
rm -r docs/diagrams/
```

- [ ] **Step 4: Delete concept/requirements duplicates**

```bash
rm docs/mnemonic-concept.md
rm docs/mnemonic-requirements.md
rm docs/requirements-traceability.md
```

- [ ] **Step 5: Verify remaining docs structure**

```bash
find docs/ -name '*.md' | sort
```

Expected remaining files — only historical artifacts:
```
docs/architecture/review-cycles-7-12-vocabulary.md
docs/architecture/review-pivot-phases-9-13.md
docs/code-reviews/feature-dynamic-lang-and-domain.md
docs/code-reviews/feature-dynamic-lang-and-domain-vocab.md
docs/code-reviews/feature-improved-chunking.md
docs/code-reviews/feature-pattern-chunking.md
docs/code-reviews/openapi-compliance.md
docs/plans/2026-02-27-pattern-schema-chunks-design.md
docs/plans/2026-02-27-pattern-schema-chunks.md
docs/plans/2026-03-02-openapi-spec-compliance.md
docs/plans/e2e-test-implementation.md
docs/plans/phase-02/part-01/PRD.md
docs/plans/phase-02/part-01/PROMPT.md
docs/plans/phase-03/PRD.md
docs/plans/phase-03/PROMPT.md
docs/superpowers/plans/2026-03-12-enriched-chunk-embedding.md
docs/superpowers/specs/2026-03-24-graph-enhanced-search-design.md
```

- [ ] **Step 6: Commit in mnemonic-mcp**

```bash
git add -A docs/
git commit -m "docs: remove architecture, design, and diagram docs

These are now maintained in the mnemonic-docs repository, which is
the single source of truth for all Mnemonic documentation.

Historical artifacts (plans, code reviews, architecture reviews)
are retained in this repo."
```

---

## Task 6: Delete duplicate docs from mnemonic-api

**Files:**
- Delete: `mnemonic-api/docs/architecture/` (00-08, README.md — 10 files; keep `review-*.md`)
- Delete: `mnemonic-api/docs/design/` (all 7 files)
- Delete: `mnemonic-api/docs/diagrams/` (all 3 files)
- Delete: `mnemonic-api/docs/mnemonic-concept.md`
- Delete: `mnemonic-api/docs/mnemonic-requirements.md`

- [ ] **Step 1: Delete architecture duplicates (keep review files)**

```bash
cd /home/jeremy/dev/mnemonic/mnemonic-api
rm docs/architecture/00-architectural-decisions.md
rm docs/architecture/01-security-architecture.md
rm docs/architecture/02-system-architecture.md
rm docs/architecture/03-communication-patterns.md
rm docs/architecture/04-data-architecture.md
rm docs/architecture/05-database-integration-flow.md
rm docs/architecture/06-deployment-architecture.md
rm docs/architecture/07-observability-architecture.md
rm docs/architecture/08-mcp-tools.md
rm docs/architecture/README.md
```

- [ ] **Step 2: Delete design, diagrams, concept/requirements**

```bash
rm -r docs/design/
rm -r docs/diagrams/
rm docs/mnemonic-concept.md
rm docs/mnemonic-requirements.md
```

- [ ] **Step 3: Verify remaining docs structure**

```bash
find docs/ -name '*.md' -o -name '*.yaml' | sort
```

Expected remaining:
```
docs/architecture/review-cycles-7-12-vocabulary.md
docs/architecture/review-phase-02-fix-cycles-1-5.md
docs/architecture/review-phase-02-queue-publishing.md
docs/architecture/review-pivot-phases-9-13.md
docs/code-reviews/phase-02-fix-cycles.md
docs/code-reviews/phase-02-queue-publishing.md
docs/openapi/mnemonic-v1.yaml
docs/plans/e2e-test-implementation.md
docs/plans/phase-01/PRD.md
docs/plans/phase-01/PROMPT.md
docs/plans/phase-02-code-review/PRD.md
docs/plans/phase-02-code-review/PROMPT.md
docs/plans/phase-02-code-review/plan-remaining-items.md
docs/plans/phase-02/PRD.md
docs/plans/phase-02/PROMPT.md
```

- [ ] **Step 4: Commit in mnemonic-api**

```bash
git add -A docs/
git commit -m "docs: remove architecture, design, and diagram docs

These are now maintained in the mnemonic-docs repository, which is
the single source of truth for all Mnemonic documentation.

Historical artifacts (plans, code reviews, architecture reviews, OpenAPI spec)
are retained in this repo."
```

---

## Task 7: Delete duplicate docs from mnemonic-enricher

**Files:**
- Delete: `mnemonic-enricher/docs/architecture/` (00-08, README.md — 10 files; keep `review-*.md`)
- Delete: `mnemonic-enricher/docs/design/` (all 7 files)
- Delete: `mnemonic-enricher/docs/diagrams/` (all 7 files)
- Delete: `mnemonic-enricher/docs/mnemonic-concept.md`
- Delete: `mnemonic-enricher/docs/mnemonic-requirements.md`
- Delete: `mnemonic-enricher/docs/requirements-traceability.md`

- [ ] **Step 1: Delete architecture duplicates (keep review files)**

```bash
cd /home/jeremy/dev/mnemonic/mnemonic-enricher
rm docs/architecture/00-architectural-decisions.md
rm docs/architecture/01-security-architecture.md
rm docs/architecture/02-system-architecture.md
rm docs/architecture/03-communication-patterns.md
rm docs/architecture/04-data-architecture.md
rm docs/architecture/05-database-integration-flow.md
rm docs/architecture/06-deployment-architecture.md
rm docs/architecture/07-observability-architecture.md
rm docs/architecture/08-mcp-tools.md
rm docs/architecture/README.md
```

- [ ] **Step 2: Delete design, diagrams, concept/requirements**

```bash
rm -r docs/design/
rm -r docs/diagrams/
rm docs/mnemonic-concept.md
rm docs/mnemonic-requirements.md
rm docs/requirements-traceability.md
```

- [ ] **Step 3: Verify remaining docs structure**

```bash
find docs/ -name '*.md' | sort
```

Expected remaining:
```
docs/architecture/review-cycles-7-12-vocabulary.md
docs/architecture/review-pivot-phases-9-13.md
docs/code-reviews/feature-dynamic-lang-and-domain.md
docs/code-reviews/feature-dynamic-lang-and-domain-vocab.md
docs/code-reviews/feature-improved-chunking.md
docs/code-reviews/feature-pattern-chunking.md
docs/code-reviews/openapi-compliance.md
docs/code-reviews/phase-01-cleanup-and-hardening.md
docs/code-reviews/phase-02-e2e-test-overhaul.md
docs/plans/phase-01/PRD.md
docs/plans/phase-01/PROMPT.md
docs/plans/phase-01/phase-01.md
docs/plans/phase-02/PRD.md
docs/plans/phase-02/PROMPT.md
docs/superpowers/plans/2026-03-21-e2e-test-rebuild.md
docs/superpowers/specs/2026-03-20-e2e-test-fix-design.md
docs/superpowers/specs/2026-03-21-e2e-test-rebuild-design.md
```

- [ ] **Step 4: Commit in mnemonic-enricher**

```bash
git add -A docs/
git commit -m "docs: remove architecture, design, and diagram docs

These are now maintained in the mnemonic-docs repository, which is
the single source of truth for all Mnemonic documentation.

Historical artifacts (plans, code reviews, architecture reviews)
are retained in this repo."
```

---

## Task 8: Delete duplicate docs from mnemonic-dbs

**Files:**
- Delete: `mnemonic-dbs/docs/architecture/` (all 5 files — entire directory)

- [ ] **Step 1: Delete entire architecture directory**

```bash
cd /home/jeremy/dev/mnemonic/mnemonic-dbs
rm -r docs/architecture/
```

- [ ] **Step 2: Check if docs/ directory is now empty and remove if so**

```bash
find docs/ -type f 2>/dev/null | head -5
```

If empty, remove the docs directory:

```bash
rmdir docs/
```

- [ ] **Step 3: Commit in mnemonic-dbs**

```bash
git add -A docs/
git commit -m "docs: remove architecture docs

These are now maintained in the mnemonic-docs repository
(docs/architecture/dbs/) as the single source of truth."
```

---

## Task 9: Delete duplicate docs from mnemonic-admin

**Files:**
- Delete: `mnemonic-admin/docs/adr/` (1 file — now in mnemonic-docs)
- Delete: `mnemonic-admin/docs/design/` (3 files — now in mnemonic-docs)

- [ ] **Step 1: Delete ADR and design directories**

```bash
cd /home/jeremy/dev/mnemonic/mnemonic-admin
rm -r docs/adr/
rm -r docs/design/
```

- [ ] **Step 2: Verify remaining docs structure**

```bash
find docs/ -type f | sort
```

Expected remaining — only plans:
```
docs/plans/phase-01/PRD.md
docs/plans/phase-01/PROMPT.md
docs/plans/phase-01/progress.txt
docs/plans/phase-02/PRD.md
docs/plans/phase-02/PROMPT.md
docs/plans/phase-02/mnemonic-api-v1.json
docs/plans/phase-02/progress.txt
```

- [ ] **Step 3: Commit in mnemonic-admin**

```bash
git add -A docs/
git commit -m "docs: remove ADR and design docs

These are now maintained in the mnemonic-docs repository
(docs/architecture/admin/) as the single source of truth."
```

---

## Task 10: Delete duplicate docs from mnemonic-patterns

**Files:**
- Delete: `mnemonic-patterns/docs/pattern-file-schema.md` (now in mnemonic-docs/reference/)
- Delete: `mnemonic-patterns/docs/authoring-patterns.md` (superseded by mnemonic-docs/guides/05-authoring-patterns.md)

- [ ] **Step 1: Delete docs files**

```bash
cd /home/jeremy/dev/mnemonic/mnemonic-patterns
rm docs/authoring-patterns.md
rm docs/pattern-file-schema.md
```

- [ ] **Step 2: Check if docs/ directory is now empty and remove if so**

```bash
find docs/ -type f 2>/dev/null | head -5
```

If empty:

```bash
rmdir docs/
```

- [ ] **Step 3: Commit in mnemonic-patterns**

```bash
git add -A docs/
git commit -m "docs: remove authoring guide and schema reference

The authoring guide is superseded by mnemonic-docs/docs/guides/05-authoring-patterns.md.
The pattern schema reference is now at mnemonic-docs/docs/reference/pattern-schema.md."
```

---

## Task 11: Update CLAUDE.md in workspace root

**Files:**
- Modify: `/home/jeremy/dev/mnemonic/CLAUDE.md`

- [ ] **Step 1: Update the Repository Structure table**

Add `mnemonic-docs/` row if not already present, and note that architecture/design docs live there exclusively:

In the "## Repository Structure" table, ensure `mnemonic-docs` is listed and update descriptions of `docs/` content in other repos to mention that architecture docs are in `mnemonic-docs`.

- [ ] **Step 2: Commit in mnemonic root (if CLAUDE.md is tracked)**

If CLAUDE.md is tracked by git:

```bash
cd /home/jeremy/dev/mnemonic
git add CLAUDE.md
git commit -m "docs: update CLAUDE.md to reflect docs consolidation"
```

If it's not tracked (workspace-level file), skip this step.
