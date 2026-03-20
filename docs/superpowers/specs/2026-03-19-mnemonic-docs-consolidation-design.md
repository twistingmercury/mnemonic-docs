# Mnemonic Docs Consolidation — Design Spec

**Date:** 2026-03-19
**Status:** Approved

---

## Problem

The mnemonic solution has been split into five repositories. Documentation was copied across repos and has since diverged. There is no single source of truth.

Repositories:
- `mnemonic` — MCP tools
- `mnemonic-api` — RESTful API for pattern management
- `mnemonic-dbs` — database schemas and dev Docker images
- `mnemonic-enricher` — enrichment worker
- `mnemonic-admin` — web UI for pattern management

---

## Goal

Consolidate all canonical reference documentation into `mnemonic-docs`. Other repos reference this project for architectural context; they retain their own implementation-specific design docs.

**Out of scope:** process artifacts (plans, code reviews, `_ideas/`, superpowers plans). These remain in their respective repos.

---

## Target Structure

```
mnemonic-docs/
  README.md
  PURPOSE.md
  docs/
    concept.md                        # what mnemonic is (reconciled)
    requirements.md                   # system requirements (reconciled)
    requirements-traceability.md      # requirements traceability (reconciled)
    architecture/
      system/                         # cross-cutting architecture docs (00–07)
        00-architectural-decisions.md
        01-security-architecture.md
        02-system-architecture.md
        03-communication-patterns.md
        04-data-architecture.md
        05-database-integration-flow.md
        06-deployment-architecture.md
        07-observability-architecture.md
        README.md
      mnemonic/                       # MCP service architecture
        08-mcp-tools.md               # moved here: MCP-specific, not system-wide
      api/                            # api-specific architecture (post-diff)
      enricher/                       # enricher-specific architecture (post-diff)
      dbs/                            # database-specific architecture
        00-overview.md
        02-architectural-decisions.md
        05-deployment-architecture.md
        08-data-architecture.md
        README.md
      admin/                          # admin-specific architecture
        0001-frontend-stack.md        # ADR
    diagrams/                         # concept and iteration diagrams
```

---

## Source Discovery

Before reconciling, enumerate all `.md` files under `docs/` in each source repo and compare lists. Files present in only one repo are single-source. Files present in multiple repos require diffing.

Source repos for shared docs: `mnemonic`, `mnemonic-api`, `mnemonic-enricher`.

Use `mnemonic` as the baseline when a shared doc exists in all three. If a doc exists in only one or two of the three, use the version that exists as the baseline.

---

## Reconciliation Process

For each shared doc (found in two or more of: `mnemonic`, `mnemonic-api`, `mnemonic-enricher`):

1. **Diff** the file across all repos that contain it.
2. **Classify** each difference using these heuristics:
   - A section that references component-internal concepts (e.g., "the MCP tool X", "the enricher uses Y", "the API endpoint Z") → component-specific; extract into `mnemonic/`, `api/`, or `enricher/` subdir as appropriate
   - A section that describes system-wide behavior (data flow, security model, deployment topology) → system-level; merge into `architecture/system/`
   - A section where two repos have conflicting factual content → use the most recently modified version; note the discrepancy in the header note
   - A section that appears only in one repo's version → assess whether it is an addition (include it) or reflects a deletion (if deleted in mnemonic, treat as stale and discard)
3. **Write** the reconciled output to the appropriate location in `mnemonic-docs/docs/`.
4. **Add a header note** (see format below) to each reconciled doc.

---

## Header Note Format

Add this block at the top of every reconciled or copied doc, below the H1 title:

```markdown
> **Source:** Reconciled from `mnemonic/docs/...`, `mnemonic-api/docs/...`, `mnemonic-enricher/docs/...`
> **Component-specific divergences:** See `docs/architecture/api/` and `docs/architecture/enricher/`
> **Last reconciled:** 2026-03-19
```

For single-source docs (copied without reconciliation), use:

