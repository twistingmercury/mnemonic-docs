# Product Requirements Document: Mnemonic Docs Initial Consolidation

*Gralph processes cycles in this document from top to bottom. Checklist markers are significant: `- [ ]` (open), `- [x]` (complete), `- [~]` (abandoned). Each cycle must be small, independently verifiable, and assigned to exactly one agent.*

## Objective

Consolidate all canonical reference documentation from the five mnemonic repositories into `mnemonic-docs`, producing a single authoritative source of truth for architecture, diagrams, concept, and requirements. No edits are made to source repos.

## Problem Statement

The mnemonic solution was split into five repos. Documentation was copied across them and has since diverged. There is no authoritative version. Engineers must consult multiple repos to understand the full system, and conflicting content exists with no resolution.

## Success Criteria

- All canonical reference docs exist at their target paths in `mnemonic-docs/docs/`
- All reconciled docs have a header note identifying their source repos
- No internal cross-links in consolidated docs reference source-repo paths
- `mnemonic-docs` has a top-level `README.md` describing the project and doc structure

## Scope

### In scope

- Top-level system docs: concept, requirements, requirements-traceability
- Architecture docs 00–07 (system-wide, reconciled from mnemonic + mnemonic-api + mnemonic-enricher)
- Architecture doc 08-mcp-tools (MCP-specific, reconciled into architecture/mnemonic/)
- mnemonic-dbs architecture docs (single-source copy)
- mnemonic-admin architecture ADR (single-source copy)
- Diagrams (deduplicated across mnemonic + mnemonic-api + mnemonic-enricher)
- Internal cross-link fixes
- Top-level README.md

### Out of scope

- Design docs (remain in each component repo)
- Process artifacts: plans, code-reviews, _ideas, superpowers plans
- review-cycles-* and review-pivot-* files (process artifacts, not canonical reference docs)
- Any edits to source repositories

## Constraints and Decisions

- Source repos are read-only; all output goes to `mnemonic-docs/`
- Use `mnemonic` as the baseline when a doc exists across all three shared repos
- Reconciliation heuristics: component-internal references → component subdir; system-wide behavior → system/; conflicting content → use most recently modified version
- Conflict escalation: when classification is ambiguous, default to system/ and add `<!-- REVIEW: origin ambiguous -->` inline
- Header note format: blockquote with Source, Component-specific divergences (if any), and Last reconciled date
- Commits must be signed (`git commit -S`)

## Implementation Plan

- [x] **Cycle 1 - Top-level system docs**: Reconcile concept.md, requirements.md, and requirements-traceability.md from source repos into docs/.
  - Agent: `technical writer`
  - Files: `docs/concept.md`, `docs/requirements.md`, `docs/requirements-traceability.md`
  - Steps:
    - Read `mnemonic/docs/mnemonic-concept.md`, `mnemonic-api/docs/mnemonic-concept.md`, `mnemonic-enricher/docs/mnemonic-concept.md`; diff and reconcile into `docs/concept.md`; add header note
    - Read `mnemonic/docs/mnemonic-requirements.md`, `mnemonic-api/docs/mnemonic-requirements.md`, `mnemonic-enricher/docs/mnemonic-requirements.md`; diff and reconcile into `docs/requirements.md`; add header note
    - Read `mnemonic/docs/requirements-traceability.md`, `mnemonic-enricher/docs/requirements-traceability.md`; diff and reconcile into `docs/requirements-traceability.md`; add header note
  - Verify: `ls docs/concept.md docs/requirements.md docs/requirements-traceability.md`
  - Done: All three files exist at target paths and each contains a blockquote header note with Source field

- [x] **Cycle 2 - Architecture docs 00–02**: Reconcile architectural-decisions, security-architecture, and system-architecture into docs/architecture/system/; extract any component-specific content.
  - Agent: `technical writer`
  - Files: `docs/architecture/system/00-architectural-decisions.md`, `docs/architecture/system/01-security-architecture.md`, `docs/architecture/system/02-system-architecture.md`, `docs/architecture/system/README.md`
  - Steps:
    - Read each of 00, 01, 02 across mnemonic, mnemonic-api, mnemonic-enricher; diff each pair
    - Apply reconciliation heuristics: system-wide content → system/; component-specific content → docs/architecture/api/, docs/architecture/enricher/, or docs/architecture/mnemonic/ as appropriate
    - Write reconciled docs to docs/architecture/system/; add header notes
    - Reconcile architecture README.md from the three source repos; write to docs/architecture/system/README.md
  - Verify: `ls docs/architecture/system/00-architectural-decisions.md docs/architecture/system/01-security-architecture.md docs/architecture/system/02-system-architecture.md docs/architecture/system/README.md`
  - Done: All four files exist at target paths with header notes

