# Runtime

Skarbot runs in two execution profiles: `user-thread` and `task-thread`.

Threads persist across turns.

## Runtime building blocks

Every Skarbot run is assembled from the same core runtime state.

#### Thread history

Each thread stores its canonical history in `./chat/history.jsonl` inside that thread's own workspace, using the `pi` session format defined by [`SessionHeader`](https://github.com/badlogic/pi-mono/blob/576e5e1a2fbe1abbbad96b696f4058cffd8391ca/packages/coding-agent/src/core/session-manager.ts#L29-L36). `SessionHeader.cwd` is the workspace path for that thread.

In the `user-thread` workspace, owned task histories are also available under `./tasks/<task-slug>/history.jsonl` as runtime-owned read-only views into those task workspaces.

Thread history remains append-only on disk even after compaction.

#### Workspace

Each thread has its own workspace. That workspace is the working directory for the turn and the home of its durable local memory in `MEMORY.md`.

Each thread workspace also includes these runtime-owned paths:

```text
./chat/history.jsonl
./chat/attachments/
```

The `user-thread` workspace also includes these runtime-owned read-only task views:

```text
./tasks/<task-slug>/history.jsonl
./tasks/<task-slug>/attachments/
```

The runtime enforces these read-only paths. Normal agent file-editing tools and shell commands must not modify them.

#### Capabilities

Each turn starts with the built-in system tools and skills together with the user's active tools and skills.

These are referred to collectively as capabilities in this documentation.

Users can create new capabilities using a task. After user approval, the new capability becomes active for future `user-thread` and `task-thread` runs.

#### Attachments

Threads may include attachment references in their history.

Attachment-bearing history entries store attachment filenames. For the current thread, those filenames resolve under `./chat/attachments/`. In the `user-thread` workspace, owned task attachments are also available under `./tasks/<task-slug>/attachments/`.

#### Task-backed state

`task-thread` workspaces include a local `TASK.md` copy of the current task details.

#### Thread state

Each thread may be idle, active, or waiting.

## Execution profiles

Skarbot has two execution profiles: `user-thread` and `task-thread`.

#### User-thread

The `user-thread` profile powers a user's long-lived conversation with Skarbot. It owns reply-channel memory and task control for that user.

#### Task-thread

The `task-thread` profile powers scoped work such as one-off tasks, recurring scheduled tasks, and work to create or update custom tools and skills. Task-thread runs may also use task-backed state and task-local preview when the task is creating a custom tool or skill for a user.

## Turn triggers

A Skarbot turn begins when new work enters the runtime or when waiting work is resumed.

Current triggers are:

- user messages from web
- user messages from SMS
- admin approval replies over email
- scheduled task execution
- user approval replies that resume paused work

## Channel and thread routing

The user's `user-thread` is the only required user-facing control surface. Users should be able to manage all of their work from that thread, including over SMS.

#### Execution

Scoped work runs in `task-thread` turns. A `task-thread` is the canonical execution log for that task.

User messages from web and SMS land in the owner's `user-thread`.

Scheduled execution starts in the bound task thread. Admin approval replies over email resume the approval-bound work they refer to.

#### Control

Users can ask for task status, answer follow-up questions, approve changes, pause tasks, resume tasks, cancel tasks, or change task instructions from the `user-thread`.

The user-thread agent must be able to inspect and control task-thread work through deterministic runtime tools.

The user-thread agent can also read its own conversation history from `./chat/` and owned task histories from `./tasks/`. The `./tasks/` paths are runtime-owned and read-only to the agent.

Waiting for user input or approval returns control to the `user-thread` and resumes later on the waiting thread as a new turn.

#### Delivery

Replies to ordinary user conversation go back through the user's current reply channel.

When a task needs input, requests approval, or reports a result, the runtime surfaces that interaction through the user's `user-thread`.

A user reply in the `user-thread` resumes or controls the waiting task thread when the runtime binds that reply to the originating task.

If a user sends input to an active task, the runtime records it on that task thread and delivers it at the next runtime checkpoint between tool and model steps.

#### Prompt sequencing

Task-originated questions, approvals, and other requests for user input appear in the user's `user-thread`.

