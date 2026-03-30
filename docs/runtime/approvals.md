# Runtime Approvals

Status: Research

## Purpose

- This doc is the source of truth for Skarbot approvals in v1
- Skarbot should use one shared approval system with two approver classes:
  - `user`
  - `admin`
- Different approval cases should not invent different reply grammars or storage models

## Approval Classes

- `user` approvals should be the common path in v1
- `user` approvals should be delivered into the approver's long-lived direct thread
- `admin` approvals should be rare and operational
- `admin` approvals should be delivered to the admin through the `exe.dev` email path in v1

## Shared Approval Record

- All approvals should use one file-backed approval record under `~/skarbot/state/approvals/`
- Each approval should include at least:
  - `id`
  - `approver_type`
  - `approver_id`
  - `kind`
  - `summary`
  - `details`
  - `status`
  - `source_thread`
  - `source_task` (optional)
- `status` should be one of:
  - `pending`
  - `approved`
  - `denied`
  - `expired`
- Approval handling should be deterministic runtime behavior rather than model judgment

## Canonical Reply Grammar

- Every approval should have a short stable id such as `A14`
- Every approval message should include the exact accepted actions
- The canonical approval actions in v1 should be:
  - `approve <id>`
  - `deny <id>`
  - `feedback <id>: <text>`
- Web UI buttons or other richer surfaces may exist later, but they should map to the same actions
- V1 should not rely on fuzzy free-form replies such as `@approved`

## User Approval Flow

- User approvals should be posted into the owner's direct thread
- The approval message should include:
  - a short summary
  - the source task or thread
  - any relevant link back to the task thread
  - the approval id
  - the canonical reply actions
- When the user replies with `approve <id>`, the approval should resolve as approved
- When the user replies with `deny <id>`, the approval should resolve as denied
- When the user replies with `feedback <id>: <text>`, the approval should remain pending and the feedback should be written back into the source task or thread

## Admin Approval Flow

- Admin approvals should be sent to the admin through the `exe.dev` email path in v1
- Admin approvals originating from user-owned work should require explicit user consent before the approval is created
- The agent should not be allowed to open an admin approval directly from a task without the user's consent
- The email should include:
  - a short summary
  - the approval id
  - the canonical reply actions
  - enough context to understand the requested action
- The admin should resolve the approval by replying with:
  - `approve <id>`
  - `deny <id>`
  - `feedback <id>: <text>`
- V1 should treat email as the primary admin approval surface rather than requiring a dedicated web approval queue

## Escalation From User Work

- If a task encounters an action that requires admin approval, runtime should reject the action deterministically with an `admin approval required` result
- The task may then ask the user in the user's direct thread whether they want to request admin approval
- If the user declines, no admin approval should be created
- If the user agrees, Skarbot should create the admin approval and send it through the admin email path
- This keeps the user as the principal for user-owned work even when an operator decision is required

## Waiting Behavior

- A task blocked on an approval should pause rather than keep retrying or improvising around the blocked action
- While paused, the task should remain tied to the pending approval id
- When the approval resolves:
  - `approve` should resume the task with an approved result
  - `deny` should resume the task with a denied result
  - `feedback` should resume the task with the feedback content
  - `expired` should resume the task as an expired or denied-equivalent outcome
- Approval resolution should flow back into the source task or thread deterministically

## Typical Ownership Split

- User approvals should cover actions that are mainly about that user's workflow or data
- Examples:
  - capability promotion
  - schedule creation, edits, and deletion
  - other user-facing behavior changes that need confirmation
- Admin approvals should cover deployment-wide or operator-sensitive changes
- Examples:
  - `exe.dev` integration or sharing changes
  - new outbound destinations such as SMS recipients
  - host package installs
  - destructive actions outside owned roots
  - deployment-wide runtime or host changes

## Relationship To Other Docs

- Execution context is defined in `docs/runtime/execution-profiles.md`
- Autonomy and approval boundaries are summarized in `docs/runtime/autonomy.md`
- Capability promotion is a task-thread workflow that uses this approval system rather than defining its own separate approval syntax
