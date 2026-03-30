# Architecture

This document defines the persistent Skarbot data model: users, threads, tasks, capabilities, and the directory layout that stores them. User-facing behavior lives in [Product](product.md). Agent execution rules live in [Runtime](runtime.md).

## Top-level layout

```text
~/skarbot/
  core/          product code and repo-backed system capabilities
  state/         canonical runtime state
  workspaces/    per-thread working directories and scratch
~/.skarbot/
  auth.json      deployment-wide Codex auth
```

`~/skarbot/core` is the product. `~/skarbot/state` is the durable runtime. `~/skarbot/workspaces` is execution space, not product source.

## Users

Each approved person has one user record. User lifecycle is directory-based:

```text
~/skarbot/state/users/
  active/
  pending/
  denied/
```

A user record stays the same shape in all three directories and keeps the same filename when it moves between them.

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
- timezone uses an IANA name and is the default timezone for scheduled work

Admin status is not stored on the user record. The deployment admin lives in `~/skarbot/state/config/admin.json`.

## Threads

Skarbot has one thread model with two concrete uses:

- a **direct thread** for each approved user
- a **task thread** for each task

A direct thread is the user’s long-lived conversation. A task thread is scoped work with its own history and workspace.

Direct threads are created lazily on the first real user message. Task threads, task session files, and task workspaces are created lazily on the first actual run.

### Thread storage

```text
~/skarbot/state/threads/
  <user-id>.jsonl
  <user-id>.meta.json
  <user-id>.attachments/
  task-<task-slug>.jsonl
  task-<task-slug>.attachments/
```

Each thread uses one append-only `.jsonl` session file. History is never rewritten. Compaction adds summary boundaries to history; it does not replace history.

Thread files follow the pi session format and message model rather than a custom Skarbot transcript format.

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

Each thread has a sibling attachment directory. Files keep the original filename visibly, but get a timestamp prefix for uniqueness, for example:

```text
1732531234567_invoice.pdf
```

Attachments are saved locally at ingress and converted into normal substrate-compatible user content.

## Workspaces

Each thread owns one workspace:

```text
~/skarbot/workspaces/
  users/<user-id>/
  tasks/<task-slug>/
```

Every workspace contains a local `MEMORY.md`. Workspaces are thread-local. A user’s direct-thread workspace is not the same thing as that user’s capability directory.

Direct-thread workspaces are persistent by default. Task workspaces follow task lifecycle and can be marked for deterministic cleanup.

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

The task filename is the canonical task identifier.

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

Notification values are:

- `[]` — keep output in the task thread only
- `["direct-thread"]`
- `["sms"]`
- `["direct-thread", "sms"]`

Notifications always target the task owner.

Schedule forms are:

- omitted — start now in a task thread
- `{ "at": "<ISO timestamp>" }` — run once later
- `{ "cron": "<expr>", "timezone": "<IANA zone>" }` — recurring schedule

Recurring schedules reuse one bound task thread. One-off schedules create throwaway task threads.

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

A tool lives in `tools/<name>/` and is a local package with `package.json` and at least one extension entry in `pi.extensions`. One tool package may register multiple named tools.

Candidate capability promotion replaces the active same-named capability and removes the candidate. Promotion is handled through the runtime approval flow defined in [Runtime](runtime.md).

## Approvals and config

Approvals are stored under:

```text
~/skarbot/state/approvals/
```

Deployment config lives under:

```text
~/skarbot/state/config/
```

That config includes the admin record and the generated runtime configuration.

## See also

- [Product](product.md)
- [Runtime](runtime.md)
- [Deployment](deployment.md)
