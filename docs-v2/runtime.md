# Runtime

This document defines how Skarbot composes an agent run: execution profiles, tools, memory, approvals, and autonomy boundaries. Persistent file layout lives in [Architecture](architecture.md). Host setup lives in [Deployment](deployment.md).

## Execution profiles

Skarbot has exactly two execution profiles.

### Direct-thread profile

The direct-thread profile powers the user’s long-lived conversation.

It:

- runs against `~/skarbot/state/threads/<user-id>.jsonl`
- uses `~/skarbot/workspaces/users/<user-id>/`
- loads system capabilities plus the user’s active capabilities
- never loads candidate capabilities
- owns reply routing through the direct-thread sidecar
- owns schedule management

### Task-thread profile

The task-thread profile powers scoped work.

It:

- runs against `~/skarbot/state/threads/task-<task-slug>.jsonl`
- uses `~/skarbot/workspaces/tasks/<task-slug>/`
- loads system capabilities plus the owner’s active capabilities
- may overlay one candidate capability when the task is building or reviewing that capability
- keeps recurring scheduled work on one reusable task thread
- uses throwaway task threads for one-off scheduled runs

A scheduled run is still a task-thread run. A capability-building task is still a task-thread run. Skarbot does not create extra profile types for those cases.

## Tool surface

Skarbot reuses pi’s coding substrate and adds a small system layer.

### Baseline substrate tools

Both execution profiles inherit the baseline coding tools:

- `read`
- `write`
- `edit`
- `bash`
- `grep`
- `find`
- `ls`

Skarbot does not wrap or rename them.

### Shared system tools

Both execution profiles also expose:

| Tool | Purpose |
| --- | --- |
| `web_search` | Find candidate sources on the web |
| `web_fetch` | Retrieve readable content from one URL |
| `plan` | Manage one active plan for the current thread |
| `subagents` | Spawn, list, and kill generic worker subagents in the current thread |
| `model` | Inspect models and switch the current thread model |

Rules:

- `plan` is thread-local and supports one active plan per thread
- `subagents` are generic workers, scoped to the current thread
- active worker names must be unique within that thread
- `model` changes only the current thread, never the deployment default

### Direct-thread tools

The direct-thread profile adds:

| Tool | Purpose |
| --- | --- |
| `schedule` | Create, inspect, edit, and delete user-owned schedules |

Schedules are owned from the direct thread even when they execute as task files under the hood.

### Task-thread tools

The task-thread profile adds:

| Tool | Purpose |
| --- | --- |
| `ask-user` | Ask the owner for missing information or a decision |
| `capabilities` | List, inspect, get, and promote capabilities |

`capabilities` supports exactly four actions:

- `list`
- `inspect`
- `get`
- `promote`

`get` seeds a working bundle from the candidate first and the active capability second. `promote` runs deterministic validation before the candidate is staged for approval.

## Models

Skarbot uses one configured model provider for the deployment. New threads start on the deployment default model. A thread may switch models during its lifetime, and that choice is remembered per thread in session history.

Rules:

- a thread’s model choice does not change any other thread
- resuming a thread restores its last valid model
- the deployment default is a protected runtime setting
- thread usage and cost come from session metadata

Skarbot does not build a fallback matrix or a multi-provider routing layer into the product runtime.

## Memory and history

Memory is deliberately simple.

### Durable memory

Each workspace has a local `MEMORY.md`. That file holds durable notes worth carrying forward inside the same thread workspace.

The agent maintains `MEMORY.md` using the normal coding tools. There is no dedicated memory tool.

### Thread history

Each thread keeps one append-only `.jsonl` history file. Full history stays on disk even after compaction.

### Compaction

Compaction is automatic. Users do not manually manage context windows.

Compaction behavior:

- trigger before the hard context limit when possible
- compact and retry once on overflow
- append a compaction boundary into thread history
- keep the latest rolling summary plus a recent verbatim tail in future context
- truncate oversized tool-call and tool-result content before summarization

Compaction is part of context management, not a substitute for `MEMORY.md`.

## Capability workflow

Capability work happens in task threads.

Flow:

1. Create or resume a capability-building task.
2. Seed the task workspace from the candidate capability if one exists, otherwise from the active capability if one exists.
3. Edit the working bundle inside the task workspace.
4. Run `capabilities promote`.
5. If validation fails, return diagnostics to the task immediately.
6. If validation succeeds, stage the candidate and request user approval.
7. On approval, replace the active capability and delete the candidate.

Newly approved capabilities become visible on the next run or turn. Skarbot does not hot-swap them into a run that is already in progress.

## Approvals

Skarbot uses one approval system with two approver classes:

- `user`
- `admin`

Approvals are deterministic runtime objects, not model guesses.

### Reply grammar

Every approval has a short id such as `A14`. Every approval message includes the accepted actions:

- `approve <id>`
- `deny <id>`
- `feedback <id>: <text>`

Different surfaces may render different UI, but they map to the same actions.

### User approvals

User approvals are delivered into the owner’s direct thread.

Typical user approvals include:

- capability promotion
- schedule creation
- schedule edits
- schedule deletion
- other user-facing behavior changes that need confirmation

### Admin approvals

Admin approvals are rare and operational. They are delivered through the admin email path.

Typical admin approvals include:

- host package installs
- new outbound destinations such as SMS recipients
- `exe.dev` sharing or integration changes
- destructive actions outside owned roots
- deployment-wide runtime changes

A task may not open an admin approval on its own when the work belongs to a user. It must first ask the user whether they want to escalate the request.

### Waiting behavior

A blocked task pauses against the pending approval id.

When the approval resolves:

- `approve` resumes the task with an approved result
- `deny` resumes the task with a denied result
- `feedback` resumes the task with the feedback content
- `expired` resumes the task as a denied-equivalent outcome

## Autonomy boundary

Skarbot is autonomous by default only inside its owned runtime roots:

- `~/skarbot/state`
- `~/skarbot/workspaces`

It is **not** autonomous in:

- `~/skarbot/core`
- host package management
- deployment-sharing and integration settings
- destructive operations outside owned roots

Additional rules:

- product code changes go through normal git workflow
- the running agent does not self-merge
- outbound SMS is allowed only to explicitly allowlisted numbers
- SMS is not an approval channel

## See also

- [Architecture](architecture.md)
- [Deployment](deployment.md)
- [Product](product.md)
