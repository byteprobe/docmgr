---
Title: Diary
Ticket: 008-GLAZED-FACADE-MIGRATION
Status: active
Topics:
    - docmgr
    - glaze
    - cli
    - tooling
    - testing
DocType: reference
Intent: long-term
Owners: []
RelatedFiles:
    - Path: ../../../../../../../glazed/pkg/doc/tutorials/migrating-to-facade-packages.md
      Note: Reference used to derive the task breakdown
    - Path: cmd/docmgr/cmds/common/common.go
      Note: First shared migration target
    - Path: pkg/commands/add.go
      Note: Baseline compile failure originates here
ExternalSources: []
Summary: ""
LastUpdated: 2026-03-24T21:55:59.020132324-04:00
WhatFor: ""
WhenToUse: ""
---


# Diary

## Goal

Track the implementation work required to make `docmgr` compile and run against the new Glazed facade packages, including the exact failures encountered, the migration decisions taken, and the commands used to validate each step.

## Step 1: Create ticket workspace and capture the migration plan

The first step was to turn the user request into a ticketed workstream inside `docmgr` itself, then record the failure mode before touching application code. The current branch already contains the new Glazed dependency, so the first hard signal was the compiler: package loading fails immediately because `pkg/cmds/layers` and `pkg/cmds/parameters` no longer exist.

That made the migration guide the primary design input. I created a new ticket workspace, seeded tasks that mirror the real dependency order in the codebase, and wrote an implementation plan that treats shared Cobra wiring as the first code slice before the broader `pkg/commands` migration.

### Prompt Context

**User prompt (verbatim):** "1. Create a new ticket to make docmgr work with new glazed version glazed/pkg/doc/tutorials/migrating-to-facade-packages.md .

Create an implementation plan and then add tasks and then work on them one by one, comitting at appropriate intervals, keeping a diary"

**Assistant interpretation:** Create a docmgr ticket for the Glazed facade migration, document the plan and tasks in the ticket workspace, then execute the migration in reviewed, incremental commits while maintaining a diary.

**Inferred user intent:** Use docmgr’s own workflow to manage a real dependency-upgrade task end to end and leave behind a reviewable record of both the code changes and the process used to make them.

**Commit (code):** N/A

### What I did
- Read the Glazed migration guide at `glazed/pkg/doc/tutorials/migrating-to-facade-packages.md`.
- Ran `go test ./...` in `/home/manuel/workspaces/2026-03-24/fix-docmgr-doctor/docmgr` to capture the current failure mode.
- Created ticket `008-GLAZED-FACADE-MIGRATION`.
- Added a design doc for the implementation plan and a reference doc for this diary.
- Seeded the initial task list with migration slices derived from the failing compile surface.

### Why
- The compile errors show broad API removal, not a one-file regression.
- Capturing the plan before editing code makes the subsequent commits easier to review against the intended migration order.

### What worked
- The installed `docmgr` CLI was able to operate on the repo’s existing `ttmp` workspace.
- `go test ./...` failed fast enough to confirm the migration guide is directly relevant to this repo state.
- The new ticket workspace was created cleanly under the current date hierarchy.

### What didn't work
- `go test ./...` does not currently reach actual test execution. The initial failure is package loading:

```text
pkg/commands/add.go:17:2: no required module provides package github.com/go-go-golems/glazed/pkg/cmds/layers
pkg/commands/add.go:18:2: no required module provides package github.com/go-go-golems/glazed/pkg/cmds/parameters
```

### What I learned
- The branch is already pointed at `github.com/go-go-golems/glazed v0.7.3`, so this is a pure migration task rather than a dependency-bump task.
- `cmd/docmgr/cmds/common/common.go` is a critical leverage point because it configures Glazed defaults for many commands.

### What was tricky to build
- The worktree root is not itself a git repository; `docmgr` is the nested repo. That matters for every validation and commit command because they need to run with `/home/manuel/workspaces/2026-03-24/fix-docmgr-doctor/docmgr` as the git root.
- The ticket tooling uses the repo-local docmgr workspace rooted at `docmgr/ttmp`, while the config file is at `/home/manuel/workspaces/2026-03-24/fix-docmgr-doctor/.ttmp.yaml`. That split is fine, but it needs to be kept straight when reading tool output and relating files.

### What warrants a second pair of eyes
- The exact Glazed command runtime interface expected by v0.7.3 may force more than a mechanical import rename.
- Struct-tag compatibility needs confirmation once the code compiles far enough to exercise decoding.

### What should be done in the future
- After the code migration is stable, sweep contributor docs under `pkg/doc` and `CONTRIBUTING.md` for stale `layers`/`parameters` examples.

### Code review instructions
- Start with `glazed/pkg/doc/tutorials/migrating-to-facade-packages.md`, then compare it to `cmd/docmgr/cmds/common/common.go` and `pkg/commands/add.go`.
- Validate the baseline failure with `go test ./...` from `/home/manuel/workspaces/2026-03-24/fix-docmgr-doctor/docmgr`.

### Technical details
- Ticket ID: `008-GLAZED-FACADE-MIGRATION`
- Ticket path: `ttmp/2026/03/24/008-GLAZED-FACADE-MIGRATION--migrate-docmgr-to-glazed-facade-packages`
- Baseline compile failure surfaces first in `pkg/commands/add.go`, but the import inventory shows the legacy packages are referenced throughout `pkg/commands`, `cmd/docmgr/cmds/common`, and `scenariolog`.
