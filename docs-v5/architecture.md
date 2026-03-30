# Architecture

This document defines the persistent Skarbot data model: users, threads, tasks, approvals, capabilities, logs, and the directory layout that stores them. User-facing behavior lives in [Product](product.md). Runtime behavior lives in [Runtime](runtime.md). Host setup lives in [Deployment](deployment.md).

## Top-level layout

```text
~/skarbot/
  core/          product code and repo-backed system capabilities
  state/         canonical runtime state
  workspaces/    per-thread working directories and scratch
~/.skarbot/
  admin.json     deployment admin identity
  auth.json      deployment-wide Codex auth
  instance.toml  generated instance-local config
```

`~/skarbot/core` is the product. `~/skarbot/state` is the durable runtime. `~/skarbot/workspaces` is execution space, not product source. Files under `~/.skarbot` are host-managed deployment files, not portable runtime state.

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
    users/
    candidates/
      users/
  logs/
  tasks/
    inactive/
    failed/
  threads/
  users/
    active/
    pending/
    denied/
~/skarbot/workspaces/
  users/
  tasks/
```

This tree is the contract. If a runtime file matters, it lives under one of these roots.

## Host-managed files

Skarbot keeps only a small set of deployment-local files under `~/.skarbot`:

- `auth.json` — deployment-wide Codex auth
- `admin.json` — the one allowed admin user id
- `instance.toml` — generated local instance config that cannot be expressed cleanly through checked-in defaults or filesystem placement

There is no generic `~/skarbot/state/config/` authority layer.

## Users

Each person has one user record. User lifecycle is directory-based:

```text
~/skarbot/state/users/
  active/
  pending/
  denied/
```

A user record keeps the same shape and filename as it moves between those directories.

### User record

```json
{
  "id": "neil",
  "name": "Neil",
  "email": "neil@example.com",
  "phone": "+15551234567",
  "timezone": "America/Chicago"
}
```

Rules:

- `id` is a stable internal identifier created once
- user records are stored as `<id>.json`
- email is trimmed, lowercased, unique, and used for web identity lookup
- phone is stored in E.164, unique, and used for SMS identity lookup
- timezone uses an IANA name and is the default timezone for recurring schedules

Admin status is not stored on the user record. The deployment admin lives in `~/.skarbot/admin.json`.

## Threads

Skarbot has one thread model with two concrete uses:

- a direct thread for each approved user
- a task thread for each task

A direct thread is the user’s long-lived conversation. A task thread is scoped work with its own history and workspace.

### Thread storage

```text
~/skarbot/state/threads/
  <user-id>.jsonl
  <user-id>.meta.json
  <user-id>.attachments/
  task-<task-slug>.jsonl
  task-<task-slug>.attachments/
```

Naming rules:

- direct-thread files use `<user-id>.jsonl`
- direct-thread sidecars use `<user-id>.meta.json`
- direct-thread attachments live in `<user-id>.attachments/`
- task-thread files use `task-<task-slug>.jsonl`
- task-thread attachments live in `task-<task-slug>.attachments/`
- the session header `id` matches the thread file stem

Creation rules:

- a direct-thread session file is created lazily on the first real inbound user message
- a direct-thread sidecar is created lazily beside the thread file on that same first inbound user message
- a direct-thread attachment directory is created lazily on the first attachment
- a task-thread session file is created lazily on the first actual run
- a task-thread attachment directory is created lazily on the first attachment
- task threads do not use a `.meta.json` sidecar

### Session file format

Each thread uses one append-only `.jsonl` session file. Full history remains on disk even after compaction. History is never rewritten into a custom transcript format.

Thread files follow the same pi session-file format and message model used by the runtime:

- the first line is a `SessionHeader`
- the remaining lines are `SessionEntry` objects

The `SessionHeader` contains `type = "session"`, `version`, `id`, `timestamp`, `cwd`, and optional `parentSession`. `SessionHeader.cwd` is the workspace path for that thread.

### Message constraints

Message-bearing entries use `type = "message"` and `message = <AgentMessage>`.

Core message rules:

- core message roles are `user`, `assistant`, and `toolResult`
- user messages may use plain text or structured `TextContent` and `ImageContent`
- assistant messages preserve structured content blocks, provider, model, usage, stop reason, thinking, tool calls, and timestamp
- tool use is represented through assistant `ToolCall` content blocks
- tool results use `role = "toolResult"` and carry `toolCallId`, `toolName`, structured `content`, `isError`, and timestamp
- core messages stay free of ad-hoc routing fields

Direct-thread routing metadata lives in the sidecar file, not inside the message payload.

### Direct-thread sidecar

A direct thread has a tiny sidecar file that remembers the latest reply surface:

```json
{ "latest_reply_channel": "web" }
```

Allowed channel names are:

- `web`
- `sms`
- `admin-email`

If the sidecar is missing or unreadable, replies default to `web`.

### Attachments

Each thread has a sibling attachment directory. Saved files preserve the original filename visibly, but get a millisecond timestamp prefix for uniqueness, for example:

```text
1732531234567_invoice.pdf
```

Attachment filenames are sanitized before they are written to disk.

Attachment-capable channels save files locally at ingress and convert them into standard substrate-compatible `user.content` blocks. Skarbot does not use a custom transcript-only attachment field.

### Compaction in thread history

Compaction is serialized into thread history with the substrate-compatible compaction entry shape. It does not replace, rewrite, or delete older entries.

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
  tasks/<task-slug>/
```

