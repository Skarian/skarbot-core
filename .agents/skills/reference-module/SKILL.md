---
name: reference-module
description: Use to research git repos when user requests.
---

# Reference Module

Trigger: use when the user asks to research another git repository.

## Workflow

1. Choose target path:
   - default to `references/<repo-name>`
   - only exception is a user-provided path when the skill is called
2. Collect required inputs:
   - repository URL (`https://...` or `git@...`)
   - optional branch/tag/commit override
3. Confirm behavior for this run:
   - default is pinning to the latest available commit at `HEAD` (or branch `HEAD` when `-b <branch>` is used) at add time
   - explicitly confirm this default with the user every time before running commands
4. Add the submodule from this repo root:

    git submodule add <repo-url> <target-path>

   If a branch is specified:

    git submodule add -b <branch> <repo-url> <target-path>

5. Initialize recursively when needed:

    git submodule update --init --recursive <target-path>

6. If a specific commit or tag is requested, check it out inside the submodule and stage the submodule pointer update.
7. Treat submodule contents as read-only for research unless the user explicitly asks for edits.
8. Report the outcome:
   - submodule path
   - source URL
   - checked-out commit SHA