```markdown
> **Source:** Copied from `<repo>/docs/...`
> **Last copied:** 2026-03-19
```

---

## Single-Source Docs

Docs that exist in only one repo (mnemonic-dbs architecture, mnemonic-admin ADR) are copied directly to their target location. During copy:
- Update any internal cross-links that reference paths from the source repo to point to the equivalent path in `mnemonic-docs`
- Do not change content or style

---

## Diagram Deduplication

Diagrams folder (`docs/diagrams/`) exists in `mnemonic`, `mnemonic-api`, and `mnemonic-enricher`.

Rules:
- Compare files by filename across repos
- If a diagram file has the same name and identical content across repos, copy once
- If a diagram file has the same name but different content, diff and take the most recently modified version; note the source in the header comment of the `.md` file
- If a diagram file exists in only one repo, copy it directly

---

## Link Fixing

After all docs are written to `mnemonic-docs`, fix internal cross-links within the consolidated docs:
- Identify any links that reference source-repo paths (e.g., `../design/...`, `./architecture/...` with old relative paths)
- Rewrite them to reference the correct relative path within `mnemonic-docs/docs/`
- Links that reference files NOT consolidated into mnemonic-docs (e.g., design docs that remain in each repo) should be replaced with a note: `<!-- design docs remain in <repo>/docs/design/ -->`

---

## Source Mapping

| Source file | Repos | Target |
|---|---|---|
| `docs/mnemonic-concept.md` | mnemonic, mnemonic-api, mnemonic-enricher | `docs/concept.md` |
| `docs/mnemonic-requirements.md` | mnemonic, mnemonic-api, mnemonic-enricher | `docs/requirements.md` |
| `docs/requirements-traceability.md` | mnemonic, mnemonic-enricher | `docs/requirements-traceability.md` |
| `docs/architecture/00–07 + README` | mnemonic, mnemonic-api, mnemonic-enricher | `docs/architecture/system/` (+ component subdirs post-diff) |
| `docs/architecture/08-mcp-tools.md` | mnemonic, mnemonic-api, mnemonic-enricher | `docs/architecture/mnemonic/08-mcp-tools.md` (MCP-specific) |
| `docs/diagrams/` | mnemonic, mnemonic-api, mnemonic-enricher | `docs/diagrams/` |
| `docs/architecture/*` | mnemonic-dbs only | `docs/architecture/dbs/` |
| `docs/adr/0001-frontend-stack.md` | mnemonic-admin only | `docs/architecture/admin/0001-frontend-stack.md` |

---

## What Stays in Each Repo

Each component repo retains its own:
- `design/` docs (implementation-specific, not consolidated)
- `README.md` and `CHANGELOG.md`
- Process artifacts (`plans/`, `_ideas/`, `code-reviews/`, `superpowers/`)

Design docs in each repo should include a note referencing `mnemonic-docs` for architectural context.

---

## Validation Checklist

After the ralph-loop completes, verify:
- [ ] All files listed in the Source Mapping table exist at their target path
- [ ] All reconciled docs have a header note
- [ ] No internal links in `mnemonic-docs/docs/` reference source-repo paths
- [ ] Component-specific content extracted to `mnemonic/`, `api/`, or `enricher/` subdirs has its own header note
- [ ] `mnemonic-dbs` and `mnemonic-admin` docs are present at their target paths
- [ ] `docs/architecture/mnemonic/08-mcp-tools.md` exists
- [ ] Diagrams folder is populated and deduplicated

---

## Conflict Escalation

If the ralph-loop encounters a diff where it cannot determine whether content is system-level or component-specific (no clear heuristic applies), it should:
1. Copy the content to `system/` as the default
2. Add an inline comment: `<!-- REVIEW: origin ambiguous, verify this belongs at system level -->`
3. Continue without blocking

---

## Execution

This consolidation will be executed via a ralph-loop (`docs/plans/PRD.md` + `docs/plans/PROMPT.md`). No edits are made to source repos.
