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
- `subagents(spawn, ...)` returns the active handle that later `kill` calls use
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

### Thread inspection and attachments

The runtime will likely need a small structured thread-inspection surface plus attachment import.

The exact contract still needs to be refined, but the current direction is:

- `thread_events(since?, limit?, type?, query?)`
- `attachments_import(id, path?)`

Current notes:

- attachment references should live in thread history rather than being discovered by scanning storage
- the runtime may store canonical attachment blobs separately from thread workspaces
- `attachments_import(...)` should stage an attachment into the active workspace rather than exposing raw cross-thread filesystem paths

## User-thread tools

The user-thread profile adds grouped schedule management:

- `schedule()`
- `schedule(set, at)`
- `schedule(set, cron)`
- `schedule(edit, handle, at)`
- `schedule(edit, handle, cron)`
- `schedule(delete, handle)`

Rules:

- `schedule()` lists the user’s existing scheduled task files by schedule handle
- the schedule handle is the task filename stem
- `schedule(set, at)` creates a new active task file with a one-off `schedule.at`
- `schedule(set, cron)` creates a new active task file with a recurring `schedule.cron`
- `schedule(edit, handle, at)` edits the matching active task file into a one-off schedule
- `schedule(edit, handle, cron)` edits the matching active task file into a recurring schedule
- `schedule(delete, handle)` moves the matching active task file into that owner's `inactive/` directory
- a schedule is not a separate object; it is an active task file with a `schedule` field
- the scheduler discovers schedules only by scanning active task files
- schedule mutations require explicit user approval before persistence
- recurring schedules default to the owner’s configured timezone when none is provided

The user-thread also exposes grouped review and approval for custom tools and skills:

- `capabilities()`
- `capabilities(review, kind, name)`
- `capabilities(approve, kind, name)`
- `capabilities(deny, kind, name)`
- `capabilities(feedback, kind, name, text)`

Rules:

- `capabilities()` lists the user's custom tools and skills, including draft and active versions
- `capabilities(review, ...)` shows the current draft that is waiting for user review
- `capabilities(approve, ...)` moves the matching draft to `active/`
- `capabilities(deny, ...)` keeps the draft but does not activate it
- `capabilities(feedback, ...)` keeps the draft and sends revision feedback back to the task thread
- activation only happens from the `user-thread`

## User-thread task control

The user-thread profile should also expose a grouped task-control tool for inspecting and controlling task-thread work from the user's `user-thread`.

This is a deterministic control plane. It should not rely on freeform agent-to-agent conversation.

The exact contract still needs to be finalized, but it should cover operations such as:

- listing the user's tasks
- reading task status
- pausing a task
- resuming a task
- canceling a task
- updating task instructions
- replying to a task that is waiting for user input or approval

The runtime should apply these operations as structured task events or task-state changes bound to the target task thread.

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
