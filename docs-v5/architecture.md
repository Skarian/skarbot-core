# Architecture

This document defines the persistent Skarbot data model: users, threads, tasks, approvals, capabilities, logs, and the directory layout that stores them.

## File System Layout

```text
~/skarbot/
  core/          product code and provided system capabilities
  state/         runtime state
  workspaces/    per-thread working directories and scratch
~/.skarbot/
  admin.json     deployment admin identity
  auth.json      deployment-wide Codex auth token
```

## Canonical runtime tree

```text
~/skarbot/state/
  approvals/
    pending/
      user/
      admin/
    approved/
      user/
      admin/
    denied/
      user/
      admin/
    feedback/
      user/
      admin/
    expired/
      user/
      admin/
  capabilities/
    <user-id>/
      skills/
        active/
        drafts/
      tools/
        active/
        drafts/
  logs/
  tasks/
    <user-id>/
      paused/
      inactive/
      failed/
  users/
    active/
    pending/
    denied/
~/skarbot/workspaces/
  users/
  tasks/
```

## Host-managed files

Skarbot keeps only a small set of deployment-local files under `~/.skarbot`:

- `auth.json` — deployment-wide Codex auth
- `admin.json` — the one allowed admin user id

## Users

Each person has one user record. User membership status is directory-based:

```text
~/skarbot/state/users/
  active/
  pending/
  denied/
```

A user record keeps the same shape and filename as it moves between those directories as part of membership approval.

### User record

```json
{
  "id": "john_smith",
  "name": "John Smith",
  "email": "john@example.com",
  "phone": "+15551234567",
  "timezone": "America/Chicago"
}
```

Rules:

- `id` is derived from the user's first and last name when the record is first created, lowercased, and joined with `_`
- user records are stored as `<id>.json`
- email is trimmed, lowercased, unique, and used for web identity lookup
- phone is stored in E.164, unique, and used for SMS identity lookup
- timezone uses an IANA name and is the default timezone for recurring schedules

Admin status is not stored on the user record. The deployment admin lives in `~/.skarbot/admin.json`.

## pi agent SDK references

Skarbot builds on the `pi` agent SDK. The following sections may reference thread and message structures that come from `pi`.

## Threads

Skarbot has one thread model with two concrete uses:

- a `user-thread` for each approved user
- a `task-thread` for each task

A `user-thread` is the user’s long-lived conversation. A `task-thread` is scoped work with its own history and workspace.

### Thread history and attachments

```text
~/skarbot/workspaces/
  users/
    <user-id>/
      chat/
        history.jsonl
        thread.meta.json
        attachments/
      tasks/
        <task-slug>/
          history.jsonl
          attachments/
  tasks/
    <user-id>/
      <task-slug>/
        chat/
          history.jsonl
          attachments/
```

Naming rules:

- user-thread history lives at `users/<user-id>/chat/history.jsonl`
- user-thread metadata lives at `users/<user-id>/chat/thread.meta.json`
- user-thread attachments live at `users/<user-id>/chat/attachments/`
- task-thread history lives at `tasks/<user-id>/<task-slug>/chat/history.jsonl`
- task-thread attachments live at `tasks/<user-id>/<task-slug>/chat/attachments/`
- owned task history is also exposed at `users/<user-id>/tasks/<task-slug>/history.jsonl` as a runtime-owned read-only view
- owned task attachments are also exposed at `users/<user-id>/tasks/<task-slug>/attachments/` as a runtime-owned read-only view
- the session header `id` matches the user id for a `user-thread` or the owner-scoped task identifier for a `task-thread`

Creation rules:

- a user workspace is created lazily on the first real inbound user message
- `chat/history.jsonl` is created lazily on that same first inbound user message
- `chat/thread.meta.json` is created lazily on that same first inbound user message
- `chat/attachments/` is created lazily on the first user-thread attachment
- the task execution workspace under `workspaces/tasks/...` is created lazily on the first actual run
- `tasks/<user-id>/<task-slug>/chat/history.jsonl` is created lazily on that same first task run
- `tasks/<user-id>/<task-slug>/chat/attachments/` is created lazily on the first task-thread attachment
- `users/<user-id>/tasks/<task-slug>/` is created lazily when that task first becomes visible from the `user-thread`

