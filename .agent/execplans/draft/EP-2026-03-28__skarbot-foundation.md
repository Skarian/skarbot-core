# Skarbot Foundation and Project Definition

This ExecPlan is a living document. The sections `Progress`, `Surprises & Discoveries`, `Decision Log`, and `Outcomes & Retrospective` must be kept up to date as work proceeds.

This document must be maintained in accordance with `.agent/PLANS.md`.

- Plan ID: EP-2026-03-28__skarbot-foundation
- Status: DRAFT
- Created: 2026-03-28
- Last Updated: 2026-03-28
- Owner: UNCONFIRMED

## Purpose / Big Picture

Set up `skarbot-core` as the research and planning home for a personalized claw-like assistant built on `pi.dev`. After this milestone, the repo should contain pinned upstream references, a current focused docs set rooted at `docs/overview.md`, and an execution plan that can drive the next implementation steps without relying on chat history.

## Progress

- [x] (2026-03-28 14:18Z) Collected first-pass web research on `pi.dev`, `openclaw`, `nanoclaw`, `nanobot`, `NemoClaw`, and `oh-my-pi`.
- [x] (2026-03-28 14:18Z) Added the agreed reference repositories under `references/` and pinned each to the imported `HEAD`.
- [x] (2026-03-28 14:18Z) Created initial docs scaffolding and a project-definition stub for iterative Q&A.
- [x] (2026-03-28 14:29Z) Recorded the desired v1 surfaces: terminal, web UI, email, SMS via ClickSend, and scheduled jobs / cron jobs.
- [x] (2026-03-28 14:38Z) Recorded the inbound-first architecture and unified adapter-core direction for first install.
- [x] (2026-03-28 14:44Z) Narrowed email for v1 to inbound-only via the `exe.dev` VM inbox.
- [x] (2026-03-28 14:44Z) Recorded the runtime baseline as one Linux VM on `exe.dev`, with local development optional for v1.
- [x] (2026-03-28 15:13Z) Recorded the dedicated-VM autonomy model, including PR workflow, private approvals, SMS escalation, and configurable owned roots.
- [x] (2026-03-28 15:13Z) Recorded the minimum v1 web UI scope and the decision to represent tasks as threads.
- [x] (2026-03-28 15:24Z) Recorded per-user inboxes, onboarding for identity linking, and multi-user workflow-thread support.
- [x] (2026-03-28 16:03Z) Recorded the external-state memory model: deterministic policy outside memory, plain directory shards for users and workflows, and `MEMORY.md` plus append-only history as the preferred v1 shape.
- [x] (2026-03-28 16:08Z) Recorded `~/skarbot-state` as the default external VM state root.
- [x] (2026-03-28 16:15Z) Recorded that v1 should skip a standalone heartbeat concept and keep recurring work attached to workflows and explicit scheduling.
- [x] (2026-03-28 16:22Z) Recorded a file-first runtime state model with no DB source of truth, plus the inbox/workflow consolidation triggers.
- [x] (2026-03-28 16:29Z) Recorded that no additional day-one integrations are required beyond ClickSend, the `exe.dev` inbox, and `exe.dev` GitHub access.
- [x] (2026-03-28) Recorded the three-root VM layout: `~/skarbot-core` for repo code, `~/skarbot-state` for canonical state, and `~/skarbot-data` for execution space and artifacts.
- [x] (2026-03-28) Recorded that v1 should not add extra sandboxing inside the dedicated VM beyond the approval model and owned-root boundaries.
- [x] (2026-03-28) Recorded `vm-dotfiles` as the standard `exe.dev` VM prerequisite and reframed `skarbot-core` as the post-bootstrap setup layer.
- [x] (2026-03-28) Recorded the Skarbot-specific post-bootstrap setup contract, including `pi-agent`, runtime wiring, health checks, and initial admin registration.
- [x] (2026-03-28) Recorded the setup-config capture model: interactive first run, plus config-file and env-driven non-interactive reruns.
- [x] (2026-03-28) Recorded `~/skarbot-state/config/` as the default generated instance-config location.
- [x] (2026-03-28) Recorded `TOML` as the main generated config format, with env vars reserved for secrets and temporary overrides.
- [x] (2026-03-28) Recorded the split secrets policy: no generated secret files, ClickSend through `exe.dev` proxy/integration endpoints, and model providers allowed to use local CLI or subscription-backed auth outside repo config.
- [x] (2026-03-28) Recorded that v1 should use one main generated `TOML` file rather than a split config-file set.
- [x] (2026-03-28) Deferred exact top-level `TOML` sections until later in the docs pass and shifted the next question to concrete workflows.
- [x] (2026-03-28) Recorded that v1 should scaffold recurring workflows from natural-language user requests, with a morning briefing as the anchor example.
- [x] (2026-03-28) Recorded that newly scaffolded recurring tasks should start as draft task records in runtime state, with no live schedule created until approval.
- [x] (2026-03-28) Reclassified this ExecPlan from `ACTIVE` to `DRAFT` while work remains in docs and research mode.
- [x] (2026-03-28) Recorded the hybrid recurring-task approval flow: file-backed drafts, admin-only web approval, optional SSH edits, and immediate activation on approve.
- [x] (2026-03-28) Simplified recurring tasks to one JSON file per task under `~/skarbot-state/tasks/`, then later moved lifecycle into filesystem placement instead of a status field.
- [x] (2026-03-28) Removed recurring-task approvals from the normal flow and simplified the runtime interface to one `tasks` skill plus direct task-file edits.
- [x] (2026-03-28) Replaced the catch-all `docs/project-definition.md` with focused docs rooted at `docs/overview.md`.
- [x] (2026-03-28T20:43Z) Simplified the memory model so the repo baseline is not treated as memory, deployment and user memory stay file-backed under `~/skarbot-state`, and task-specific context stays attached to each task JSON file.
- [x] (2026-03-28) Defined task JSON `memory` as a dedicated lessons section for durable improvements rather than routine outputs or run logs.
- [x] (2026-03-28) Recorded that routine recurring-task outputs belong in workflow thread history rather than in task `memory`.
- [x] (2026-03-28) Recorded that each recurring task owns exactly one workflow thread in v1 and that the web UI must distinguish task threads from user inbox threads.
- [x] (2026-03-28T21:09Z) Simplified threads further to one long-lived direct thread per human user and one task-linked thread per recurring task, with UI grouping derived from the task link.
- [x] (2026-03-28T21:24Z) Recorded that direct replies stay on the initiating surface and recurring tasks require an explicit notification method instead of inferring one.
- [x] (2026-03-28T21:31Z) Limited v1 recurring-task notification methods to `none`, `direct-thread`, and `sms`.
- [x] (2026-03-28T21:39Z) Reworked recurring-task notifications into a list field where `[]` means no external delivery and supported entries are `direct-thread` and `sms`.
- [x] (2026-03-28T21:44Z) Recorded that recurring-task notifications in v1 always target the task owner.
- [x] (2026-03-28T22:02Z) Broadened task threads to cover any scoped work in v1, with omitted `schedule` meaning start now in a task thread, `schedule.at` for one-off later work, and `schedule.cron` plus `timezone` for recurring work.
- [x] (2026-03-28T22:11Z) Replaced the task `status` field with folder-based lifecycle under `tasks/`, `tasks/inactive/`, and `tasks/failed/`.
- [x] (2026-03-28T22:18Z) Removed the separate task `channel` field from the v1 schema.
- [x] (2026-03-28) Kept the task `owner` field explicit in the v1 schema instead of deriving ownership from task-thread metadata.
- [x] (2026-03-28) Removed the separate task `id` field from the v1 schema and made the filename the canonical task identifier.
- [x] (2026-03-28) Chose human-readable slug filenames derived from the task title for v1 task files.
- [x] (2026-03-28) Kept task filenames stable after creation even if the task title changes later.
- [x] (2026-03-28) Resolved rare slug filename collisions with numeric suffixes such as `-2` and `-3`.
- [x] (2026-03-28) Kept an explicit task `title` field in v1 as the human-facing label.
- [x] (2026-03-28) Kept task `instructions` as one plain string in v1 instead of a richer structured field.
- [x] (2026-03-28) Kept task `memory` as a simple list of short durable lesson strings in v1.
- [x] (2026-03-28) Omitted explicit `created_at` and `updated_at` fields from the v1 task schema.
- [x] (2026-03-28) Narrowed task `owner` to a stable internal Skarbot user id rather than direct contact details.
- [x] (2026-03-28) Defined file-backed membership, setup-created initial admin identity, and `exe.dev` login-gated onboarding as the v1 user model.
- [ ] Run a one-question-at-a-time discovery process to fill the focused docs set with confirmed scope and constraints.
- [ ] Convert the confirmed definition into concrete bootstrap, runtime, and feature milestones for implementation.

