# Product Requirements Document: Stale Documentation Cleanup

*Gralph processes cycles in this document from top to bottom. Checklist markers are significant: `- [ ]` (open), `- [x]` (complete), `- [~]` (abandoned). Each cycle must be small, independently verifiable, and assigned to exactly one agent.*

## Objective

Fix all stale documentation across the Mnemonic multi-repo workspace by resolving 13 GitHub issues. Each cycle picks up one ticket, reads its description for full context and instructions, makes the fix, commits on a dedicated branch, creates a PR, and comments on the issue.

## Problem Statement

A comprehensive audit found 13 documentation issues across 7 repositories. Docs reference old embedding models, incorrect defaults, stale architecture, removed features, and outdated schema diagrams. Each issue has been filed with detailed instructions, affected files, correct values, and verification steps. The work is mechanical — read the ticket, follow the instructions, verify the fix.

## Success Criteria

- All 13 GitHub issues have a linked PR with the fix
- All 13 issues have a comment describing the fix and linking to the PR
- Each PR's changes match what the ticket requested

## Scope

### In scope

- Fixing documentation files as described in each ticket
- One source code fix (`input.go` jsonschema tag in mnemonic-mcp#73)
- Creating branches, commits, PRs, and issue comments

### Out of scope

- Changing application source code (except mnemonic-mcp#73 jsonschema tag)
- Updating user guides in `mnemonic-docs/docs/guides/`
- Closing issues (leave open for human review)

## Constraints and Decisions

- **Workspace root:** `/home/jeremy/dev/mnemonic/`
- **Repos are side-by-side:** each cycle's ticket specifies which repo to work in
- **Branch naming:** `fix/<kebab-case-derived-from-ticket-title>`
- **PRs target `develop`** (trunk branch in all repos)
- **GitHub CLI (`gh`)** is available and authenticated
- **The ticket description contains everything needed** — affected files, what to change, source of truth, and verification steps

## Implementation Plan

- [x] **Cycle 1**: `gh issue view 1 --repo twistingmercury/mnemonic-docs`
  - Agent: `technical writer`
  - Repo: `mnemonic-docs`
  - Verify: see ticket description
  - Done: PR created, issue commented with fix description and PR link

- [x] **Cycle 2**: `gh issue view 2 --repo twistingmercury/mnemonic-docs`
  - Agent: `technical writer`
  - Repo: `mnemonic-docs`
  - Verify: see ticket description
  - Done: PR created, issue commented with fix description and PR link

- [x] **Cycle 3**: `gh issue view 3 --repo twistingmercury/mnemonic-docs`
  - Agent: `technical writer`
  - Repo: `mnemonic-docs`
  - Verify: see ticket description
  - Done: PR created, issue commented with fix description and PR link

- [x] **Cycle 4**: `gh issue view 72 --repo twistingmercury/mnemonic-mcp`
  - Agent: `technical writer`
  - Repo: `mnemonic-mcp`
  - Verify: see ticket description
  - Done: PR created, issue commented with fix description and PR link

- [x] **Cycle 5**: `gh issue view 73 --repo twistingmercury/mnemonic-mcp`
  - Agent: `technical writer`
  - Repo: `mnemonic-mcp`
  - Verify: see ticket description
  - Done: PR created, issue commented with fix description and PR link

- [x] **Cycle 6**: `gh issue view 74 --repo twistingmercury/mnemonic-mcp`
  - Agent: `technical writer`
  - Repo: `mnemonic-mcp`
  - Verify: see ticket description
  - Done: PR created, issue commented with fix description and PR link

- [x] **Cycle 7**: `gh issue view 3 --repo twistingmercury/mnemonic-api`
  - Agent: `technical writer`
  - Repo: `mnemonic-api`
  - Verify: see ticket description
  - Done: PR created, issue commented with fix description and PR link

- [x] **Cycle 8**: `gh issue view 4 --repo twistingmercury/mnemonic-api`
  - Agent: `technical writer`
  - Repo: `mnemonic-api`
  - Verify: see ticket description
  - Done: PR created, issue commented with fix description and PR link

- [ ] **Cycle 9**: `gh issue view 3 --repo twistingmercury/mnemonic-enricher`
  - Agent: `technical writer`
  - Repo: `mnemonic-enricher`
  - Verify: see ticket description
  - Done: PR created, issue commented with fix description and PR link

- [ ] **Cycle 10**: `gh issue view 4 --repo twistingmercury/mnemonic-enricher`
  - Agent: `technical writer`
  - Repo: `mnemonic-enricher`
  - Verify: see ticket description
  - Done: PR created, issue commented with fix description and PR link

- [ ] **Cycle 11**: `gh issue view 1 --repo twistingmercury/mnemonic-dbs`
  - Agent: `technical writer`
  - Repo: `mnemonic-dbs`
  - Verify: see ticket description
  - Done: PR created, issue commented with fix description and PR link

- [ ] **Cycle 12**: `gh issue view 2 --repo twistingmercury/mnemonic-patterns`
  - Agent: `technical writer`
  - Repo: `mnemonic-patterns`
  - Verify: see ticket description
  - Done: PR created, issue commented with fix description and PR link

- [ ] **Cycle 13**: `gh issue view 2 --repo twistingmercury/mnemonic-admin`
  - Agent: `technical writer`
  - Repo: `mnemonic-admin`
  - Verify: see ticket description
  - Done: PR created, issue commented with fix description and PR link

## Risks and Mitigations

- Risk: Ticket description is ambiguous or incomplete for a particular fix.
  - Mitigation: Each ticket was reviewed to contain affected files, line numbers, incorrect vs correct values, source of truth references, and verification steps. If something is unclear, check the source of truth file referenced in the ticket.

- Risk: Branch conflicts from prior attempts.
  - Mitigation: Check if branch exists before creating. If it does, delete and recreate from develop.

## Definition of Done

- All 13 cycles are marked `- [x]`
- Each cycle has a PR linked in its GitHub issue comment
- Issues are left open for human review
