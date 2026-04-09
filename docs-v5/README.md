# Skarbot

Skarbot is a self-hosted personal assistant for one's household. It runs on a single VM deployed to `exe.dev`, is administered by one owner, builds on top of the pi agent SDK, and is powered by a personal OpenAI ChatGPT subscription on that VM. Skarbot is intentionally simple and lightweight.

This doc set is the canonical contract for the project.

## Design principles

- **Keep the core small.** Reuse the pi agent SDK and extensions instead of rebuilding an agent stack from scratch
- **Keep state on disk.** Rely on the filesystem's layout and placement over complex config
- **Meet users where they are.** Skarbot should be usable through web, SMS, and email
- **Adapt to each user.** User interactions should be tailored to the user with separate memory, tools, skills and scheduled tasks
- **Stay extensible.** The agent should be able to create and refine tools and skills to better meet its user's needs
- **Design for a household UX.** Favor clarity, low operational overhead, and approachable channels over power-user complexity

## Document map

- [Product](product.md) — what Skarbot is, who it serves, and how people interact with it
- [Architecture](architecture.md) — the persistent data model: users, threads, tasks, approvals, capabilities, logs, and files
- [Runtime](runtime.md) — how runs are composed, which tools exist, how task execution works, how approvals behave, and where autonomy stops
- [Tools](tools.md) — the detailed contract for built-in coding tools, shared system tools, and profile-specific tools
- [Deployment](deployment.md) — the host contract, setup flow, auth, instance files, integrations, and operations
- [exe.dev appendix](appendix/exe-dev.md) — why `exe.dev` is the standard deployment target
- [Reference projects appendix](appendix/reference-projects.md) — outside projects that influence the design without defining it

## Project boundaries

This repo resides in `~/skarbot/core`. It defines the assistant product, manages runtime state under `~/skarbot/state`, execution workspaces under `~/skarbot/workspaces`, and a small set of host-managed deployment files under `~/.skarbot`. `~/skarbot/core` is managed via GitOps and cannot be mutated by runtime state.

## Canonical vocabulary

- **User thread** — long-lived conversations for each approved user
- **Task thread** — a thread scoped for a task, workflow, or capability-building effort
- **Workspace** — the filesystem working directory attached to each thread
- **Capability** — a reusable skill or tool package made available to the agent per user
- **Candidate capability** — a staged capability under review, not part of the user’s normal runtime
- **Schedule** — an active task file with a `schedule` field
- **Schedule handle** — the task filename stem
- **Approval** — a file-backed decision request whose lifecycle and approver class are encoded by its directory path

## Reading order

Start with this file, then read [Product](product.md), [Architecture](architecture.md), and [Runtime](runtime.md). Read [Deployment](deployment.md) when you need host, auth, or setup details. Read the appendices only when you need background.