## Surprises & Discoveries

- Observation: `oh-my-pi` was not obvious from generic web search results but the canonical repo is `can1357/oh-my-pi`, a substantial fork of `pi-mono`.
  Evidence: imported repo at `references/oh-my-pi` and reviewed its `README.md`.
- Observation: `NemoClaw` is useful mainly as a security/deployment reference because it is explicitly alpha and layered around OpenShell/OpenClaw.
  Evidence: `references/nemoclaw/README.md` marks the project as early preview and non-production-ready.
- Observation: the smallest claw variants are the best fit for your stated goals because both `nanoclaw` and `nanobot` frame small code size as a design principle rather than just an optimization.
  Evidence: `references/nanoclaw/README.md` and `references/nanobot/README.md`.
- Observation: `nanoclaw`'s registry-based channel model is a strong fit for `skarbot-core` because it keeps surfaces as thin adapters over one core.
  Evidence: `references/nanoclaw/src/channels/registry.ts`, `references/nanoclaw/src/types.ts`, `references/nanoclaw/src/router.ts`, `references/nanoclaw/docs/SPEC.md`.
- Observation: ClickSend appears suitable for inbound SMS webhooks, but not as the primary answer for inbound email.
  Evidence: ClickSend docs for inbound SMS automations and send-email documentation.
- Observation: `exe.dev` covers several platform concerns directly, including HTTPS exposure, inbound email delivery, and token-free GitHub repo access, which reduces first-version infrastructure work.
  Evidence: `https://exe.dev/llms.txt`, `https://exe.dev/docs/proxy.md`, `https://exe.dev/docs/receive-email.md`, `https://exe.dev/docs/integrations-github.md`.
- Observation: The dedicated-VM deployment model allows a much looser autonomy boundary than typical claw assistants, so the approval system can focus on external side effects and platform controls instead of routine dev work.
  Evidence: user direction for package installs, process changes, and PR-driven repo iteration.
- Observation: The web UI does not need a separate task product if scheduled work can be rendered as thread activity, which keeps the UI simpler.
  Evidence: user direction that tasks should just be specific threads.
- Observation: `exe.dev` auth provides a clean anchor for per-user inbox scoping, but inbound email/SMS still require explicit identity linking to users.
  Evidence: user direction to scope inbox views by authenticated user and to add onboarding for channel identity linking.
