# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.3.0] - 2026-09-16

### Added

- `Makefile` with a self-documenting `help` target and an `analyze` target that runs markdownlint over all tracked Markdown files
- CC-BY-4.0 license

### Changed

- Documentation now records `mnemonic-mcp` as the sole owner of MCP tooling (README and system architecture)
- README: added `mnemonic-patterns` to the repository table and Quick Start directory layout; corrected "five source repositories" to "six"
- README: repointed the vision link from the deleted `PURPOSE.md` to `docs/concept.md`
- README: realigned Markdown tables (MD060) and tagged the directory-tree fence as `text` (MD040)
- Quickstart guide clarified with a clone step and a patterns-loading tip
- `.gitignore`: added `**/code-reviews/` and `**/code_reviews/` so local review artifacts are not committed

### Removed

- `PURPOSE.md` — its GOAL section was already stated in the README, and its PROBLEM section had gone stale, describing the repository split as still in progress
- Planning artifacts: `docs/plans/` and `plans/stale-docs-cleanup/`
- `docs/code_reviews/` taken out of version control; the file remains on disk

### Fixed

- Five broken relative links left by the reorganization into `system/`, `dbs/`, and `mnemonic/` subfolders: two in `docs/design/data-storage.md`, three in `docs/design/mcp-server.md`

## [0.2.0] - 2026-03-26

Large rewrite for MVP 2 decomposed three-service topology.

### Added

- Admin UI design documentation: screen map, use cases, and tech stack justification
- Guides: Introduction, Concepts, Quickstart, and Authoring Patterns
- Pattern file schema reference documentation
- Glossary of key terms used across solution
- Amendment notes to ADR-002 and ADR-004 reflecting MVP 2 decomposition

### Changed

**System Architecture (for three-service topology):**

- System architecture rewritten for decomposed MCP, Admin API, and Enricher services
- Deployment architecture rewritten for multi-service topology
- Observability architecture updated for three independent services
- Security architecture updated for three-service model
- Database integration flow redesigned for service-to-database communication
- Communication patterns documented for service interactions
- Dbs architecture updated to reference all three consuming services

**Service Design:**

- Enrichment worker redesigned as standalone RabbitMQ-based service
- Configuration design rewritten for per-service configs with Viper and environment variables
- Service layer design updated for three-service codebase split

**Data & Observability:**

- Data architecture updated for decomposed service diagram and enrichment flow
- Embedding model references updated from text-embedding-3-small/1536 to text-embedding-3-large/2000
- Default search threshold corrected from 0.7 to 0.5

**Documentation Structure:**

- Cross-references fixed for consolidated documentation structure
- Back-links standardized across all architecture documents
- Copyright footers standardized
- Table of contents added to design documents
- Reading order and context added to design docs index

### Fixed

- Factual errors: typos, stale embedding dimensions, deprecated index names, macOS paths
- Data-storage migration section updated to reflect Docker entrypoint approach (not migration tool)
- Pre-pivot routing engine references removed from observability design
- Duplicate concept.md diagram removed (02-concept-local-iteration.md is canonical)
- Markdown table alignment issues corrected
- Broken cross-references and anchors throughout documentation
- Stale imports and monolith language references

## [0.1.0] - 2026-03-23

### Added

- Initial consolidation of architecture, design, requirements, and reference documentation from source repositories
- Architecture documentation: system, databases, MCP server, and admin UI design
- Design documentation: configuration, data storage, enrichment flow, observability, pattern processing, and service layers
- Requirements and requirements traceability documentation
- Diagrams documenting MVP evolution (1-5) and system flow patterns
- .gitignore file

[Unreleased]: https://github.com/twistingmercury/mnemonic-docs/compare/v0.3.0...HEAD
[0.3.0]: https://github.com/twistingmercury/mnemonic-docs/releases/tag/v0.3.0
[0.2.0]: https://github.com/twistingmercury/mnemonic-docs/releases/tag/v0.2.0
[0.1.0]: https://github.com/twistingmercury/mnemonic-docs/releases/tag/v0.1.0