- [x] **Cycle 3 - Architecture docs 03 and 05**: Reconcile communication-patterns and database-integration-flow into docs/architecture/system/.
  - Agent: `technical writer`
  - Files: `docs/architecture/system/03-communication-patterns.md`, `docs/architecture/system/05-database-integration-flow.md`
  - Steps:
    - Read 03-communication-patterns.md across mnemonic, mnemonic-api, mnemonic-enricher; diff and reconcile into docs/architecture/system/03-communication-patterns.md; add header note; extract component-specific content if found
    - Read 05-database-integration-flow.md across mnemonic, mnemonic-api, mnemonic-enricher; diff and reconcile into docs/architecture/system/05-database-integration-flow.md; add header note; extract component-specific content if found
  - Verify: `ls docs/architecture/system/03-communication-patterns.md docs/architecture/system/05-database-integration-flow.md`
  - Done: Both files exist at target paths with header notes

- [x] **Cycle 4 - Architecture doc 04 (data architecture)**: Reconcile data-architecture (the largest doc, 1,092 lines) into docs/architecture/system/.
  - Agent: `technical writer`
  - Files: `docs/architecture/system/04-data-architecture.md`
  - Steps:
    - Read 04-data-architecture.md from mnemonic, mnemonic-api, and mnemonic-enricher
    - Diff all three versions carefully; classify each section as system-level or component-specific
    - Write reconciled system-level content to docs/architecture/system/04-data-architecture.md; add header note
    - Extract any component-specific sections to docs/architecture/api/, docs/architecture/enricher/, or docs/architecture/mnemonic/ as appropriate; add header notes to extracted files
  - Verify: `ls docs/architecture/system/04-data-architecture.md`
  - Done: File exists at target path with header note; any component-specific extracts have been written to their component subdirs

- [x] **Cycle 5 - Architecture docs 06–07**: Reconcile deployment-architecture and observability-architecture into docs/architecture/system/.
  - Agent: `technical writer`
  - Files: `docs/architecture/system/06-deployment-architecture.md`, `docs/architecture/system/07-observability-architecture.md`
  - Steps:
    - Read 06-deployment-architecture.md across mnemonic, mnemonic-api, mnemonic-enricher; diff and reconcile; write to docs/architecture/system/; add header note; extract component-specific content if found
    - Read 07-observability-architecture.md across mnemonic, mnemonic-api, mnemonic-enricher; diff and reconcile; write to docs/architecture/system/; add header note; extract component-specific content if found
  - Verify: `ls docs/architecture/system/06-deployment-architecture.md docs/architecture/system/07-observability-architecture.md`
  - Done: Both files exist at target paths with header notes

- [x] **Cycle 6 - Architecture doc 08 (MCP tools)**: Reconcile 08-mcp-tools.md into docs/architecture/mnemonic/.
  - Agent: `technical writer`
  - Files: `docs/architecture/mnemonic/08-mcp-tools.md`, `docs/architecture/mnemonic/README.md`
  - Steps:
    - Read 08-mcp-tools.md from mnemonic, mnemonic-api, and mnemonic-enricher; diff all three
    - Reconcile into docs/architecture/mnemonic/08-mcp-tools.md; add header note
    - Write a brief docs/architecture/mnemonic/README.md explaining this subdir covers the MCP service component
  - Verify: `ls docs/architecture/mnemonic/08-mcp-tools.md docs/architecture/mnemonic/README.md`
  - Done: Both files exist with header notes

- [x] **Cycle 7 - mnemonic-dbs architecture**: Copy mnemonic-dbs architecture docs (single source) to docs/architecture/dbs/.
  - Agent: `technical writer`
  - Files: `docs/architecture/dbs/00-overview.md`, `docs/architecture/dbs/02-architectural-decisions.md`, `docs/architecture/dbs/05-deployment-architecture.md`, `docs/architecture/dbs/08-data-architecture.md`, `docs/architecture/dbs/README.md`
  - Steps:
    - Copy each of the four architecture docs from mnemonic-dbs/docs/architecture/ to docs/architecture/dbs/; update any internal cross-links to point to paths in mnemonic-docs
    - Add single-source header note to each file
    - Copy or create docs/architecture/dbs/README.md from mnemonic-dbs/docs/architecture/README.md
  - Verify: `ls docs/architecture/dbs/00-overview.md docs/architecture/dbs/02-architectural-decisions.md docs/architecture/dbs/05-deployment-architecture.md docs/architecture/dbs/08-data-architecture.md docs/architecture/dbs/README.md`
  - Done: All five files exist at target paths with header notes

