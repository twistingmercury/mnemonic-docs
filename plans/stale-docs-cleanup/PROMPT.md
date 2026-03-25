# Ralph Loop Prompt for Stale Documentation Cleanup

You are executing one Ralph loop cycle across the Mnemonic multi-repo workspace.

## Objective

Complete exactly one unchecked cycle from the active PRD, verify it, update the
PRD state, append a progress entry, and stop.

## Inputs

The active PRD for this project is:

- `/home/jeremy/dev/mnemonic/mnemonic-docs/plans/stale-docs-cleanup/PRD.md`

The canonical progress log path is:

- `/home/jeremy/dev/mnemonic/mnemonic-docs/plans/stale-docs-cleanup/progress.txt`

If the progress log does not exist, create it when completing the first cycle.

The workspace root is `/home/jeremy/dev/mnemonic/`. All repos are cloned
side-by-side under this directory.

## Non-Negotiable Rules

1. Execute exactly one PRD cycle.
2. Work only on the first unchecked `- [ ]` cycle in the PRD.
3. Do not skip ahead.
4. Do not combine multiple cycles into one run.
5. Search the repository before editing. Do not assume code or files are
   missing.
6. Respect the cycle's `Agent`, `Repo`, and `Verify` fields.
7. Keep changes scoped to the selected cycle.
8. Run verification before marking the cycle complete.
9. Update the PRD and progress log only after the cycle passes verification.
10. Stop after finishing that one cycle.

## Repo-Specific Build and Test Rules

This is a documentation-only project. No application builds or tests are needed.

- **Each repo is an independent Git repository** with its own branches and remotes.
- **Branch from develop:** Before creating a branch, ensure you are on `develop` and
  it is up to date: `git checkout develop && git pull`.
- **Branch naming:** `fix/<kebab-case-derived-from-ticket-title>`
- **Commit message format:** `docs: <short description>` — keep it under 72 chars.
- **Do not close issues.** Leave them open for human review.
- **Markdown edits:** Use the Edit tool for surgical changes. Do not rewrite
  entire files.

## Ralph Loop Procedure

### Step 1: Read the PRD and select the cycle

- Open the PRD.
- Find the first unchecked `- [ ]` cycle under `## Implementation Plan`.
- Extract the `gh issue view` command from the cycle.

If no unchecked cycle exists, stop and report that the PRD is complete.

### Step 2: Read the ticket

- Run the `gh issue view` command from the cycle to read the full ticket.
- The ticket description contains everything you need:
  - what is wrong
  - which files to change
  - what the correct values are
  - source of truth references
  - verification steps
- Read this carefully. The ticket is your spec.

### Step 3: Read the affected files

- cd into the target repo: `/home/jeremy/dev/mnemonic/<Repo>`
- Read each file mentioned in the ticket before editing.
- Search for the stale strings the ticket describes to understand full scope.

### Step 4: Branch and fix

- Ensure you are on `develop` and up to date: `git checkout develop && git pull`
- Create the branch: `git checkout -b fix/<kebab-case-from-ticket-title>`
- Follow the ticket's instructions to make the fixes.
- Use the Edit tool for targeted changes. Read before editing.

### Step 5: Verify

- Run the verification described in the ticket.
- Most tickets include a grep command that should return zero matches for
  stale strings.
- If stale strings remain, fix them and rerun verification.
- Do not proceed until verification passes.

### Step 6: Commit, push, create PR, comment on issue

After verification passes:

- Stage the changed files.
- Commit: `docs: <short description>`
- Push: `git push -u origin <branch-name>`
- Create a PR with `gh pr create`:
  - Title matches the commit message
  - Body summarizes the changes and references the issue number
- Comment on the issue with `gh issue comment`:
  - Describe what was fixed
  - Include the PR URL

### Step 7: Update project records

After the PR is created:

- cd back to `/home/jeremy/dev/mnemonic/mnemonic-docs`
- Change the completed cycle from `- [ ]` to `- [x]` in the PRD
- Append an entry to `progress.txt` with:
  - cycle number
  - date
  - repo and issue
  - summary of fix
  - PR URL
- Stage and commit the PRD and progress log update in mnemonic-docs

### Step 8: Report and stop

Report:

- the completed cycle number
- the repo and issue
- the files changed
- the PR URL
- the next unchecked cycle

Then stop. Do not continue into the next cycle.

## Failure Modes to Avoid

- completing more than one cycle in one run
- editing files not mentioned in the ticket
- marking a cycle complete before verification passes
- forgetting to cd into the correct repo before branching
- creating a branch from a stale develop (always pull first)
- closing GitHub issues (leave open for human review)
- rewriting entire files instead of making targeted edits
- ignoring the ticket description and inventing your own fix

## Output Contract

Your final response for a completed loop should contain:

- the completed cycle number
- the repo and issue
- the files changed
- the verification that passed
- the PR URL
- the next unchecked cycle

If you could not complete the cycle, state exactly why and do not mark it done.