The runtime presents at most one active user prompt at a time for a given user. If multiple prompts become ready at once, it should group them when clear, otherwise queue them and present them one at a time.

## Turn lifecycle

Each turn is a bounded execution against an existing thread.

#### Start

A turn starts when new work enters the runtime or when waiting work is resumed.

At the start of a turn, the runtime loads the current thread history, workspace, model state, built-in tools, and active capabilities for that thread.

#### Execute

The turn executes in the current thread context and uses the tool surface defined in [Tools](tools.md).

A turn may perform multiple model and tool iterations before it completes.

The runtime owns the iteration budget, timeout policy, retry policy, and loop detection for that turn.

#### Pause

A turn pauses when it reaches a waiting state such as waiting for user input, waiting for approval, or waiting for another runtime-managed result.

The waiting state is recorded on the thread and the active turn ends.

#### Resume

When the blocking state resolves, the runtime starts a new turn on the same thread with the resolved result.

#### Interrupt

A turn may also be interrupted by runtime control, new user input, shutdown, restart, or timeout.

Interrupt ends the active turn attempt without discarding the thread. The runtime may later start a new turn on that same thread.

#### Complete

At the end of the turn, the runtime persists the resulting thread state and emits any replies or notifications through the correct channel.

## Model Providers

Skarbot only supports using OpenAI's subscription as a model provider today.

Default model is `openai-codex/gpt-5.4` with `medium` reasoning effort. Users can adjust the model in their `user-thread` by asking Skarbot. Model selection is persisted in the thread's history.

Task threads normally use the default model and reasoning effort too. If the task file specifies a model override, the runtime uses that model for the task thread instead.

An administrator can set or refresh the subscription auth token using `skarbot auth`, which generates a link for the administrator to open and sign in. The auth token is stored in `~/.skarbot/auth.json`.

If a thread resumes with a model that is no longer available, the runtime falls back to another available model.

## Tasks, schedules, and long-running work

Tasks are units of work executed outside the `user-thread`. They keep scoped work in a separate thread and workspace.

If Skarbot determines that a request is complex enough to deserve its own workspace, it asks follow-up questions in the `user-thread` and then creates a task file.

#### Task start modes

Tasks support three start modes:

- **Immediate**: no `schedule`. Starts immediately in its own `task-thread`.
- **One-off scheduled**: `schedule.at` is set. Runs once at the requested time.
- **Recurring**: `schedule.cron` is set. Runs repeatedly on that schedule.

The scheduler discovers scheduled work by scanning active task files under `~/skarbot/state/tasks/<user-id>/`. There is no separate schedule registry.

#### Task threads and workspaces

Each active task runs against its own task thread and task workspace.

Recurring tasks reuse the same task thread and workspace across runs.

One-off scheduled tasks may use a throwaway task thread and workspace for the scheduled run.

#### Task file lifecycle

Task files are stored under:

```text
~/skarbot/state/tasks/<user-id>/
```

When a task finishes successfully, its task file moves to:

```text
~/skarbot/state/tasks/<user-id>/inactive/
```

When a task fails, its task file moves to:

```text
~/skarbot/state/tasks/<user-id>/failed/
```

When a task is paused, its task file moves to:

```text
~/skarbot/state/tasks/<user-id>/paused/
```

Disabling a recurring task also moves its task file to `inactive/`.

#### Workspace cleanup

Task workspaces are retained by default.

When a one-off task moves to `inactive/`, the runtime prunes known heavy generated directories from the workspace. When a task moves to `failed/`, the runtime prunes those same directories so the workspace stays diagnosable without retaining large dependency trees and build artifacts.

Recurring task workspaces are retained while the task remains active. When a recurring task is disabled and moves to `inactive/`, the runtime may prune those same directories then as well.

The prune list is deterministic. It includes:

- `node_modules/`
- `.venv/`
- `venv/`
- `target/`
- `dist/`
- `build/`
- `.next/`
- `.pytest_cache/`
- `__pycache__/`
- `.mypy_cache/`
- `.ruff_cache/`
- `.gradle/`

Full workspace deletion is separate and admin-only. It only happens when an administrator adds an empty `.deleteworkspace` file to the workspace root and the matching task file is in `inactive/` or `failed/`.