- Observation: the best-fit memory model is a split between deterministic policy, runtime state, and portable user/workflow shards stored outside git rather than one global assistant memory blob.
  Evidence: user direction on keeping policy out of memory and keeping private state out of the repo, plus local review of `references/nanobot/nanobot/agent/memory.py` and `references/pi-mono/packages/mom/src/context.ts`.
- Observation: heartbeat in the reference projects is mainly a periodic wake/checklist mechanism, not the same thing as memory consolidation, so it is safe to defer without blocking the shard design.
  Evidence: local review of `references/nanobot/README.md`, `references/openclaw/docs/gateway/heartbeat.md`, `references/openclaw/docs/automation/cron-vs-heartbeat.md`, and `references/oh-my-pi/docs/memory.md`.
- Observation: the user prefers file-first state over a database-backed control plane, which fits the lightweight target and still leaves room for a derived index later if the web UI needs one.
  Evidence: user direction approving no DB source of truth and local comparison with `nanoclaw`'s SQLite-heavy task/message state.
- Observation: the day-one external integration surface can stay intentionally narrow because the user does not require anything beyond the hosting platform features already in use plus ClickSend.
  Evidence: user direction that no extra day-one integrations are needed.
- Observation: the user's existing `vm-dotfiles` repo already owns the base VM bootstrap story, so `skarbot-core` should start after that prerequisite rather than duplicating generic machine setup.
  Evidence: `https://raw.githubusercontent.com/Skarian/vm-dotfiles/main/README.md`.
- Observation: the post-bootstrap setup needs to own both runtime bring-up and initial identity seeding because the first admin account anchors web auth, SMS allowlisting, and inbox routing from day one.
  Evidence: user direction that setup should collect admin email and phone number and use them for auth and access.
- Observation: setup needs both an ergonomic first-run UX and a repeatable automation path, so interactive prompting alone is insufficient even in a single-VM deployment.
  Evidence: user approval for interactive first run plus config-file and env-driven reruns.
- Observation: live instance config needs to stay outside git for privacy and portability, but still have a stable default location that setup and runtime can agree on.
  Evidence: user approval for `~/skarbot-state/config/` as the default generated config location.
- Observation: config needs a readable primary file format for hand-edits, while secrets should stay outside that main config surface.
  Evidence: user approval for `TOML` as the main format and prior approval to keep env vars for secrets and temporary overrides.
- Observation: `exe.dev` HTTP Proxy integrations provide a clean way to keep API secrets out of Skarbot config for service integrations like ClickSend, while model-provider auth can remain separate if it is already handled by local CLI/subscription auth.
  Evidence: `https://exe.dev/docs/integrations-http-proxy.md` and user direction on ClickSend versus model providers.
- Observation: a single generated config file keeps first-run setup and hand-editing simpler at this stage, and there is no clear pressure yet to split by subsystem.
  Evidence: user approval for one main generated `TOML` file in v1.
- Observation: `exe.dev` already provides enough identity and email primitives for first-pass onboarding, so a dedicated provider like Resend is unnecessary in v1.
  Evidence: `https://exe.dev/docs/all`, `https://exe.dev/docs/login-with-exe`, and user direction after review.
- Observation: the reference projects do not really provide a ready-made membership system; they mostly use stable route or session handles and allowlists, which supports keeping Skarbot membership small and file-backed.
  Evidence: local review of `references/pi-mono/packages/mom/src/store.ts`, `references/nanobot/SECURITY.md`, `references/openclaw/docs/automation/cron-jobs.md`, and `references/nanoclaw/docs/SPEC.md`.
- Observation: exact config schema is easier to design after the product behavior and first workflows are fully named, so deferring section layout reduces premature structure.
  Evidence: user direction to leave the top-level `TOML` sections until the end of docs creation.
- Observation: the user values workflow scaffolding more than a hardcoded starter list, which points the product toward "define recurring behavior from natural language" instead of shipping a fixed catalog first.
  Evidence: user direction that Skarbot should take requests like a morning briefing and scaffold recurring tasks as needed.
- Observation: the single-file `project-definition` doc became too broad, and the user wants focused docs with overview, providers, channels, web UI, and runtime sections instead.
  Evidence: user direction to split the docs into logical sections and drop `project-definition` in favor of `overview`.

## Decision Log

- Decision: Use `pi.dev` as the core runtime and keep `skarbot-core` personalized rather than general-purpose.
  Rationale: This matches the requested project shape and keeps scope aligned with a single-user assistant.
  Date/Author: 2026-03-28 / USER
- Decision: Import upstream references as git submodules under `references/`, pinned to latest `HEAD` at add time.
  Rationale: This preserves an inspectable research baseline while keeping upstream code read-only.
  Date/Author: 2026-03-28 / USER
- Decision: Track this work with a draft ExecPlan during the docs and research phase, and only promote it to active when implementation work is approved.
  Rationale: The repo still needs durable planning artifacts during the interview, but the current phase is definition work rather than active implementation.
  Date/Author: 2026-03-28 / USER
- Decision: The desired v1 surfaces are terminal, web UI, email, SMS via ClickSend, and scheduled jobs / cron jobs.
  Rationale: These are the interaction paths the user explicitly wants supported in the project definition.
  Date/Author: 2026-03-28 / USER
- Decision: First install must support inbound email and inbound SMS; outbound-only surfaces are not acceptable for phase 1.
  Rationale: Deferring inbound would distort the core architecture and likely force a redesign later.
  Date/Author: 2026-03-28 / USER
