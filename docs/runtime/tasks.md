# Runtime Tasks

Status: Research

## Task Model

- Tasks should use the simplest viable runtime model in v1
- Active tasks should live as one JSON file under `~/skarbot/state/tasks/`
- Inactive tasks should move to `~/skarbot/state/tasks/inactive/`
- Failed tasks should move to `~/skarbot/state/tasks/failed/`
- Skarbot should not use a task `status` field in v1; folder placement should define lifecycle instead
- If Skarbot does not have enough information to create a task, it should ask follow-up questions and should not create the file yet
- A task should not be created until its notification list is defined
- The task file should be editable over SSH if desired
- Ordinary tasks should not go through a human approval step before activation
- The scheduler should automatically pick up task files only from `~/skarbot/state/tasks/`
- Each task should link to exactly one task thread in v1
- Every task should run under the `task-thread` execution profile described in `docs/runtime/execution-profiles.md`
- In v1, task notifications should always target the task owner
- Tasks with no `schedule` should start immediately in their own task thread rather than running in the direct thread
- User-owned schedules should be managed from the owner's direct thread through `schedule(...)`, even when the underlying execution is represented by a task file
- Recurring schedules should bind to one dedicated reusable task thread
- One-off schedules should create one-off throwaway task threads
- Capability-building tasks should edit working copies inside their task workspace rather than modifying active capabilities in place
- When the target capability is a tool, the working copy should stay a local `pi` package or extension rather than a custom executable-manifest bundle
- If a capability-building task targets an existing capability, its task workspace should be initialized from the candidate capability when present, otherwise from the active capability
- If a capability-building task targets a brand-new capability, its task workspace should start empty
- A capability-building task should be able to submit its result into the candidate capabilities root for later approval
- Capability-building tasks may adapt existing `pi` extensions, `pi` packages, or OpenClaw tool plugins into Skarbot-owned candidate capabilities during that workspace edit flow
- Candidate preview behavior is defined centrally in `docs/runtime/execution-profiles.md`
- Approval routing and reply syntax are defined centrally in `docs/runtime/approvals.md`
- If a task hits an action that requires admin approval, it should not be allowed to create that admin approval directly
- Instead, the task should ask the owner whether they want to request admin approval
- If the owner agrees, Skarbot should create the admin approval through the admin email path and pause the task until that approval resolves
- All task-thread runs should receive the task-thread system tool surface defined in `docs/runtime/tools.md`
- The grouped `capabilities` tool should support exactly these v1 actions:
  - `list`
  - `inspect`
  - `get`
  - `promote`
- `plan(...)` should be thread-local rather than task-file-backed
- In v1, each task thread should have at most one active plan
- `plan(create, steps)` should replace the current active plan for that task thread
- `plan(update, id, status)` should update one step in the current active plan
- Once all steps are `completed`, the task thread's active plan should be considered complete and may be removed from active state
- `subagents(...)` should manage generic worker subagents scoped to the current task thread
- `subagents(spawn, ...)` should create one generic worker rather than selecting among different subagent personas in v1
- Spawned subagents should use model-chosen cute Malayalee names as their only active handles
- Duplicate active subagent names in the same task thread should be rejected
- Task-thread subagents should be capped per thread in v1 so one task cannot spawn unbounded background workers
- `capabilities(action = "get")` should copy an editable working bundle into the task workspace from candidate first, then active, and fail if neither exists
- `capabilities(action = "promote")` should run deterministic validation internally before staging the candidate
- If that internal validation fails, `promote` should return diagnostics directly to the task and should not contact the owner
- If that internal validation succeeds, `promote` should replace the same-named candidate bundle and send a user approval request into the owner's main direct thread with a link back to the task thread
- Scheduled execution may still use the task file's stored `schedule` field under the hood:
  - one-off later: `{ "at": "<ISO timestamp>" }`
  - recurring: `{ "cron": "<expr>", "timezone": "<IANA zone>" }`
- Task thread session files should be created lazily on first actual run rather than when the task file is created
- Task workspaces should be created lazily on first run rather than when the task file is created
- After a one-off task succeeds, it should move to `~/skarbot/state/tasks/inactive/`
- If a task fails, it should move to `~/skarbot/state/tasks/failed/`
- Disabling a recurring task should move it to `~/skarbot/state/tasks/inactive/`
- A successful task that should discard its workspace should move its task file to `~/skarbot/state/tasks/inactive/` and write an empty `.deleteworkspace` file at the workspace root
- Failed task workspaces should be retained when their task moves to `~/skarbot/state/tasks/failed/`
- Task workspace cleanup should be driven by a deterministic local cleanup path rather than by the LLM
- Task workspace deletion should require both an inactive task file and a `.deleteworkspace` marker at the workspace root
- The cleanup path should run once on startup and then from system cron every 30 minutes
- The cleanup path should stay zero-token in v1 and should exit immediately when no `.deleteworkspace` marker exists
- Routine outputs from scheduled tasks should live in the associated task thread history rather than in `MEMORY.md`

## Minimum Task Fields

- `title`
- `instructions`
- `schedule` (optional)
- `owner`
- `notification`

In v1, the task filename should be the canonical task identifier rather than a separate `id` field inside the JSON.
Task filenames should be human-readable slugs derived from the task title, such as `morning-briefing.json`.
Once created, a task filename should stay stable even if the task title changes later.
If a slug collision occurs within the destination directory, Skarbot should append `-2`, `-3`, and so on to the filename.
The task JSON should keep an explicit `title` field as the human-facing label even though the filename is canonical.
The task JSON should keep `instructions` as one plain string in v1 rather than a richer structured field.
The task JSON should not include explicit `created_at` or `updated_at` fields in v1; filesystem metadata and thread or run history should be sufficient.

In v1, `owner` should remain explicit in the task JSON rather than being derived from task-thread metadata.
In v1, `owner` should store a stable internal Skarbot user id rather than an email address or phone number.

In v1, `notification` should be a list:
- `[]` means the task keeps results only in its own task thread
- supported entries are `direct-thread` and `sms`
- a task may use either one or both supported entries
- `direct-thread` means delivery into the owner's long-lived direct thread
- `sms` means delivery to the owner's linked and allowlisted SMS destination

In v1, `schedule` should behave like this:
- omitted: start now in a task thread
- `{ "at": "<ISO timestamp>" }`: run once later
- `{ "cron": "<expr>", "timezone": "<IANA zone>" }`: recurring schedule
- The timezone for recurring schedules should default from the owner's user record when the user does not explicitly override it

## Runtime Skill

- The built-in runtime skill name should be `tasks`
- The `tasks` skill should teach the JSON shape, the `~/skarbot/state/tasks/` path, and how task files are created, inspected, and edited
- The `tasks` skill should point capability and tool-surface behavior back to `docs/runtime/tools.md` and `docs/runtime/execution-profiles.md` rather than restating those contracts independently
- Updates to existing tasks should happen by editing the task file rather than through a separate task action API
- The `tasks` skill should treat the task workspace `MEMORY.md` as the durable note file for that task thread
- The `tasks` skill should treat task thread history as the default home for routine task output
- The `tasks` skill should create a task thread for scoped work that should not live in a user's direct thread
- The `tasks` skill should treat user-owned schedules as a direct-thread concern even if scheduled execution is persisted through task files under the hood
- The `tasks` skill should expose a completion action that can move a task to `inactive/` and mark its workspace for deletion
- Capability-building tasks should also have access to the repo-backed `skills` and `tools` system skills for designing user capabilities
