# Runtime Platform and Setup

Status: Research

## Runtime Baseline

- V1 targets one Linux VM on `exe.dev`
- `vm-dotfiles` is the standard prerequisite bootstrap for that VM
- Local macOS or WSL support is useful but not required for first release
- The expected iteration loop is primarily on-box

## VM Roots

- `~/skarbot/core` for repo code
- `~/.skarbot/auth.json` for host-managed deployment-wide model auth
- `~/skarbot/core/capabilities/skills/` for repo-backed system skills
- `~/skarbot/core/capabilities/tools/` for repo-backed system tools
- `~/skarbot/state` for canonical state
- `~/skarbot/workspaces` for non-canonical execution space and artifacts
- `~/skarbot/state/capabilities/` for persistent promoted and candidate user capabilities
- `~/skarbot/workspaces/` for per-thread execution workspaces
- `~/skarbot/workspaces/users/` for per-user direct-thread workspaces
- `~/skarbot/workspaces/tasks/` for per-task workspaces

## Post-Bootstrap Setup Ownership

- `skarbot-core` setup begins after `vm-dotfiles` has prepared the VM
- Setup should support an interactive first-run flow that writes Skarbot config to disk
- Setup should also support a non-interactive path driven by config files and environment variables for reruns and automation
- Setup should create and validate `~/skarbot/state` and `~/skarbot/workspaces`
- Setup should create and validate the capability roots under `~/skarbot/state/capabilities/`
- Setup should create and validate `~/skarbot/workspaces/users/` and `~/skarbot/workspaces/tasks/`
- Setup should validate the repo-backed system capability roots under `~/skarbot/core/capabilities/` rather than creating them dynamically
- Setup should install repo-local dependencies, including `pi-agent` as the substrate for the Skarbot agent runtime
- Setup should write Skarbot-specific config and templates
- Setup should wire Skarbot to the expected `exe.dev` inbox and proxy environment
- Setup should validate that required `exe.dev` proxy or integration endpoints exist for services like ClickSend
- Setup should create the long-running launch/runtime mechanism and scheduled-job wiring needed by Skarbot
- Setup should run health checks and print the relevant local or private status endpoints for the newly started agent
- Setup should register the initial admin user by collecting the admin name, email, and phone number
- Setup should create the initial admin user record directly under `~/skarbot/state/users/active/`
- Setup should derive the initial admin `id` once and store it explicitly
- Setup should write a protected `~/skarbot/state/config/admin.json` file containing the one allowed admin user id
- Setup should support the web app running behind an `exe.dev` proxy that may be public while protected routes still require `exe.dev` login
- Setup should guide the admin through establishing the deployment-wide OpenAI Codex subscription auth on the VM
- Setup should treat `~/.skarbot/auth.json` as the canonical deployment-wide model-auth path
- Setup should keep deployment model auth outside portable `~/skarbot/state`

## User and Membership State

- Membership should be managed by Skarbot in runtime state rather than by repo config
- Pending onboarding requests should be stored as files under `~/skarbot/state/users/pending/`
- Approved users should live under `~/skarbot/state/users/active/`
- Denied onboarding requests should move to `~/skarbot/state/users/denied/`
- Admin notifications about new membership requests may be sent through outbound email from the `exe.dev` VM
- Membership approval in v1 may be handled by moving user files between these directories rather than through a dedicated app workflow
- Approval should have no automatic success-notification side effect in v1

## User Capabilities