- Decision: Use a lightweight adapter-based core inspired by `nanoclaw`, with a thin web UI over the same routing/state layer.
  Rationale: This preserves the desired feature set while keeping the core small and understandable.
  Date/Author: 2026-03-28 / USER
- Decision: Keep ClickSend for SMS, but leave email as a separate provider decision.
  Rationale: Current ClickSend docs support the SMS use case more clearly than the email ingress use case.
  Date/Author: 2026-03-28 / USER
- Decision: Make email inbound-only for v1 and use the inbox already provided by the `exe.dev` VM.
  Rationale: This removes unnecessary outbound email scope while keeping inbound email available from the hosting environment.
  Date/Author: 2026-03-28 / USER
- Decision: Standardize the v1 deployment path on one Linux VM running on `exe.dev`.
  Rationale: The platform already provides inbox, proxy, SSH, and GitHub integration features that simplify the first release.
  Date/Author: 2026-03-28 / USER
- Decision: Local development is nice to have, but not a first-release requirement.
  Rationale: The user expects fast on-box iteration and may later have the agent iterate on itself in place.
  Date/Author: 2026-03-28 / USER
- Decision: Attached repos may be cloned and modified autonomously, but the user merges to `main`.
  Rationale: `exe.dev` repo access is already scoped in the platform, while final integration to `main` remains under user control.
  Date/Author: 2026-03-28 / USER
- Decision: Use a stable working branch per repo plus short-lived PR branches to avoid branch sprawl.
  Rationale: A single always-open branch would make PR review messy, while many long-lived branches would create noise.
  Date/Author: 2026-03-28 / USER
- Decision: Privileged approvals use a private page behind `exe.dev` auth, with SMS escalation after one hour instead of SMS approvals.
  Rationale: The web page carries context safely, while SMS remains a notification/escalation mechanism.
  Date/Author: 2026-03-28 / USER
- Decision: The dedicated VM is autonomous by default, with approval mainly reserved for `exe.dev` platform changes, new SMS recipients, and destructive actions outside configurable owned roots.
  Rationale: The VM exists for the agent, so the approval burden should stay low.
  Date/Author: 2026-03-28 / USER
- Decision: The v1 web UI is limited to a home page, approvals queue, chat-style thread viewer, and basic logs/status.
  Rationale: This covers the required operator surfaces without turning the web app into a broad control plane.
  Date/Author: 2026-03-28 / USER
- Decision: Represent tasks as threads rather than as a separate top-level product surface.
  Rationale: This keeps the mental model unified across conversations, scheduled work, and approvals.
  Date/Author: 2026-03-28 / USER
- Decision: Use one long-lived direct thread per human user in v1, regardless of whether messages arrive from terminal, web UI, SMS, or email.
  Rationale: This is the simplest workable direct-conversation model and matches the user's preference for a very small mental model.
  Date/Author: 2026-03-28 / USER
- Decision: Add an onboarding flow that links inbound email/SMS identities to authenticated users.
  Rationale: Web identities and channel identities are separate and need explicit binding for correct routing.
  Date/Author: 2026-03-28 / USER
- Decision: The user's wife should be supported as a first-class non-admin user who can have her own direct thread and create recurring-task threads in v1.
  Rationale: The project is personalized, but not strictly single-human.
  Date/Author: 2026-03-28 / USER
- Decision: Keep live Skarbot state outside the repo, with deterministic policy in code/config, deployment-wide memory, per-user memory, and task-linked context kept in normal state files.
  Rationale: This keeps private memory portable without committing it to git and keeps authority in deterministic code/config instead of model discretion.
  Date/Author: 2026-03-28 / USER
- Decision: Use `~/skarbot-state` as the default external state root on the VM.
  Rationale: It is explicit, easy to back up, separate from the repo, and simple for a single-user VM deployment.
  Date/Author: 2026-03-28 / USER
- Decision: Skip a standalone heartbeat concept in v1.
  Rationale: The user prefers to keep recurring checks tied to explicit workflows and schedules for now, and heartbeat is not required to make the memory model work.
  Date/Author: 2026-03-28 / USER
- Decision: Keep v1 runtime state file-first under `~/skarbot-state` rather than requiring a DB as the canonical store.
  Rationale: This matches the lightweight target, keeps the state portable and inspectable, and avoids unnecessary platform complexity in the first version.
  Date/Author: 2026-03-28 / USER
- Decision: Consolidate user memory once after 30 minutes idle per activity window, and refresh task-linked context after each succeeded run when something durable was learned.
  Rationale: Direct conversation memory should compact only after conversations settle, while recurring tasks should refresh durable state after each clean execution without treating recurring tasks as permanently complete.
  Date/Author: 2026-03-28 / USER
- Decision: Require no additional day-one integrations beyond ClickSend, the `exe.dev` inbox, and `exe.dev` GitHub access.
  Rationale: This keeps the first release narrow and aligned with the lightweight personalized scope.
  Date/Author: 2026-03-28 / USER
- Decision: Use three main VM roots: `~/skarbot-core` for repo code, `~/skarbot-state` for canonical state, and `~/skarbot-data` for task execution space and artifacts.
  Rationale: This keeps durable state separate from disposable work while avoiding a generic top-level `projects` bucket.
  Date/Author: 2026-03-28 / USER
- Decision: Add no extra sandboxing inside the dedicated VM in v1 beyond the current approval rules and owned-root boundaries.
  Rationale: The VM is dedicated to the agent, and the user prefers low approval overhead over an additional isolation layer in the first version.
  Date/Author: 2026-03-28 / USER
