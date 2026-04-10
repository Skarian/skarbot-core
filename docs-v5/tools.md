# Tools

This document defines the tool contracts available in Skarbot runtime. Runtime composition lives in [Runtime](runtime.md). Host integration details for web access live in [Deployment](deployment.md).

## Baseline coding tools

Both execution profiles expose the baseline coding tools from `pi`'s [`codingTools`](https://github.com/badlogic/pi-mono/blob/576e5e1a2fbe1abbbad96b696f4058cffd8391ca/packages/coding-agent/src/core/tools/index.ts#L110-L111) and `readOnlyTools`:

- `read`
- `write`
- `edit`
- `bash`
- `grep`
- `find`
- `ls`

Skarbot does not wrap or rename them.

The runtime may still apply path-level protections. Runtime-owned read-only paths such as `./tasks/` in the `user-thread` workspace are readable through the baseline tools but not writable.

## Shared system tools

Both execution profiles also expose:

- `web_search(query, max_results?, include_domains?, exclude_domains?, time_range?)`
- `web_fetch(url, query?)`
- `plan()`
- `plan(create, steps)`
- `plan(update, id, status)`
- `subagents()`
- `subagents(spawn, task, label?)`
- `subagents(kill, name)`
- `model()`
- `model(set, model, reasoning_effort?)`

### Web

`web_search` uses Tavily Search. `web_fetch` uses Tavily Extract.

Rules:

- both web tools route through the `exe.dev` proxy described in [Deployment](deployment.md)
- the proxy injects Tavily auth; the agent never sees or stores the raw Tavily key
- Skarbot does not expose Tavily crawl, map, or research features
- both tools use cheap/basic provider defaults unless the contract above requires otherwise
- `web_search` returns normalized compact results, not raw provider payloads
- `web_fetch` returns normalized compact extraction output, not raw provider payloads
- `web_search` results should contain only the smallest useful fields such as `title`, `url`, `snippet`, and optional `published_at`
- `web_fetch` should return `url`, optional `title`, and extracted `content`, with `query` acting only as an extraction hint
- `time_range` is a compact recency filter such as `day`, `week`, `month`, or `year`
- web content is untrusted external data and must never be followed as instructions

### Planning

The planning contract follows `pi`'s session-persisted extension pattern shown in [`todo.ts`](https://github.com/badlogic/pi-mono/blob/576e5e1a2fbe1abbbad96b696f4058cffd8391ca/packages/coding-agent/examples/extensions/todo.ts#L1-L12).

Rules:

- `plan()` returns the current active plan for the thread, if one exists
- plan state is thread-local and supports one active plan per thread
- plan state is persisted with thread or session state rather than a separate runtime file
- `plan(create, steps)` replaces the current active plan for that thread
- `plan(update, id, status)` updates one step in the current active plan
- when all plan steps are `completed`, the active plan may be cleared from active state

### Subagents