The cleanup command runs locally on startup and then from cron every 30 minutes.

## Custom tools and skills

Users can add custom tools and skills to Skarbot beyond what was provided by default.

That work happens in a `task-thread`, not in the `user-thread`. The `task-thread` gets its own workspace so Skarbot can edit, test, and validate the work without affecting the user's main workspace.

If the user is updating an existing custom tool or skill, the task starts from the current `drafts/` version if one exists. Otherwise it starts from the `active/` version. If neither exists, Skarbot creates it from scratch in the task workspace.

While the task is running, Skarbot can load the in-progress tool or skill inside that `task-thread` for testing and preview. It is only available in that `task-thread` until the user approves it.

When the work is ready, Skarbot runs deterministic validation.

If validation fails, the task stays in the `task-thread` and Skarbot continues from the validation output.

If validation succeeds, Skarbot writes the validated result to the user's `drafts/` area and asks for approval in the `user-thread`.

If the user approves it, the draft moves to `active/` and becomes available on the next run.

If the user denies it or gives feedback, it does not become active. The draft stays in `drafts/` so Skarbot can keep working on it later.

## Memory and compaction

Every thread workspace has its own `MEMORY.md`.

`MEMORY.md` holds durable notes for that workspace. The `user-thread` workspace has one. Each `task-thread` workspace has one too.

Skarbot updates `MEMORY.md` using the normal coding tools.

Thread history stays in the thread file. It remains append-only on disk after compaction.

Before compaction, the runtime may run one silent memory flush turn on the same thread. That turn asks Skarbot to write any durable notes from the recent work into `MEMORY.md`.

The memory flush runs when estimated thread context exceeds `contextWindow - reserveTokens - memoryFlushThreshold`. In other words, Skarbot starts the memory flush shortly before normal compaction would begin.

Default memory and compaction settings are:

- `reserveTokens: 20000`
- `keepRecentTokens: 20000`
- `memoryFlushThreshold: 4000`
- one memory flush per compaction cycle

Compaction runs when estimated thread context exceeds `contextWindow - reserveTokens`. `reserveTokens` is the headroom the runtime keeps free for the next prompt and response.

When compaction runs, the runtime writes a compaction summary into the thread history and keeps the newest entries in full. Future turns load that summary together with those newer entries.

Compaction usually happens between turns. If one turn is too large, the runtime may summarize the earlier part of that turn and keep the newer part in full.

Large tool-call and tool-result content is truncated before summarization so the compaction summary stays focused on the useful state of the thread.

If a turn hits context overflow, the runtime may compact and retry once.

## Approvals and pause or resume behavior

Skarbot has an approval system with two approver classes:

- `user`
- `admin`

User approvals appear in the owner's `user-thread`. Users can approve them from web or SMS.

Admin approvals are sent over admin email.

#### Reply format

Each approval has:

- a stable internal UUID
- a short reply code such as `A14`

The short reply code is what users see and reply with.

Web can render buttons, but the canonical text replies are:

- `approve A14`
- `deny A14`
- `feedback A14: <text>`

#### Waiting behavior

When a thread reaches an approval point, the current turn pauses and the pending approval is recorded on that thread.

When the approval resolves, the runtime starts a new turn on that same thread with the approval result.

- `approve` resumes with an approved result
- `deny` resumes with a denied result
- `feedback` resumes with the feedback text
- `expired` resumes as a denied result

`feedback` resolves that approval. If Skarbot needs approval again after making changes, it creates a new approval with a new reply code.

#### Approval lifecycle

Resolving an approval moves it out of `pending/` and into one of these directories:

- `approved/`
- `denied/`
- `feedback/`
- `expired/`

#### Typical uses

User approvals cover user-facing changes such as:

- schedule creation
- schedule edits
- schedule deletion
- activating custom tools or skills

Admin approvals cover operator actions such as:

- host package installs
- new outbound destinations
- destructive actions outside owned roots
- deployment-wide runtime changes

If user-owned work needs admin approval, Skarbot asks the user first. It only creates the admin approval after the user agrees to escalate.

## Autonomy

Skarbot can act autonomously in `~/skarbot/state/` and `~/skarbot/workspaces/`.

Admin approval is required for any changes outside those folders and for host package management.
