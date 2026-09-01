---
description: >
  Writes a pull-request description from the staged git diff, using the team template.
  Use when the user asks for a PR description, PR summary, PR body, or says "write the PR".
argument-hint: "[ticket-id]"
disable-model-invocation: false
allowed-tools: Bash(git diff:*) Bash(git log:*) Read
---

## Staged changes

!`git diff --staged --stat`

## Full staged diff

!`git diff --staged`

## Recent commits on this branch

!`git log --oneline origin/main..HEAD`

## Instructions

Produce a PR description by filling in [template.md](template.md):

1. **Summary** — one or two sentences on what changes and why.
2. **Changes** — bullet list grouped by area (API, domain, tests, config…). Describe
   behaviour, not file names.
3. **Testing** — how this was verified (commands run, cases covered).
4. **Risk** — call out anything reviewers must scrutinise: schema/migration changes,
   config or env changes, changes to `src/domain/billing/`, backwards-incompatible API
   changes. Write "None" only if you are sure.
5. If a ticket id was passed as `$1`, add a "Closes $1" line at the top.

Output only the finished description in markdown. Do not include the diff.