- Decision: Treat `vm-dotfiles` as the standard prerequisite bootstrap for the `exe.dev` VM and scope `skarbot-core` to the Skarbot-specific post-bootstrap setup and product behavior.
  Rationale: The base machine bootstrap already exists elsewhere, and duplicating it here would blur product scope and the docs story.
  Date/Author: 2026-03-28 / USER
- Decision: Make `skarbot-core` setup responsible for state/data root creation, repo-local dependency install including `pi-agent`, config writing, `exe.dev` environment wiring, runtime and scheduled-job start wiring, health checks, and initial admin registration by email and phone.
  Rationale: The user wants a direct "bootstrap VM, then run Skarbot setup" flow that leaves the agent usable and accessible immediately.
  Date/Author: 2026-03-28 / USER
- Decision: Capture initial config through interactive first-run prompts that write config to disk, while also supporting non-interactive config-file and env-driven reruns.
  Rationale: This preserves a smooth first install without giving up repeatability or automation.
  Date/Author: 2026-03-28 / USER
- Decision: Store generated instance config under `~/skarbot-state/config/` by default, with the repo containing only templates and checked-in defaults.
  Rationale: This keeps private instance-specific config out of git while preserving a predictable runtime location.
  Date/Author: 2026-03-28 / USER
- Decision: Use `TOML` as the main generated config format and reserve environment variables for secrets and temporary overrides.
  Rationale: This keeps the main config human-readable without turning env vars into the primary configuration system.
  Date/Author: 2026-03-28 / USER
- Decision: Do not store secrets in repo config or generated local secret files; for service integrations like ClickSend, prefer `exe.dev` proxy/integration endpoints, while allowing model providers to use local CLI or subscription-backed auth outside repo config.
  Rationale: This keeps Skarbot config secret-free without forcing every provider through the same access pattern.
  Date/Author: 2026-03-28 / USER
- Decision: Use one main generated `TOML` file in v1 rather than multiple generated config files.
  Rationale: A single file is simpler for setup, inspection, and initial documentation while the configuration surface is still small.
  Date/Author: 2026-03-28 / USER
- Decision: Defer the exact top-level `TOML` section layout until later in the docs process, after the product behavior and workflow definitions are clearer.
  Rationale: This avoids designing the config schema around guesses instead of the actual documented behavior.
  Date/Author: 2026-03-28 / USER
- Decision: Treat recurring-workflow scaffolding from natural-language requests as a first-class v1 capability rather than prioritizing a fixed initial workflow list.
  Rationale: The user wants Skarbot to turn requests like a stock-and-weather morning briefing into recurring workflows without having to predefine a rigid starter catalog.
  Date/Author: 2026-03-28 / USER
- Decision: Ordinary recurring tasks should not require human approval; if information is missing, the agent should ask follow-up questions and should not create the task file yet.
  Rationale: The user wants the smallest workable model and does not want a recurring-task approval flow for normal automation.
  Date/Author: 2026-03-28 / USER
- Decision: Use one JSON file per task under `~/skarbot-state/tasks/`, with lifecycle carried by filesystem placement rather than a task `status` field.
  Rationale: This keeps the runtime file-first and inspectable, while avoiding a richer state machine in the task schema itself.
  Date/Author: 2026-03-28 / USER
- Decision: Arm the agent with one `tasks` skill that defines the task JSON shape, task path, and how task files are created, inspected, and edited; handle later changes by editing task files instead of exposing a separate task action API.
  Rationale: This keeps the runtime surface small and lets one focused skill plus the file contract do most of the work.
  Date/Author: 2026-03-28 / USER
- Decision: In v1, active tasks live in `~/skarbot-state/tasks/`, inactive tasks live in `~/skarbot-state/tasks/inactive/`, and failed tasks live in `~/skarbot-state/tasks/failed/`.
  Rationale: This preserves visibility into completed, disabled, and failed work without overloading one folder or relying on deletion.
  Date/Author: 2026-03-28 / USER
- Decision: Keep the built-in runtime skill set very small in v1: `tasks` plus `memory`.
  Rationale: The user wants a lightweight runtime, and the reference review suggests `memory` is the only clear second foundational skill worth baking in now.
  Date/Author: 2026-03-28 / USER
- Decision: Replace the catch-all `project-definition` doc with focused docs rooted at `docs/overview.md`, plus separate docs for providers, channels, web UI, users and threads, and runtime topics.
  Rationale: The user wants crisper docs and explicitly prefers `overview` over `project-definition`.
  Date/Author: 2026-03-28 / USER
- Decision: The `memory` skill should write durable facts to shard `MEMORY.md`, search shard `HISTORY.ndjson`, honor explicit “remember this” requests, support lightweight recall, and prefer filesystem operations wherever practical.
  Rationale: This matches the approved defaults and keeps the memory model simple, inspectable, and file-backed.
  Date/Author: 2026-03-28 / USER
- Decision: Treat repo-owned instructions, skills, and product docs as the static Skarbot baseline rather than as memory; in v1, use deployment-wide memory, per-user memory, and task-specific context attached directly to each task JSON file.
  Rationale: This keeps the system simple, preserves the filesystem as the source of truth, and avoids inventing a heavier memory subsystem before it is needed.
  Date/Author: 2026-03-28 / USER
- Decision: Each task JSON file should include a dedicated `memory` section used for durable lessons and improvements, not for routine outputs or run-by-run logs.
  Rationale: The user wants the agent to learn from mistakes and improve recurring tasks over time without letting task files bloat into historical archives.
  Date/Author: 2026-03-28 / USER