### Thread file format

Each thread uses one append-only `.jsonl` session file. Full history remains append-only on disk even after compaction.

Thread files use the `pi` session file format defined by [`SessionHeader`](https://github.com/badlogic/pi-mono/blob/576e5e1a2fbe1abbbad96b696f4058cffd8391ca/packages/coding-agent/src/core/session-manager.ts#L29-L36) and [`SessionEntry`](https://github.com/badlogic/pi-mono/blob/576e5e1a2fbe1abbbad96b696f4058cffd8391ca/packages/coding-agent/src/core/session-manager.ts#L137-L146):

- the first line is a `SessionHeader`
- the remaining lines are `SessionEntry` records

In the `pi` session format, the `SessionHeader` contains `type = "session"`, `version`, `id`, `timestamp`, `cwd`, and optional `parentSession`. `SessionHeader.cwd` is the workspace path for that thread.

### Stored message format

Stored conversation records use the `pi` agent message format defined by [`AgentMessage`](https://github.com/badlogic/pi-mono/blob/576e5e1a2fbe1abbbad96b696f4058cffd8391ca/packages/agent/src/types.ts#L241-L245), [`ToolCall`](https://github.com/badlogic/pi-mono/blob/576e5e1a2fbe1abbbad96b696f4058cffd8391ca/packages/ai/src/types.ts#L159-L165), and [`ToolResultMessage`](https://github.com/badlogic/pi-mono/blob/576e5e1a2fbe1abbbad96b696f4058cffd8391ca/packages/ai/src/types.ts#L203-L210). Message-bearing entries use `type = "message"` and `message = <AgentMessage>`.

Core `pi` message rules:

- core message roles are `user`, `assistant`, and `toolResult`
- user messages may use plain text or structured `TextContent` and `ImageContent`
- assistant messages preserve structured content blocks, provider, model, usage, stop reason, thinking, tool calls, and timestamp
- tool use is represented through assistant `ToolCall` content blocks
- tool results use `role = "toolResult"` and carry `toolCallId`, `toolName`, structured `content`, `isError`, and timestamp
- core messages stay free of ad-hoc routing fields
- attachment-bearing content blocks should store attachment filenames that resolve within the owning thread's runtime-owned attachment directory

### User-thread metadata file

A `user-thread` has a tiny metadata file that remembers the latest reply channel:

```json
{ "latest_reply_channel": "web" }
```

This file lives at `~/skarbot/workspaces/users/<user-id>/chat/thread.meta.json`.

Allowed channel names are:

- `web`
- `sms`
- `admin-email`

If the metadata file is missing or unreadable, replies default to `web`.

### Attachments

Each thread has an attachment directory. Saved files keep the original filename recognizable while being renamed as needed for safe storage and uniqueness.

Attachment filenames are sanitized before they are written to disk.

Attachment-capable channels save files locally at ingress under the thread's attachment directory and convert them into standard `pi`-compatible `user.content` blocks.

Attachment references in thread history use stored filenames such as `invoice.pdf`.

When an attachment is needed for work, the agent reads it directly from that thread's own `chat/attachments/`.

When a task-thread history file is read from the `user-thread` task view at `./tasks/<task-slug>/history.jsonl`, those filenames resolve through the matching read-only task view at `./tasks/<task-slug>/attachments/`.

### Compaction in thread history

Compaction is recorded in thread history as a `pi` [`CompactionEntry`](https://github.com/badlogic/pi-mono/blob/576e5e1a2fbe1abbbad96b696f4058cffd8391ca/packages/coding-agent/src/core/session-manager.ts#L66-L74).

Compaction rules that affect stored history:

- the latest compaction summary remains in history
- the next compaction incorporates that summary plus older messages since the last compaction point
- tool-call and tool-result content is truncated during compaction serialization so large outputs do not dominate the summary
- compaction events are visible in thread history and in the web thread viewer only

## Workspaces

Each thread owns one workspace:

```text
~/skarbot/workspaces/
  users/<user-id>/
  tasks/<user-id>/<task-slug>/
```

Every workspace contains a local `MEMORY.md`. See [runtime.md](./runtime.md) for memory behavior and usage rules.

Workspaces may also contain copies or links of thread attachments that are being actively used during execution.

Every thread workspace contains its own canonical `chat/` history tree. User workspaces also contain the runtime-owned `tasks/` read-only views described above. These paths are readable by the agent and not edited directly by the agent.

The runtime enforces those read-only paths. Normal agent file-editing tools and shell commands must not modify them.

Creation and lifecycle rules:

- user-thread workspaces are created lazily on the first actual user-thread action
- task workspaces are created lazily on the first run
- user-thread workspaces persist by default
- task workspaces follow task lifecycle and deterministic cleanup rules

A task workspace is eligible for deterministic deletion only when both conditions are true:

- the matching task file is in `inactive/` or `failed/`
- the workspace root contains an empty `.deleteworkspace` sentinel file

Task workspaces are retained by default. When a one-off task moves to `inactive/`, the runtime prunes known heavy generated directories from the workspace. When a task moves to `failed/`, the runtime prunes those same directories. When a recurring task is disabled and moves to `inactive/`, the runtime may prune those same directories then as well.

## Tasks

Tasks are file-backed and intentionally simple.

```text
~/skarbot/state/tasks/
  <user-id>/
    <task-slug>.json
    paused/
      <task-slug>.json
    inactive/
      <task-slug>.json
    failed/
      <task-slug>.json
```

Lifecycle is defined by directory placement, not by a mutable `status` field.

- active tasks live in `~/skarbot/state/tasks/<user-id>/`
- paused tasks live in `~/skarbot/state/tasks/<user-id>/paused/`
- inactive tasks move to `inactive/` within that user’s task directory
- failed tasks move to `failed/` within that user’s task directory

A task is canonically identified by its owner id plus task filename stem. When a task has a `schedule`, that same owner-scoped identifier is its internal schedule identifier, while the user-facing handle in that owner’s `user-thread` remains the task filename stem.

### Task filename rules

Task filenames are human-readable slugs derived from the task title, such as `morning-briefing.json`.

Rules:

- once created, a task filename stays stable even if the title changes later
- if a slug collision occurs within the same user’s task directory, Skarbot appends `-2`, `-3`, and so on
- the task JSON keeps `title` as the human-facing label
- the filesystem creation time of the task file is the canonical task creation time
- the filesystem modification time reflects later edits
- `created_at` and `updated_at` are derived from the task file's filesystem timestamps

### Task record

```json
{
  "title": "Morning briefing",
  "instructions": "Prepare a weekday briefing for Neil.",
  "owner": "john_smith",
  "notification": ["user-thread"],
  "model": "openai-codex/gpt-5.4",
  "reasoning_effort": "medium",
  "capability": {
    "kind": "skill",
    "name": "briefing"
  },
  "schedule": {
    "cron": "0 7 * * 1-5"
  }
}
```

Required fields:

- `title`
- `instructions`
- `owner`
- `notification`

`schedule` is optional.

`capability` is optional.

`model` is optional.

`reasoning_effort` is optional.

Field rules:

- `owner` stores a stable internal Skarbot user id
- `instructions` is one plain string
- `notification` is a list
- notifications always target the task owner
- if `model` is omitted, the task thread uses the deployment default model
- if `reasoning_effort` is omitted, the task thread uses the deployment default reasoning effort
- `model`, when present, overrides the default model for that task thread
- `reasoning_effort`, when present, overrides the default reasoning effort for that task thread
- `capability`, when present, identifies the custom tool or skill the task is creating or updating

Notification values are:

- `[]` — keep output in the `task-thread` only
- `["user-thread"]`
- `["sms"]`
- `["user-thread", "sms"]`

Schedule forms are:

- omitted — start now in a `task-thread`
- `{ "at": "<ISO timestamp>" }` — run once later
- `{ "at": "<ISO timestamp>", "timezone": "<IANA timezone>" }` — run once later with a timezone override
- `{ "cron": "<expr>" }` — recurring schedule
- `{ "cron": "<expr>", "timezone": "<IANA timezone>" }` — recurring schedule with a timezone override

A scheduled automation is just an active task file with a `schedule` field. There is no separate schedule registry, schedule database, or schedule config layer.

`schedule.at` uses ISO 8601 date-time text.

`schedule.cron` uses standard 5-field cron syntax.

`schedule.timezone` uses an IANA timezone name such as `America/Chicago`.

If `schedule.timezone` is omitted, the task uses the owner's user-record timezone.

If `schedule.at` includes its own timezone, that value wins over `schedule.timezone`.

The scheduler discovers schedules only by scanning active task files. Schedule creation, edit, and deletion are task-file create, edit, and move operations.

Recurring schedules reuse one bound `task-thread`. One-off schedules create throwaway `task-thread`s.

Capability form:

```json
{
  "kind": "tool",
  "name": "weather"
}
```

Allowed `kind` values are:

- `tool`
- `skill`

When `capability` is present, the task is building or updating a custom tool or skill for that user.

### Task thread and workspace creation

Creating a task file does not immediately create the `task-thread`, task session file, or task workspace. Those are all created lazily on the first actual run in that task owner’s namespace.

## Approvals

Skarbot uses one file-backed approval model whose lifecycle is encoded by directory placement:

```text
~/skarbot/state/approvals/
  pending/
    user/
    admin/
  approved/
    user/
    admin/
  denied/
    user/
    admin/
  feedback/
    user/
    admin/
  expired/
    user/
    admin/
```

Each approval file is named `<approval-id>.json`, for example `550e8400-e29b-41d4-a716-446655440000.json`.

### Approval record

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "reply_code": "A14",
  "approver_id": "john_smith",
  "kind": "schedule-edit",
  "summary": "Move the morning briefing to 8:00 AM.",
  "details": "Change weekday cron from 07:00 to 08:00 America/Chicago.",
  "source": "john_smith/morning-briefing",
  "expires_at": "2026-04-01T12:00:00Z"
}
```

Rules:

- `id` is the stable internal UUID
- `reply_code` is the short user-facing code included in approval messages
- `reply_code` only needs to be unique within the active approval set
- replies over web, SMS, and email use `reply_code`, not the UUID
- `approver_id` remains in the record
- `source` stores the source identifier the approval originated from
- task-thread approvals use the owner-scoped task identifier, for example `john_smith/morning-briefing`; user-thread approvals use just the user id, for example `john_smith`
- `expires_at` is optional
- approval handling is deterministic runtime behavior, not model judgment

## Capabilities

Skarbot supports two types of capabilities:

- **skills** — markdown packages
- **tools** — pi extension packages that register named tools

Custom tools and skills are stored under this capabilities tree.

### System capabilities

Repo-backed system capabilities live under:

```text
~/skarbot/core/capabilities/
  skills/
  tools/
```

These are part of the product baseline.

### User capabilities

User-owned capabilities live under runtime state:

```text
~/skarbot/state/capabilities/
  <user-id>/
    skills/
      active/
      drafts/
    tools/
      active/
      drafts/
```

Rules:

- active user capabilities load automatically for that user’s runs
- draft capabilities do not load into normal runs
- workspace-local capabilities may be loaded temporarily in a `task-thread` for testing before they are submitted into `drafts/`
- system capability names are reserved
- user capabilities must have unique names at activation time
- moving a draft to `active/` is a user-thread approval action, not a task-thread action

### Capability package shape

A skill package lives under either `skills/active/<name>/` or `skills/drafts/<name>/` and must contain `SKILL.md`.

A tool package lives under either `tools/active/<name>/` or `tools/drafts/<name>/` as a local `pi` package with `package.json` and at least one declared extension entry under `pi.extensions`. One tool package may register multiple named tools.

## Logs

Deterministic runtime logs live under:

```text
~/skarbot/state/logs/
```

The unauthorized inbound log lives at:

```text
~/skarbot/state/logs/unauthorized-inbound.log
```

Unknown or unauthorized inbound SMS and email attempts are recorded there even when Skarbot rejects them before any model call.