- [x] **Cycle 8 - mnemonic-admin architecture**: Copy mnemonic-admin ADR (single source) to docs/architecture/admin/.
  - Agent: `technical writer`
  - Files: `docs/architecture/admin/0001-frontend-stack.md`, `docs/architecture/admin/README.md`
  - Steps:
    - Copy mnemonic-admin/docs/adr/0001-frontend-stack.md to docs/architecture/admin/0001-frontend-stack.md; update any internal cross-links
    - Add single-source header note
    - Write a brief docs/architecture/admin/README.md explaining this subdir covers the admin web UI component
  - Verify: `ls docs/architecture/admin/0001-frontend-stack.md docs/architecture/admin/README.md`
  - Done: Both files exist with header notes

- [x] **Cycle 9 - Diagrams**: Deduplicate and consolidate diagrams from mnemonic, mnemonic-api, and mnemonic-enricher into docs/diagrams/.
  - Agent: `technical writer`
  - Files: `docs/diagrams/01-concept-local-iteration.md`, `docs/diagrams/02-concept-local-iteration.md`, `docs/diagrams/03-concept-local-iteration.md`, `docs/diagrams/04-concept-local-iteration.md`, `docs/diagrams/05-concept-cloud-iteration.md`, `docs/diagrams/example-go-ci-cd-diagram.md`, `docs/diagrams/post-pattern-flow.md`, `docs/diagrams/concept.md`
  - Steps:
    - Compare diagram files by filename across all three repos
    - For files present in multiple repos with identical content: copy once; add single-source header note
    - For files present in multiple repos with differing content: diff and take the most recently modified version; add header note noting the source
    - Note: mnemonic-api has `concept.md` instead of the five numbered concept-iteration files; copy mnemonic-api/docs/diagrams/concept.md as docs/diagrams/concept.md; copy the five numbered files from mnemonic or mnemonic-enricher
    - Add header notes to all diagram files
  - Verify: `ls docs/diagrams/01-concept-local-iteration.md docs/diagrams/post-pattern-flow.md docs/diagrams/concept.md`
  - Done: All diagram files exist in docs/diagrams/ with header notes

- [x] **Cycle 10 - Fix internal cross-links**: Update internal cross-links in all consolidated docs to use correct relative paths within mnemonic-docs.
  - Agent: `technical writer`
  - Files: All `.md` files under `docs/`
  - Steps:
    - Search all docs/ markdown files for links referencing source-repo paths (e.g., `../design/`, relative paths that no longer resolve)
    - Rewrite valid internal links to use correct relative paths within mnemonic-docs/docs/
    - Replace links to design docs (which remain in each component repo) with an inline note: `<!-- design docs remain in <repo>/docs/design/ -->`
  - Verify: `grep -rE "\]\(\.\./design/|\]\(\.\./plans/" docs/ --include="*.md" | wc -l | xargs test 0 -eq`
  - Done: No consolidated docs contain relative links pointing to design/ or plans/ directories from source repos

- [x] **Cycle 11 - README and validation**: Create the top-level README.md and verify the full validation checklist passes.
  - Agent: `technical writer`
  - Files: `README.md`
  - Steps:
    - Write README.md at the repo root describing the mnemonic-docs project, its purpose, and the docs/ directory structure
    - Verify all expected files from the Source Mapping table exist at their target paths
    - Verify all reconciled docs contain a blockquote header note with a Source field
    - Verify docs/architecture/dbs/ and docs/architecture/admin/ are populated
    - Verify docs/diagrams/ is populated
  - Verify: `ls README.md docs/concept.md docs/architecture/system/00-architectural-decisions.md docs/architecture/mnemonic/08-mcp-tools.md docs/architecture/dbs/00-overview.md docs/architecture/admin/0001-frontend-stack.md docs/diagrams/post-pattern-flow.md`
  - Done: All listed files exist and README.md exists at repo root

## Risks and Mitigations

- Risk: A reconciled doc has ambiguous content that is hard to classify as system-level or component-specific.
  - Mitigation: Default to system/ and add `<!-- REVIEW: origin ambiguous -->` inline; do not block the cycle.
- Risk: Diagrams in mnemonic-api use a different naming convention (concept.md vs numbered files).
  - Mitigation: Copy both conventions; mnemonic-api's concept.md and mnemonic's numbered files are complementary, not duplicates.
- Risk: Cross-links between architecture docs break after consolidation.
  - Mitigation: Cycle 10 explicitly fixes all internal links; known design-doc links are replaced with inline notes.

## Definition of Done

- `ls README.md docs/concept.md docs/requirements.md docs/requirements-traceability.md` exits 0
- `ls docs/architecture/system/0{0,1,2,3,4,5,6,7}-*.md` exits 0
- `ls docs/architecture/mnemonic/08-mcp-tools.md docs/architecture/dbs/00-overview.md docs/architecture/admin/0001-frontend-stack.md` exits 0
- `grep -rL "Source:" docs/architecture/system/ docs/architecture/mnemonic/ docs/architecture/dbs/ docs/architecture/admin/ --include="*.md" | grep -v README | wc -l | xargs test 0 -eq`