- Decision: Routine recurring-task outputs should live in the linked task thread history rather than being stored in task `memory`.
  Rationale: This preserves task `memory` as a compact improvement layer and uses the existing thread model for ordinary output history.
  Date/Author: 2026-03-28 / USER
- Decision: Use one shared thread concept in v1: a thread linked to a recurring task is a task thread, and a thread without a task link is a direct human conversation thread.
  Rationale: This is simpler than maintaining separate formal thread types while still letting the UI distinguish direct conversation from automation activity.
  Date/Author: 2026-03-28 / USER
- Decision: Replies to direct interactions should stay on the initiating surface, while recurring tasks must declare an explicit notification list instead of inferring delivery from the originating surface.
  Rationale: This preserves the simplified one-thread model without turning every web or terminal interaction into an unwanted SMS or cross-surface notification, and it keeps recurring task delivery intentional.
  Date/Author: 2026-03-28 / USER
- Decision: In v1, recurring-task `notification` should be a list where `[]` means the task stays in its own thread, and supported entries are limited to `direct-thread` and `sms`.
  Rationale: This keeps recurring task delivery explicit and small while allowing a task to notify one or both of the user's main delivery targets without adding more routing complexity yet.
  Date/Author: 2026-03-28 / USER
- Decision: In v1, recurring-task notifications should always target the task owner.
  Rationale: This keeps the delivery model simple and avoids introducing arbitrary recipient routing into the first task schema.
  Date/Author: 2026-03-28 / USER
- Decision: In v1, tasks are the general unit of scoped work, not just recurring workflows; a task with no `schedule` starts immediately in its own task thread, `schedule.at` means one-off later execution, and `schedule.cron` plus `timezone` means recurring execution.
  Rationale: This keeps heavy or long-running work out of the direct thread without introducing a second filesystem concept, while still matching the simple shapes seen in the reference projects.
  Date/Author: 2026-03-28 / USER
- Decision: The task JSON does not need a separate `channel` field in v1.
  Rationale: Task ownership, task-thread linkage, and explicit notification lists already cover the needed routing information for the current design.
  Date/Author: 2026-03-28 / USER
- Decision: The task JSON should keep an explicit `owner` field in v1.
  Rationale: Ownership is not otherwise defined in the thread model yet, and explicit ownership keeps notification routing and task inspection self-contained.
  Date/Author: 2026-03-28 / USER
- Decision: The task filename should be the canonical identifier in v1 instead of a separate `id` field inside the JSON.
  Rationale: This is the simplest fit for the filesystem-first task model and keeps the task file shape smaller.
  Date/Author: 2026-03-28 / USER
- Decision: Task filenames in v1 should be human-readable slugs derived from the task title.
  Rationale: This preserves the filesystem-first model while keeping task files legible over SSH and in the web UI.
  Date/Author: 2026-03-28 / USER
- Decision: Once created, a task filename should stay stable even if the task title changes later.
  Rationale: Renaming is heavier when the filename is the canonical identifier, so stable filenames keep file-based tracking simpler.
  Date/Author: 2026-03-28 / USER
- Decision: If a slug filename collision occurs within the destination directory, Skarbot should append `-2`, `-3`, and so on.
  Rationale: This keeps the filesystem-first model simple while making collision handling deterministic; the user also expects collisions to stay rare because one-off tasks move out of the active directory.
  Date/Author: 2026-03-28 / USER
- Decision: The task JSON should keep an explicit `title` field in v1 as the human-facing label even though the filename is canonical.
  Rationale: Stable canonical filenames and editable human-facing labels serve different purposes, so keeping both avoids coupling display text to task identity.
  Date/Author: 2026-03-28 / USER
- Decision: The task JSON should keep `instructions` as one plain string in v1 instead of a richer structured field.
  Rationale: The reference projects all lean on one prompt/message/text field, and a single string keeps the first task schema simple.
  Date/Author: 2026-03-28 / USER
- Decision: The task `memory` section should be a simple list of short durable lesson strings in v1.
  Rationale: The reference projects keep durable memory lightweight and mostly text-oriented, so a simple list is the smallest task-local form that still captures lessons.
  Date/Author: 2026-03-28 / USER
- Decision: The task JSON should not include explicit `created_at` or `updated_at` fields in v1.
  Rationale: The references rely more on filesystem metadata, run history, or separate stores than on embedding timestamps in the task payload, so skipping them keeps the first schema smaller.
  Date/Author: 2026-03-28 / USER
- Decision: The task `owner` field should store a stable internal Skarbot user id rather than an email address or phone number.
  Rationale: The reference projects usually route work through stable internal handles rather than mutable contact data, and Skarbot now needs a distinct user-store concept for membership and identity linking.
  Date/Author: 2026-03-28 / USER
- Decision: Skarbot should manage membership itself using file-backed user records under `~/skarbot-state/users/active/`, `~/skarbot-state/users/pending/`, and `~/skarbot-state/users/denied/`.
  Rationale: This keeps membership aligned with the filesystem-first runtime model and avoids introducing a database or external member system before it is needed.
  Date/Author: 2026-03-28 / USER
- Decision: The main app may sit behind a public `exe.dev` proxy, but protected routes should force `exe.dev` login and then gate access through Skarbot membership records using `X-ExeDev-UserID` and `X-ExeDev-Email`.
  Rationale: This lets Skarbot own approval and membership without relying on `exe.dev` sharing as the primary member list, while still using `exe.dev` for identity and auth.
  Date/Author: 2026-03-28 / USER