- The runtime should support system skills and system tools in addition to per-user capabilities
- Repo-backed system capabilities should live under `~/skarbot/core/capabilities/skills/` and `~/skarbot/core/capabilities/tools/`
- Setup should create and validate `~/skarbot/state/capabilities/users/` and `~/skarbot/state/capabilities/candidates/users/`
- Per-user promoted capabilities should be stored separately from thread scratch workspaces
- Per-user promoted capabilities should be split into `skills/` and `tools/`
- Skill capabilities should use `SKILL.md` and optional helper files
- Tool capabilities should use local `pi` package directories with `package.json` and at least one extension entry declared under `pi.extensions`
- Tool capabilities should follow the same entrypoint-driven layout rules as `pi` extensions and packages rather than requiring one fixed internal source tree
- One `tools/<name>/` package may register multiple named tools in v1
- The `tools/<name>/` directory should be treated as a capability bundle identifier rather than as a 1:1 tool-name binding
- Tool capabilities should register their tools through the `pi` extension API rather than through a Skarbot-specific executable manifest
- Tool packages may include source files, helper scripts, bundled binaries, and local package dependencies beside their extension entrypoints
- Tool packages may call approved HTTP proxies or spawn colocated or system executables from their extension code when needed
- Runtime loading behavior for system, active user, and candidate capabilities is defined centrally in `docs/runtime/execution-profiles.md`
- Promoted capabilities may surface as first-class named tools in the host runtime rather than only as prompt text
- If a user capability name collides with a system capability name, the user capability should take precedence in v1
- Candidate capabilities should stay in the separate candidates root until explicitly approved
- Candidate capabilities should be validated deterministically during promotion rather than by a separate manual review step
- Approving a candidate should replace the active capability and delete the candidate
- Discarding a candidate should delete the candidate
- Capability cleanup should happen inline with capability actions rather than through a background cron path in v1
- Capability availability should not depend on mid-turn hot-swapping of a long-lived in-memory session; newly approved capabilities should be visible from the next run or turn onward
- Direct-thread versus task-thread capability loading behavior should not be redefined here; see `docs/runtime/execution-profiles.md`
- The shared and task-thread standard tool surfaces are defined centrally in `docs/runtime/tools.md`
- The grouped `capabilities` tool should support `list`, `inspect`, `get`, and `promote`
- `capabilities(action = "get")` should seed a task workspace from candidate first, then active
- `capabilities(action = "promote")` should run deterministic validation internally and fail fast back to the task when invalid
- On successful `promote`, the runtime should replace the same-named candidate bundle and send a review request into the owner's main direct thread
- `plan(...)` should be thread-local state persisted with thread or session state rather than a separate runtime file
- `subagents(...)` should create generic worker subagents only in v1
- Spawned subagents should use model-chosen cute Malayalee names as their only active handles
- Duplicate active subagent names in the same thread should be rejected

## Workspaces

- A workspace is the filesystem working area for a thread
- In v1, thread, workspace, and execution profile should stay aligned
- Workspace scratch and promoted capability storage should be separate concerns
- Each direct thread should own one workspace under `~/skarbot/workspaces/users/<user-id>/`
- Each task thread should own one workspace under `~/skarbot/workspaces/tasks/<task-slug>/`
- Direct-thread workspaces should be created lazily on first actual direct-thread action
- Task workspaces should be created lazily on first run
- `SessionHeader.cwd` should serialize that thread workspace path into the `pi` session file
- Workspace lifecycle should follow thread lifecycle rather than using one shared global workspace
- Direct-thread workspaces should persist by default in v1
- Direct-thread workspaces may be cleaned up explicitly on request, but v1 should not define a formal pruning workflow for them
- Task workspace cleanup should be performed by a deterministic local cleanup command
- That cleanup command should delete a task workspace only when the matching task file is inactive and the workspace root contains an empty `.deleteworkspace` sentinel file
- Failed task workspaces should be retained because they should not carry the deletion marker
- Setup should install a startup cleanup run plus a system cron entry that invokes the local cleanup command every 30 minutes
- The cleanup command should stay zero-token and should exit immediately when no deletion markers exist
- Capability edits should happen in task workspaces seeded from an existing candidate first, then the active capability, rather than by editing active capability directories in place

## Config

- Generated instance config should live under `~/skarbot/state/config/` by default
- The main generated config file should use `TOML`
- V1 should use one main generated `TOML` file rather than splitting generated config across multiple files
- The repo should hold only example templates and checked-in defaults, not live instance config
- Environment variables should be used for temporary overrides and automation, not as the primary configuration system
- The deployment default model should be configured in runtime config rather than inferred from whichever thread changed model most recently
