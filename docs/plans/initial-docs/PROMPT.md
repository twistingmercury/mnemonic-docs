# Ralph Loop Prompt for mnemonic-docs Initial Consolidation

You are executing one Ralph loop cycle for the `mnemonic-docs` repository.
Follow the repository-specific rules below.

## Objective

Complete exactly one unchecked cycle from the active PRD, verify it, update the
PRD state, append a progress entry, and stop.

## Inputs

You will be given:

- the active PRD
- the current progress log, if one exists
- the repository working tree

The active PRD for this project is expected to be:

- `docs/plan/initial-docs/PRD.md`

The primary supporting documents are:

- `docs/superpowers/specs/2026-03-19-mnemonic-docs-consolidation-design.md`

The canonical progress log path for this repository is:

- `docs/plan/initial-docs/progress.txt`

If `docs/plan/initial-docs/progress.txt` does not exist, create it when completing
the first cycle.

## Non-Negotiable Rules

1. Execute exactly one PRD cycle.
2. Work only on the first unchecked `- [ ]` cycle in the PRD.
3. Do not skip ahead.
4. Do not combine multiple cycles into one run.
5. Search the repository before editing. Do not assume code or files are
   missing.
6. Respect the cycle's `Agent`, `Files`, `Steps`, and `Verify` fields.
7. Keep changes scoped to the selected cycle.
8. Run verification before marking the cycle complete.
9. Update the PRD and progress log only after the cycle passes verification.
10. Stop after finishing that one cycle.

## Repo-Specific Rules

- This is a documentation-only repository. There is no build step and no test suite.
- All output files are Markdown (`.md`).
- Source files live in sibling repos at `$HOME/dev/mnemonic/`, `$HOME/dev/mnemonic-api/`, `$HOME/dev/mnemonic-enricher/`, `$HOME/dev/mnemonic-dbs/`, and `$HOME/dev/mnemonic-admin/`. These repos are read-only — do not modify them.
- Every output doc must include a blockquote header note immediately below the H1 title. See the design spec for the exact format.
- Reconciliation heuristics (from the design spec):
  - Section references a specific component → extract to that component's subdir
  - Section describes system-wide behavior → write to `docs/architecture/system/`
  - Conflicting factual content → use the most recently modified version; note discrepancy in header
  - Classification is ambiguous → default to `system/` and add `<!-- REVIEW: origin ambiguous -->` inline
- Commits must be signed: use `git commit -S`.
- Do not skip hooks or signing (`--no-verify`, `--no-gpg-sign`).

## Ralph Loop Procedure

### Step 1: Read the PRD and select the cycle

- Open `docs/plan/initial-docs/PRD.md`.
- Find the first unchecked `- [ ]` cycle under `## Implementation Plan`.
- Extract:
  - cycle title
  - cycle description
  - `Agent`
  - `Files`
  - `Steps`
  - `Verify`

If no unchecked cycle exists, stop and report that the PRD is complete.

### Step 2: Read supporting context

- Read the design spec at `docs/superpowers/specs/2026-03-19-mnemonic-docs-consolidation-design.md`.
- Read the progress log if it exists.
- Read the source files from the sibling repos relevant to the selected cycle.

### Step 3: Plan narrowly

- Form a minimal plan that completes only the selected cycle.
- Do not plan future cycles.
- Do not expand scope beyond the listed files and directly necessary support files.

### Step 4: Delegate or execute

- If your runtime supports subagents, delegate to the cycle's named `Agent`.
- If not, execute the cycle directly while still honoring the assigned role.
- Keep the implementation bounded to the selected cycle.

### Step 5: Verify

Run the cycle's `Verify` command exactly as written unless it is impossible in
the current environment.

Baseline checks for this project:

1. Every file listed in the cycle's `Files` field exists at its target path — `ls <file>` exits 0
2. Every output file contains a blockquote header note with a `Source:` field — `grep -l "Source:" <files>`
3. No output file contains relative links to `../design/` or `../plans/` from source repos

If any check fails:

- fix the problem if it is within the cycle scope
- rerun verification
- do not mark the cycle complete until all checks pass

### Step 6: Commit and tag the changes

After verification passes, stage and commit all files produced or modified by
the cycle:

- stage only the files listed in the cycle's `Files` field and any directly necessary support files
- use a concise commit message that names the cycle number and title
- sign the commit: `git commit -S`
- do not skip hooks or sign flags
- if the commit fails, fix the issue and recommit before proceeding

After the commit succeeds, create an annotated tag for the cycle:

- tag name: `cycle-<N>` (e.g., `cycle-1`, `cycle-2`)
- tag message: the cycle title (e.g., `Cycle 1 - Top-level system docs`)
- sign the tag: `git tag -s cycle-<N> -m "<cycle title>"`
- if tagging fails, fix the issue and retag before proceeding

### Step 7: Update project records

After the commit succeeds:

- change the selected PRD cycle from `- [ ]` to `- [x]`
- append a concise entry to `docs/plan/initial-docs/progress.txt`
- stage and commit the updated PRD and progress log as a follow-up commit (also signed)

Each progress entry should include:

- cycle number and title
- date
- summary of work completed
- verification performed
- important follow-up notes, if any

### Step 8: Report and stop

At the end of the loop:

- report what changed
- report what verification passed
- report the next unchecked cycle
- stop

Do not continue into the next cycle.

## Failure Modes to Avoid

- completing more than one cycle in one run
- editing files in source repos (mnemonic, mnemonic-api, mnemonic-enricher, mnemonic-dbs, mnemonic-admin)
- skipping repository search and duplicating existing content
- marking a cycle complete before verification passes
- omitting the blockquote header note from an output file
- using unsigned commits

## Output Contract

Your final response for a completed loop should contain:

- the completed cycle number and title
- the files changed
- the verification that passed
- the next unchecked cycle

If you could not complete the cycle, state exactly why and do not mark it done.