- Decision: Initial setup should create the admin user record directly, while later member requests should go through onboarding plus admin approval and may notify the admin through outbound email sent from the `exe.dev` VM.
  Rationale: This avoids circular bootstrapping, keeps the initial install simple, and reuses `exe.dev`'s built-in owner-email path instead of adding another email provider.
  Date/Author: 2026-03-28 / USER

## Outcomes & Retrospective

The repo now has a usable research baseline and a draft planning path. The next gap is continuing the one-question-at-a-time docs interview from the new focused docs structure, with special attention on the new membership and identity model, not beginning implementation yet.

## Context and Orientation

The repository started nearly empty, with only `README.md`, `AGENTS.md`, and `.agent/` scaffolding. The relevant paths after this milestone are:

- `references/pi-mono`
- `references/openclaw`
- `references/nanoclaw`
- `references/nanobot`
- `references/nemoclaw`
- `references/oh-my-pi`
- `docs/overview.md`
- `docs/providers.md`
- `docs/channels.md`
- `docs/web-ui.md`
- `docs/users-and-threads.md`
- `docs/runtime/platform-and-setup.md`
- `docs/runtime/autonomy.md`
- `docs/runtime/tasks.md`
- `docs/runtime/memory.md`
- `docs/research/reference-projects.md`
- `.agent/CONTINUITY.md`

`pi-mono` is the upstream runtime foundation. `openclaw` is the main real-world `pi` integration reference. `nanoclaw` and `nanobot` are small-codebase counterexamples that help constrain scope. `nemoclaw` is a hardening reference. `oh-my-pi` is a `pi` fork with additional harness features that may be worth cherry-picking later.

## Plan of Work

Continue by interviewing the user one question at a time and promoting each confirmed answer into the focused docs set rooted at `docs/overview.md`. The next critical choice is the exact format of the stable internal Skarbot user id now that both task ownership and membership depend on it. Revisit the exact config schema later, after those behaviors are documented. Once that contract is stable and implementation is approved, promote this plan to active and use the reference projects and the `exe.dev` baseline to decide which capabilities belong in the first implementation milestone, which should remain optional, and which should be excluded.

## Concrete Steps

Working directory: repo root.

Commands already run:

```bash
mkdir -p references
git submodule add https://github.com/badlogic/pi-mono references/pi-mono
git submodule add https://github.com/openclaw/openclaw references/openclaw
git submodule add https://github.com/qwibitai/nanoclaw references/nanoclaw
git submodule add https://github.com/HKUDS/nanobot references/nanobot
git submodule add https://github.com/NVIDIA/NemoClaw references/nemoclaw
git submodule add https://github.com/can1357/oh-my-pi references/oh-my-pi
git submodule update --init --recursive references/pi-mono references/openclaw references/nanoclaw references/nanobot references/nemoclaw references/oh-my-pi
git submodule status
```

Expected verification transcript:

```text
<sha> references/pi-mono
<sha> references/openclaw
<sha> references/nanoclaw
<sha> references/nanobot
<sha> references/nemoclaw
<sha> references/oh-my-pi
```

## Validation and Acceptance

This milestone is complete when:

- `references/` contains the agreed upstream repositories as submodules.
- `git submodule status` shows fixed SHAs for each imported reference.
- `docs/overview.md`, the focused runtime and product docs, and `docs/research/reference-projects.md` exist and reflect the current known scope.
- `.agent/CONTINUITY.md` and `.agent/execplans/INDEX.md` reflect the current plan and state.

No build or lint step is required yet because this milestone only changes documentation and git metadata, not product source code.

## Idempotence and Recovery

Read-only research and docs updates are repeatable. Re-running the `git submodule add` commands is not idempotent after initial import; use `git submodule status` and `git submodule update --init --recursive` for safe repeats. If a submodule needs to be refreshed later, advance it intentionally and record the new SHA in both the research doc and continuity ledger.

## Artifacts and Notes

Pinned references at the time of import:

```text
576e5e1a2fbe1abbbad96b696f4058cffd8391ca references/pi-mono
23772bb785731126d723fe1d82cf662ba625ab6c references/openclaw
29839464bff39b7672c0a7504d9bb7649a79c5f1 references/nanoclaw
c8c520cc9a4dbe619eb3f21200dc40971a36b665 references/nanobot
eb4ba8c486ad97ea3ca67bea388690b641dc134e references/nemoclaw
82b57365d421d05237149a70472444a455bd6d88 references/oh-my-pi
```

## Interfaces and Dependencies

Runtime dependency decisions are still open. For now, the repo depends only on git submodules for research:

- `badlogic/pi-mono`
- `openclaw/openclaw`
- `qwibitai/nanoclaw`
- `HKUDS/nanobot`
- `NVIDIA/NemoClaw`
- `can1357/oh-my-pi`

No production runtime libraries have been selected for `skarbot-core` yet.

## Plan Revision Notes (bottom-of-file change notes)

