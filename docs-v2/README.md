# Skarbot

Skarbot is a self-hosted personal assistant for a small household. It runs on a single `exe.dev` VM, is administered by one owner, embeds the pi coding agent SDK, and uses shared Codex subscription auth on that VM. The product is intentionally small: a thin interface layer over pi, a filesystem-first runtime, and a narrow set of channels.

## Design goals

- **Keep the core small.** Reuse pi instead of rebuilding an agent stack.
- **Optimize for one trusted deployment.** Skarbot is an admin-managed household assistant, not a generic multi-tenant platform.
- **Keep state on disk.** Use ordinary files and directories before inventing services.
- **Keep product code under git and runtime state out of git.**
- **Use one conversation model everywhere.** Web, SMS, admin email, and scheduled work all resolve to the same thread and task system.

## Document map

- [Product](product.md) — what Skarbot is, who it serves, and how people interact with it
- [Architecture](architecture.md) — the persistent data model: users, threads, tasks, capabilities, and files
- [Runtime](runtime.md) — how agent turns run, which tools exist, how memory works, and where approvals are required
- [Deployment](deployment.md) — the host contract, setup flow, auth, config, and integrations
- [exe.dev appendix](appendix/exe-dev.md) — why `exe.dev` is the standard deployment target
- [Reference projects appendix](appendix/reference-projects.md) — outside projects that shape the design without defining it

## Project boundaries

Skarbot owns the assistant product, the runtime state under `~/skarbot/state`, and the execution workspaces under `~/skarbot/workspaces`.

Skarbot does **not** treat `~/skarbot/core` as mutable runtime state. Product code stays under normal git control. The running agent can work inside its owned runtime roots, but it does not rewrite the product and merge its own changes.

## Canonical vocabulary

- **Direct thread** — the long-lived conversation for one approved user
- **Task thread** — a scoped thread for a task, workflow, or capability-building effort
- **Workspace** — the filesystem working directory attached to a thread
- **Capability** — a reusable skill or tool package loaded into a user’s runs
- **Candidate capability** — a staged capability under review, not part of the user’s normal runtime

## Reading order

Start with this file, then read [Product](product.md), [Architecture](architecture.md), and [Runtime](runtime.md). Read [Deployment](deployment.md) when you need host, auth, or setup details. The appendices are context, not contract.
