# Runtime Tools

Status: Research

## Purpose

- This doc is the source of truth for the standard runtime tool surface in v1
- Skarbot should reuse `pi`'s built-in coding substrate where possible and only define a small additional system layer

## Tool Layers

- Skarbot should treat `pi`'s coding-core tools as substrate rather than redefining them
- Skarbot should add a small shared system layer on top of that substrate
- Direct-thread-only tools should stay separate from tools available in both execution profiles
- Task-thread-only tools should stay separate from tools available in both execution profiles

## `pi` Coding Core

- Both execution profiles should use the `pi` coding-core tools as the baseline runtime surface:
  - `read`
  - `write`
  - `edit`
  - `bash`
  - `grep`
  - `find`
  - `ls`
- Skarbot should not wrap or rename these tools in v1

## Shared System Tools

- Memory should not be a dedicated runtime tool in v1
- Each workspace should instead expose a normal `MEMORY.md` file that the agent maintains through the `pi` coding-core tools
- Searchable thread history may be added later as a separate tool, but it should not be folded into memory
- Both execution profiles should include first-class web access tools:
  - `web_search`
  - `web_fetch`
- `web_search` should find candidate URLs and lightweight web results
- `web_fetch` should retrieve one URL and extract readable content for the agent without requiring a full browser tool
- Both execution profiles should include minimal thread-local planning:
  - `plan()`
  - `plan(create, steps)`
  - `plan(update, id, status)`
- `plan(...)` should operate on one active plan per thread
- When all steps are `completed`, the active plan should be considered complete and may be removed from active state
- Both execution profiles should include grouped subagent orchestration:
  - `subagents()`
  - `subagents(spawn, task, label?)`
  - `subagents(kill, name)`
- `subagents(...)` should create generic worker subagents only in v1
- Spawned subagents should use model-chosen cute Malayalee names as their only active handles
- Duplicate active subagent names in the same thread should be rejected
- Both execution profiles should include minimal grouped model control:
  - `model()`
  - `model(list)`
  - `model(set, provider, model)`
- `model()` should return the current thread model plus current thread usage and cost
- `model(list)` should return the configured available models rather than a larger aspirational provider catalog
- `model(set, provider, model)` should switch the current thread model and persist that change in thread session history
- `model(...)` should not change the deployment default model in v1
- Neither execution profile should require a user-facing compaction tool in v1
- Context-window management should be automatic:
  - rolling compaction should happen before the hard context limit
  - overflow recovery should compact and retry automatically once
  - users should not need to think about or trigger compaction during normal use

## Direct-Thread System Tools

- Direct-thread runs should include grouped schedule management:
  - `schedule()`
  - `schedule(set, at)`
  - `schedule(set, cron)`
  - `schedule(edit, id, at)`
  - `schedule(edit, id, cron)`
  - `schedule(delete, id)`
- `schedule()` should list the user's existing scheduled automations and their stable handles
- Schedule handles should be stable human-readable names unique per user
- One-off schedules should create one-off throwaway task threads
- Recurring schedules should bind to one dedicated reusable task thread
- Schedule mutations should require explicit user approval before persistence
- Recurring schedules should default to the user's configured timezone when timezone is not explicitly provided in the request

## Task-Thread System Tools

- All task-thread runs should include:
  - `ask-user(question)`
  - `capabilities(action, kind, name?, path?)`
- The grouped `capabilities` tool should support exactly:
  - `list`
  - `inspect`
  - `get`
  - `promote`
- `capabilities(action = "get")` should seed a task workspace from candidate first, then active
- `capabilities(action = "promote")` should run deterministic validation internally and fail fast back to the task when invalid

## Pending Grouped Tools

- No dedicated grouped `memory(...)` tool is planned for v1
- Searchable thread history remains a possible future tool, but it is intentionally separate from memory