- (2026-03-28) Created the active foundation plan after importing the first research references and seeding the docs baseline.
- (2026-03-28) Updated the plan after confirming the desired v1 surfaces and shifting the next question to minimum first-install scope.
- (2026-03-28) Updated the plan after confirming inbound-first architecture, the unified adapter-core direction, and the need for a separate email integration choice.
- (2026-03-28) Updated the plan after narrowing email to inbound-only via the `exe.dev` VM inbox and shifting the next question to runtime baseline.
- (2026-03-28) Updated the plan after standardizing on a single `exe.dev` Linux VM for v1 and adding platform research notes.
- (2026-03-28) Updated the plan after confirming the dedicated-VM autonomy model and shifting the next question to the minimum v1 web UI surface.
- (2026-03-28) Updated the plan after defining the minimum v1 web UI surface and collapsing tasks into the thread model.
- (2026-03-28) Updated the plan after defining per-user inboxes, onboarding for identity linking, and multi-user workflow-thread support.
- (2026-03-28) Updated the plan after defining the external-state memory model, plain directory shards, and the policy/runtime/memory split.
- (2026-03-28) Updated the plan after approving `~/skarbot-state` as the default external VM state root.
- (2026-03-28) Updated the plan after deferring any standalone heartbeat concept for v1.
- (2026-03-28) Updated the plan after confirming that v1 needs no additional day-one integrations.
- (2026-03-28) Updated the plan after defining the file-first runtime state model and the event-driven consolidation triggers.
- (2026-03-28) Updated the plan after defining the three-root VM layout and clarifying that identity links belong in runtime state.
- (2026-03-28) Updated the plan after confirming that v1 adds no extra sandboxing inside the dedicated VM.
- (2026-03-28) Updated the plan after standardizing on `vm-dotfiles` as the VM prerequisite and narrowing `skarbot-core` to post-bootstrap setup and product behavior.
- (2026-03-28) Updated the plan after defining the Skarbot-owned post-bootstrap setup contract and initial admin registration flow.
- (2026-03-28) Updated the plan after defining the interactive-first plus non-interactive setup-config model.
- (2026-03-28) Updated the plan after defining `~/skarbot-state/config/` as the default generated instance-config location.
- (2026-03-28) Updated the plan after recording the draft recurring-task model and downgrading the plan from active to draft while work remains docs-only.
- (2026-03-28) Updated the plan after recording the admin-only web approval plus file-backed recurring-task draft model.
- (2026-03-28) Updated the plan after simplifying recurring tasks to one JSON file per task.
- (2026-03-28) Updated the plan after removing recurring-task approvals and switching to one `tasks` skill plus direct task-file edits.
- (2026-03-28) Updated the plan after confirming that `memory` is the only other built-in runtime skill needed in v1 besides `tasks`.
- (2026-03-28) Updated the plan after splitting `project-definition` into focused docs and recording the filesystem-first `memory` skill responsibilities.
- (2026-03-28T20:43Z) Updated the plan after separating repo baseline from learned memory and simplifying v1 memory scopes to deployment memory, per-user memory, and task-attached context.
- (2026-03-28) Updated the plan after defining a dedicated task JSON `memory` section for durable lessons rather than routine outputs.
- (2026-03-28) Updated the plan after placing routine recurring-task outputs in workflow thread history instead of task memory.
- (2026-03-28) Updated the plan after defining one recurring task per workflow thread and requiring the UI to distinguish task threads from user inbox threads.
- (2026-03-28T21:09Z) Updated the plan after collapsing direct human conversation into one long-lived thread per user and deriving task-thread UI grouping from a task link instead of a formal thread type.
- (2026-03-28T21:24Z) Updated the plan after defaulting direct replies to the initiating surface, requiring explicit task notification methods, and allowing `none` for task-thread-only workflows.
- (2026-03-28T21:31Z) Updated the plan after limiting v1 recurring-task notification methods to `none`, `direct-thread`, and `sms`.
- (2026-03-28T21:39Z) Updated the plan after changing recurring-task notifications from a singular method into a list field where `[]` means no external delivery and supported entries are `direct-thread` and `sms`.
- (2026-03-28T21:44Z) Updated the plan after fixing recurring-task notifications to always target the task owner in v1.
- (2026-03-28T22:02Z) Updated the plan after broadening tasks to cover scoped work in general and collapsing immediate, one-off, and recurring scheduling into one task model.
- (2026-03-28T22:11Z) Updated the plan after replacing task `status` with folder-based lifecycle under `tasks/`, `tasks/inactive/`, and `tasks/failed/`.
- (2026-03-28T22:18Z) Updated the plan after removing the separate task `channel` field from the v1 schema.
- (2026-03-28) Updated the plan after keeping the task `owner` field explicit in v1 and shifting the next question to task identifiers.
- (2026-03-28) Updated the plan after removing the separate task `id` field and making the filename canonical, then shifting the next question to filename style.
- (2026-03-28) Updated the plan after choosing human-readable slug filenames and shifting the next question to filename stability after title edits.
- (2026-03-28) Updated the plan after keeping filenames stable after creation and shifting the next question to collision handling for slug-based filenames.
- (2026-03-28) Updated the plan after resolving slug collisions with numeric suffixes and shifting the next question to whether `title` should remain explicit.
- (2026-03-28) Updated the plan after keeping explicit task `title` and shifting the next question to the `instructions` field shape.
- (2026-03-28) Updated the plan after keeping plain-string `instructions` and shifting the next question to the task `memory` shape.
- (2026-03-28) Updated the plan after keeping task `memory` as a simple list and shifting the next question to explicit task timestamps.
- (2026-03-28) Updated the plan after omitting explicit task timestamps and shifting the next question to the `owner` value shape.
- (2026-03-28) Updated the plan after narrowing task `owner` to a stable internal user id and then extending the docs to cover file-backed membership, `exe.dev` login-gated onboarding, and setup-created admin identity.
- (2026-03-28) Updated the plan after choosing `TOML` as the main generated config format.
- (2026-03-28) Updated the plan after splitting the secrets policy between `exe.dev` service integrations and model-provider auth.
- (2026-03-28) Updated the plan after choosing one main generated `TOML` file for v1.
- (2026-03-28) Updated the plan after deferring exact `TOML` sections and shifting the next question to concrete workflows.
- (2026-03-28) Updated the plan after prioritizing natural-language workflow scaffolding over a fixed initial workflow catalog.
