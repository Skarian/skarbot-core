# Skarbot

Skarbot is a self-hosted personal assistant for one trusted household. It runs on a single `exe.dev` VM, is administered by one owner, embeds the pi coding agent SDK, and uses deployment-wide Codex subscription auth on that VM. The product is intentionally small: one runtime, one data model, one set of owned filesystem roots, and a thin set of user surfaces over pi.

This doc set is the canonical contract for the project. There is no separate “details” layer. If a behavior matters to implementation, it belongs here. The appendices are background only.

## Design principles

- **Keep the core small.** Reuse pi instead of rebuilding an agent stack.
- **Optimize for one trusted deployment.** Skarbot is an admin-managed household assistant, not a general multi-tenant platform.
- **Keep state on disk.** Prefer ordinary files and directories over services.
- **Keep product code under git and runtime state out of git.**
- **Use one thread model everywhere.** Web, SMS, admin email, and scheduled work all resolve to the same thread and task system.

## Document map

- [Product](product.md) — what Skarbot is, who it serves, and how people interact with it
- [Architecture](architecture.md) — the persistent data model: users, threads, tasks, approvals, capabilities, logs, and files
- [Runtime](runtime.md) — how runs are composed, which tools exist, how task execution works, how approvals behave, and where autonomy stops
- [Deployment](deployment.md) — the host contract, setup flow, auth, config, integrations, and operations
- [exe.dev appendix](appendix/exe-dev.md) — why `exe.dev` is the standard deployment target
- [Reference projects appendix](appendix/reference-projects.md) — outside projects that influence the design without defining it

## Project boundaries

Skarbot owns the assistant product, the runtime state under `~/skarbot/state`, and the execution workspaces under `~/skarbot/workspaces`.

Skarbot does **not** treat `~/skarbot/core` as mutable runtime state. Product code stays under normal git workflow. The running agent may operate inside its owned runtime roots, but it does not rewrite the product and merge its own changes.

## Canonical vocabulary

- **Direct thread** — the long-lived conversation for one approved user
- **Task thread** — a scoped thread for a task, workflow, or capability-building effort
- **Workspace** — the filesystem working directory attached to one thread
- **Capability** — a reusable skill or tool package loaded into a user’s runs
- **Candidate capability** — a staged capability under review, not part of the user’s normal runtime
- **Approval** — a file-backed decision request with deterministic reply handling

## Reading order

Start with this file, then read [Product](product.md), [Architecture](architecture.md), and [Runtime](runtime.md). Read [Deployment](deployment.md) when you need host, auth, or setup details. Read the appendices only when you need background.
