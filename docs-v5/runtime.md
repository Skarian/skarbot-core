# Runtime

This document defines how Skarbot composes a run: execution profiles, tool contracts, model behavior, task execution, memory, approvals, and autonomy boundaries. Persistent file layout lives in [Architecture](architecture.md). Host setup lives in [Deployment](deployment.md).

## Execution model

Skarbot has two execution profiles:

- `user-thread` for a user's long-lived conversation
- `task-thread` for scoped work such as one-off tasks, recurring scheduled tasks, and work to create new tools and skills for that user's workspace

## Shared runtime contract

Both profiles share the same runtime contract:

- Thread history is stored as append-only `pi` session data
  - Thread history uses the `pi` session format defined by [`SessionHeader`](https://github.com/badlogic/pi-mono/blob/576e5e1a2fbe1abbbad96b696f4058cffd8391ca/packages/coding-agent/src/core/session-manager.ts#L29-L36)
  - `SessionHeader.cwd` is the thread workspace path
  - Full history remains on disk even after compaction
- Workspace state lives in a dedicated workspace per thread
  - Each thread has its own workspace
  - Each workspace keeps durable memory in a local `MEMORY.md`
- Capabilities are loaded fresh from the canonical filesystem roots
  - System capabilities load from `~/skarbot/core/capabilities/`
  - The owner's active capabilities load from `~/skarbot/state/capabilities/<user-id>/`
  - Active capabilities are re-resolved at the start of each run or turn

## User-thread profile

The user-thread profile powers a user’s long-lived main conversation.

- Storage
  - Thread file: `~/skarbot/state/threads/users/<user-id>/thread.jsonl`
  - Metadata file: `~/skarbot/state/threads/users/<user-id>/thread.meta.json`
  - Attachments: `~/skarbot/state/threads/users/<user-id>/attachments/`
  - Workspace: `~/skarbot/workspaces/users/<user-id>/`

- Behavior
  - Loads system capabilities plus the owner’s active capabilities
  - Reply routing follows the user-thread metadata file
  - Schedule management is owned from the user thread

## Task-thread profile

The task-thread profile powers every scoped task.

- Storage
  - Thread file: `~/skarbot/state/threads/tasks/<user-id>/<task-slug>.jsonl`
  - Attachments: `~/skarbot/state/threads/tasks/<user-id>/<task-slug>.attachments/`
  - Workspace: `~/skarbot/workspaces/tasks/<user-id>/<task-slug>/`

- Behavior
  - Loads system capabilities plus the owner’s active capabilities
  - A task that is creating a capability for a user may stage that capability in its own task thread and load it there temporarily for testing and preview
  - Recurring scheduled runs reuse their bound task thread
  - One-off schedules use throwaway task threads

## Tool contracts

Skarbot reuses `pi`'s built-in coding tools and adds a small system layer. The detailed tool contract lives in [Tools](tools.md).

- Shared tools
  - Both execution profiles expose the baseline coding tools plus shared system tools such as web access, planning, subagents, and model selection

- User-thread tools
  - The user-thread profile adds schedule management

- Task-thread tools
  - The task-thread profile adds `ask-user(question)` and `capabilities(...)`

## Model behavior

Skarbot uses one deployment provider: `openai-codex`. New threads start on the repo-defined deployment default model. A thread may switch models during its lifetime, and that choice is remembered per thread in session history.

Rules:

- a thread’s model choice does not change any other thread
- resuming a thread restores its last selected model when that model remains available
- resuming a thread from a non-web channel still uses that thread’s persisted current model
- available models are discovered from the provider at runtime
- thread usage and cost come from session metadata
- the current model should be visible in thread or session UI
- thread or session summaries should be able to show tokens and cost for that thread

Skarbot does not build a generated model catalog, an instance-level model list, a fallback matrix, or a multi-provider routing layer into the product runtime.

## Compaction and memory

Memory is deliberately simple.

### Durable memory

Every workspace has a local `MEMORY.md`. That file holds compact durable notes worth carrying forward inside that same workspace.

The agent maintains `MEMORY.md` through the normal coding tools. There is no dedicated memory tool.

### What compaction is

Compaction is automatic context management, not memory storage.

Compaction behavior:

- it triggers proactively near the context limit
- it appends a compaction boundary into thread history
- it keeps the latest rolling summary plus a recent verbatim tail in future context
- it truncates oversized tool-call and tool-result content before summarization
- it supports one overflow-recovery compact-and-retry path

Compaction preserves access to full append-only history on disk. It does not replace `MEMORY.md`.

### What belongs in memory versus history

The repo baseline in `~/skarbot/core` is not memory. It is the checked-in product contract.

Routine outputs, run-by-run logs, and scheduled-task output stay in thread history by default. `MEMORY.md` is for durable notes that should help future turns in the same workspace.

## Task execution rules

Tasks use the simplest viable runtime model.

Creation rules:

- if Skarbot does not have enough information to create a task, it asks follow-up questions and does not create the file yet
- a task is not created until its notification list is defined
- ordinary tasks do not require approval before activation
- tasks with no `schedule` start immediately in their own task thread rather than in the user thread
- user-owned schedules are managed from the owner’s user thread even when the underlying execution is represented by a task file

Execution rules:

- the scheduler reads only active task files from each owner directory under `~/skarbot/state/tasks/`
- scheduled work is discovered only by scanning those active task files for a `schedule` field
- each task maps to exactly one task thread while it is active
- recurring schedules reuse one dedicated task thread
- one-off later schedules use throwaway task threads

Completion and cleanup rules:

- after a one-off task succeeds, its task file moves to `~/skarbot/state/tasks/<user-id>/inactive/`
- if a task fails, its task file moves to `~/skarbot/state/tasks/<user-id>/failed/`
- disabling a recurring task moves it to `~/skarbot/state/tasks/<user-id>/inactive/`
- a successful task that should discard its workspace writes an empty `.deleteworkspace` file at the workspace root before or when it moves inactive
- failed task workspaces are retained
- workspace cleanup is performed by a deterministic local cleanup command, not by the LLM
- a workspace is deleted only when the matching task file is inactive and the `.deleteworkspace` sentinel exists
- the cleanup path is zero-token and exits immediately when no deletion markers exist

## Capability workflow

Capability work happens in task threads.

Flow:

1. Create or resume a capability-building task.
2. Seed the task workspace from the candidate capability if one exists, otherwise from the active capability if one exists.
3. Edit the working bundle inside the task workspace.
4. Run `capabilities(promote, ...)`.
5. If validation fails, return diagnostics to the task immediately.
6. If validation succeeds, stage the candidate and request user approval.
7. On approval, replace the active capability and delete the candidate.

Newly approved capabilities become visible on the next run or turn. Skarbot does not hot-swap them into a run that is already in progress.

## Approvals

Skarbot uses one approval system with two approver classes:

- `user`
- `admin`

Approvals are deterministic runtime objects, not model guesses. Approval record storage and path-based lifecycle live in [Architecture](architecture.md).

Resolving an approval moves its file out of `pending/` and into exactly one of `approved/`, `denied/`, `feedback/`, or `expired/` under the correct approver class.

### Canonical reply grammar

Every approval has a stable UUID id such as `550e8400-e29b-41d4-a716-446655440000`. Every approval message includes the accepted actions:

- `approve <id>`
- `deny <id>`
- `feedback <id>: <text>`

Different channels may render richer UI, but they map to the same actions.

### User approvals

User approvals are delivered into the owner’s user thread and may be resolved over web or SMS.

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

### Feedback resolution rule

`feedback <id>: <text>` is terminal for that approval record.

When Skarbot receives feedback:

- the approval file moves from `pending/<class>/` to `feedback/<class>/`
- the feedback text is written back into the source task or source thread
- the blocked task or thread resumes with that feedback payload
- any later revised approval is created as a new approval id, not as a silent reuse of the old one

This keeps approval state deterministic and removes the ambiguity between “still pending” and “resumed with feedback.”

### Waiting behavior

A task blocked on an approval pauses against that pending approval id.

When the approval resolves:

- `approve` resumes the task with an approved result
- `deny` resumes the task with a denied result
- `feedback` resumes the task with the feedback content
- `expired` resumes the task as a denied-equivalent outcome

Approval resolution flows back into the source task or thread deterministically.

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
- SMS may carry user approvals, but not admin approvals

## See also

- [Architecture](architecture.md)
- [Deployment](deployment.md)
- [Product](product.md)