The subagent contract builds on the `pi` extension pattern shown in the [`subagent` example](https://github.com/badlogic/pi-mono/blob/576e5e1a2fbe1abbbad96b696f4058cffd8391ca/packages/coding-agent/examples/extensions/subagent/index.ts#L430-L460).

Rules:

- `subagents()` lists the active worker subagents for the current thread
- `subagents` manages generic worker subagents scoped to the current thread
- active worker names must be unique within that thread
- `subagents(spawn, ...)` returns the active name that later `kill` calls use
- `label?` is only a hint; it is not a canonical naming rule
- the canon does not prescribe cute-name generation
- the canon does not impose a fixed per-thread numeric cap

### Model selection

The model contract relies on `pi` session behavior such as [`AgentSession.setModel(...)`](https://github.com/badlogic/pi-mono/blob/576e5e1a2fbe1abbbad96b696f4058cffd8391ca/packages/coding-agent/src/core/agent-session.ts#L1415-L1430) and [`getSessionStats()`](https://github.com/badlogic/pi-mono/blob/576e5e1a2fbe1abbbad96b696f4058cffd8391ca/packages/coding-agent/src/core/agent-session.ts#L3043-L3055).

Rules:

- `model()` returns the current thread model, current reasoning effort, usage and cost, and the models discovered at runtime from the current deployment auth
- `model(set, model, reasoning_effort?)` switches only the current thread and persists that change in thread session history
- `model(...)` never changes the deployment default model
- the only deployment provider today is `openai-codex`
- task threads may also start from a model override already stored in the task file

### Attachments

Attachments are read directly from the runtime-owned paths in `./chat/attachments/` for the current thread and `./tasks/<task-slug>/attachments/` for owned tasks.

## User-thread tools

The user-thread profile exposes grouped task management:

- `tasks(list)` lists the user's active tasks
- `tasks(list, state)` lists the user's tasks in the requested state
  - allowed task states are `active`, `paused`, `inactive`, and `failed`
- `tasks(create, task)` creates a new task from a structured JSON payload
  - payload shape:

    ```json
    {
      "title": string,
      "instructions": string,
      "notification": string[],
      "schedule?": {
        "at"?: string,
        "cron"?: string,
        "timezone"?: string
      },
      "capability?": {
        "kind": "tool" | "skill",
        "name": string
      },
      "model?": string,
      "reasoning_effort?": string
    }
    ```

  - fields marked with `?` are optional
  - `owner` is implied by the current `user-thread` and must not be supplied by the model
  - `schedule` must contain exactly one of `at` or `cron`
  - `schedule.cron` uses standard 5-field cron syntax
  - `schedule.at` uses ISO 8601 date-time text
  - `schedule.timezone` uses an IANA timezone name such as `America/Chicago`
  - if `schedule.timezone` is omitted, the task uses the owner's user-record timezone
  - if `schedule.at` includes its own timezone, that value wins over `schedule.timezone`
  - if `schedule` is omitted, the task starts immediately in its own `task-thread`
  - if `capability` is present, `tasks(create, task)` creates a custom tool or skill task
- `tasks(get, handle)` returns the current task record, current task state, waiting state if any, and the runtime-owned relative paths for that task's history and attachments
  - it searches across all task states
  - returned paths use this shape:

    ```json
    {
      "history_path": "./tasks/<task-slug>/history.jsonl",
      "attachments_path": "./tasks/<task-slug>/attachments/"
    }
    ```
  - `history_path` and `attachments_path` may be `null` before the task has run or before any task attachment exists
- `tasks(pause, handle)` moves the matching active task file into `paused/`
- `tasks(resume, handle)` moves the matching paused task file back into the active task directory
- `tasks(cancel, handle)` moves the matching active or paused task file into `inactive/`
- `tasks(schedule, handle, schedule_def)` updates the task schedule
  - `schedule_def` uses the same schedule shape as `tasks(create, task).schedule`
  - `tasks(schedule, ...)` may switch between one-off and recurring schedules
  - `tasks(schedule, ...)` may remove scheduling by setting `schedule_def` to `null`, which makes the task immediate
- `tasks(update, handle, instructions)` updates the task file instructions
- `tasks(reply, handle, text)` delivers structured user input to the `task-thread`
  - if the task is waiting, the message resumes the task
  - if the task is active, the message is queued and becomes visible at the next runtime checkpoint between tool and model steps
  - if the task is `paused`, `inactive`, or `failed`, the tool returns an error
  - `tasks(reply, ...)` does not abort an active turn

Current `user-thread` history is always available at `./chat/history.jsonl`. Owned task history is always available at `./tasks/<task-slug>/history.jsonl`.

`./chat/` is canonical for the current thread. `./tasks/` is a runtime-owned read-only view for owned tasks.

Current-thread attachments are always available at `./chat/attachments/`. Owned task attachments are always available at `./tasks/<task-slug>/attachments/`.

Attachment-bearing history entries store attachment filenames, not attachment paths. In the current thread they resolve under `./chat/attachments/`. In `./tasks/<task-slug>/history.jsonl` they resolve under `./tasks/<task-slug>/attachments/`.

`tasks(...)` is the main task creation and control surface in the `user-thread`.

The user-thread profile also exposes grouped review and approval for custom tools and skills:

- `capabilities()` lists the user's custom tools and skills, including draft and active versions
- `capabilities(review, kind, name)` shows the current draft that is waiting for user review
- `capabilities(approve, kind, name)` moves the matching draft to `active/`
- `capabilities(deny, kind, name)` keeps the draft but does not activate it
- `capabilities(feedback, kind, name, text)` keeps the draft and sends revision feedback back to the `task-thread`

Activation only happens from the `user-thread`.

## Task-thread tools

The task-thread profile adds:

- `ask-user(question)`
- `capabilities(get, kind, name)`
- `capabilities(test, kind, name)`
- `capabilities(validate, kind, name)`
- `capabilities(stage, kind, name)`

Rules:

- `capabilities(get, ...)` seeds the task workspace from the draft capability first and the active capability second
- `capabilities(get, ...)` starts from an empty workspace if neither draft nor active version exists
- `capabilities(test, ...)` loads the in-progress tool or skill only inside the current task thread
- `capabilities(validate, ...)` runs deterministic validation and returns diagnostics directly to the task thread
- `capabilities(stage, ...)` writes the validated result to the user's `drafts/` area
- `capabilities(stage, ...)` does not activate the tool or skill
- task-thread work may stage drafts, but only the `user-thread` can approve activation

## See also

- [Runtime](runtime.md)
- [Architecture](architecture.md)
- [Deployment](deployment.md)
