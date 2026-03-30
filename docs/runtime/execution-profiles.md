# Runtime Execution Profiles

Status: Research

## Purpose

- This doc is the source of truth for how Skarbot composes an agent turn in v1
- The standard runtime tool surface is defined centrally in `docs/runtime/tools.md`
- Skarbot should define exactly two execution profiles:
  - `direct-thread`
  - `task-thread`
- Scheduled runs, capability review, and similar cases should not create new profile types in v1

## Shared Runtime Contract

- Both profiles should run on the same `pi` session substrate
- Both profiles should use the shared standard tool surface defined in `docs/runtime/tools.md`
- Both profiles should resolve system capabilities from:
  - `~/skarbot/core/capabilities/skills/`
  - `~/skarbot/core/capabilities/tools/`
- Both profiles should resolve the owner's active capabilities from:
  - `~/skarbot/state/capabilities/users/<user-id>/skills/`
  - `~/skarbot/state/capabilities/users/<user-id>/tools/`
- Both profiles should re-resolve active capabilities at the start of each run or turn
- Both profiles should use the same per-thread model rule:
  - new threads start from the deployment default model
  - the current model is remembered per thread through session history
- Both profiles should use the same session-file format and `pi` message model
- Both profiles should treat the thread workspace as `SessionHeader.cwd`
- Both profiles should keep durable workspace notes in a workspace-local `MEMORY.md` file
- Both profiles should use automatic rolling compaction so the user does not need to manage context windows manually
- Automatic compaction should keep full append-only thread history on disk and only change the model-facing context for future turns
- Automatic compaction should use a rolling summary plus recent-tail model:
  - the latest compaction summary stays in thread history
  - the next compaction summarizes that previous summary plus older messages since the last compaction point
  - a recent post-compaction tail should remain verbatim rather than being immediately re-summarized
- Compaction inputs should use truncated serialized tool-call and tool-result content so large outputs do not dominate summarization
- Automatic compaction should trigger proactively near the context limit and should also support one overflow-recovery compact-and-retry path
- Compaction boundaries should surface as thread-history events and in the web UI thread viewer only

## Direct-Thread Profile

- The `direct-thread` profile is used for a user's long-lived main conversation
- It should use:
  - thread file: `~/skarbot/state/threads/<user-id>.jsonl`
  - sidecar: `~/skarbot/state/threads/<user-id>.meta.json`
  - attachments: `~/skarbot/state/threads/<user-id>.attachments/`
  - workspace: `~/skarbot/workspaces/users/<user-id>/`
- The direct-thread workspace should keep its durable notes in `~/skarbot/workspaces/users/<user-id>/MEMORY.md`
- It should load only system capabilities plus the owner's active capabilities
- It should not load candidate capabilities
- Reply routing should follow the direct-thread sidecar state
- `plan(...)` should operate on the direct thread's own plan state
- `subagents(...)` should track spawned subagents for the direct thread only
- `schedule(...)` should be managed from the direct thread as a user-owned automation surface

## Task-Thread Profile

- The `task-thread` profile is used for every scoped task
- It should use:
  - thread file: `~/skarbot/state/threads/task-<task-slug>.jsonl`
  - attachments: `~/skarbot/state/threads/task-<task-slug>.attachments/`
  - workspace: `~/skarbot/workspaces/tasks/<task-slug>/`
- The task-thread workspace should keep its durable notes in `~/skarbot/workspaces/tasks/<task-slug>/MEMORY.md`
- It should load system capabilities plus the owner's active capabilities
- Task-thread runs should not need to execute in the owner's direct-thread workspace to access the owner's active capabilities
- All task-thread runs should expose the task-thread system tool surface defined in `docs/runtime/tools.md`
- A capability-building task thread may temporarily load its own candidate capability as a preview overlay
- That candidate preview should stay limited to the linked task thread and should not leak into the owner's direct thread or unrelated task threads
- `subagents(spawn, ...)` should create one generic worker subagent type in v1
- Spawned subagents should use model-chosen cute Malayalee names as their only active handles
- Duplicate active subagent names in the same thread should be rejected
- Recurring scheduled runs should reuse their bound task thread, while one-off schedules should use throwaway task threads

## What Is Not A Separate Profile

- A scheduled task run is still a `task-thread` run
- A capability-building or capability-review task run is still a `task-thread` run
- A run resumed from a different surface is still the same underlying thread profile
- These are task or thread state differences, not separate profile types:
  - trigger source
  - notification or delivery target
  - current model
  - candidate capability preview
  - task lifecycle state