Every workspace contains a local `MEMORY.md`. Workspaces are thread-local. A direct-thread workspace is not the same thing as that user’s capability directory.

Creation and lifecycle rules:

- direct-thread workspaces are created lazily on the first actual direct-thread action
- task workspaces are created lazily on the first run
- direct-thread workspaces persist by default
- task workspaces follow task lifecycle rather than a free-form pruning policy

A task workspace is eligible for deterministic deletion only when both conditions are true:

- the matching task file is inactive
- the workspace root contains an empty `.deleteworkspace` sentinel file

Failed task workspaces are retained.

## Tasks

Tasks are file-backed and intentionally simple.

```text
~/skarbot/state/tasks/
  *.json
  inactive/
  failed/
```

Lifecycle is defined by directory placement, not by a mutable `status` field.

- active tasks live in `~/skarbot/state/tasks/`
- inactive tasks move to `inactive/`
- failed tasks move to `failed/`

The task filename stem is the canonical task identifier. When a task has a `schedule`, that same filename stem is also the schedule handle.

### Task filename rules

Task filenames are human-readable slugs derived from the task title, such as `morning-briefing.json`.

Rules:

- once created, a task filename stays stable even if the title changes later
- if a slug collision occurs within the target directory, Skarbot appends `-2`, `-3`, and so on
- the task JSON keeps `title` as the human-facing label
- the task JSON does not need a separate `id` field
- the filesystem creation time of the task file is the canonical task creation time
- the filesystem modification time reflects later edits
- the task JSON does not carry `created_at` or `updated_at`

### Task record

```json
{
  "title": "Morning briefing",
  "instructions": "Prepare a weekday briefing for Neil.",
  "owner": "neil",
  "notification": ["direct-thread"],
  "schedule": {
    "cron": "0 7 * * 1-5",
    "timezone": "America/Chicago"
  }
}
```

Required fields:

- `title`
- `instructions`
- `owner`
- `notification`

`schedule` is optional.

Field rules:

- `owner` stores a stable internal Skarbot user id
- `instructions` is one plain string
- `notification` is a list
- notifications always target the task owner

Notification values are:

- `[]` — keep output in the task thread only
- `["direct-thread"]`
- `["sms"]`
- `["direct-thread", "sms"]`

Schedule forms are:

- omitted — start now in a task thread
- `{ "at": "<ISO timestamp>" }` — run once later
- `{ "cron": "<expr>", "timezone": "<IANA zone>" }` — recurring schedule

A scheduled automation is just an active task file with a `schedule` field. There is no separate schedule registry, schedule database, or schedule config layer.

If a recurring schedule does not explicitly set a timezone, it defaults from the owner’s user record.

The scheduler discovers schedules only by scanning active task files. Schedule creation, edit, and deletion are task-file create, edit, and move operations.

Recurring schedules reuse one bound task thread. One-off schedules create throwaway task threads.

### Task thread and workspace creation

Creating a task file does not immediately create the task thread, task session file, or task workspace. Those are all created lazily on the first actual run.

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

Each approval file is named `<approval-id>.json`, for example `A14.json`.

### Approval record

```json
{
  "id": "A14",
  "approver_id": "neil",
  "kind": "schedule-edit",
  "summary": "Move the morning briefing to 8:00 AM.",
  "details": "Change weekday cron from 07:00 to 08:00 America/Chicago.",
  "source_thread": "task-morning-briefing",
  "source_task": "morning-briefing",
  "expires_at": "2026-04-01T12:00:00Z"
}
```

Rules:

- the path encodes lifecycle state and approver class
- `approver_id` remains in the record
- the record does not use mutable `status` or `approver_type` fields as the primary source of truth
- `source_task` is optional
- `expires_at` is optional
- the filesystem creation time of the approval file is the canonical approval creation time
- moving the file between lifecycle directories is the canonical resolution event
- approval ids are short, stable, and unique
- approval handling is deterministic runtime behavior, not model judgment

## Capabilities

Skarbot supports two capability kinds:

- **skills** — markdown-first capability packages
- **tools** — pi extension packages that register named tools

### System capabilities

Repo-backed system capabilities live under:

```text
~/skarbot/core/capabilities/
  skills/
  tools/
```

These are part of the product baseline.

### User capabilities

User-promoted capabilities live outside git:

```text
~/skarbot/state/capabilities/
  users/<user-id>/
    skills/
    tools/
  candidates/users/<user-id>/
    skills/
    tools/
```

Rules:

- active user capabilities load automatically for that user’s runs
- candidate capabilities do not load into normal runs
- a capability-building task may preview its own candidate in that task thread only
- user capabilities override same-named system capabilities

### Capability package shape

A skill lives in `skills/<name>/` and must contain `SKILL.md`.

A tool capability lives in `tools/<name>/` as a local pi package with `package.json` and at least one declared extension entry under `pi.extensions`. One tool package may register multiple named tools.

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
