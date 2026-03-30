# Overview

Status: Research

## Purpose

Build a personalized claw-like assistant monorepo on top of `pi.dev` so a single user can bootstrap an `exe.dev` VM with `vm-dotfiles`, run Skarbot-specific setup from this repo, and get a tailored assistant with a deliberately small feature set.

## Core Constraints

- Personal-first over platform-first
- Keep the actual claw agent lightweight from a code standpoint
- Choose a small subset of features rather than copying full upstream surfaces
- Prefer explicit setup scripts over manual machine drift
- Treat filesystem artifacts as the source of truth wherever practical

## V1 Summary

- Base runtime: `pi.dev`
- Primary deployment target: one Linux VM on `exe.dev`
- Standard VM prerequisite: `vm-dotfiles`
- Core surfaces: web UI, admin inbound email, SMS via ClickSend, scheduled jobs / cron jobs
- Core runtime skills: `tasks` and `memory`
- Standard runtime tools: `pi` coding core plus a small Skarbot system layer defined in `docs/runtime/tools.md`
- V1 model provider: deployment-wide OpenAI Codex subscription auth with `openai-codex/gpt-5.4` as the default model
- Live private state stays outside git under `~/skarbot/state` and `~/skarbot/workspaces`
- Learned memory should stay file-first through workspace-local `MEMORY.md` files rather than a dedicated runtime memory tool
- Membership is app-managed inside Skarbot, with `exe.dev` login used to identify users

## Architecture Summary

- One shared adapter-based core for admin email, SMS, web UI, and scheduled jobs
- Inbound-first communication model
- Thin web UI over the same conversation and task system
- Public `exe.dev` proxy with in-app login gating for protected routes
- Recurring workflows scaffolded from natural-language requests
- Each human user has one long-lived direct thread in v1
- Each scoped task owns one task-linked thread in v1
- Each thread owns one execution workspace in v1, and runtime behavior is centralized in the two execution profiles defined in `docs/runtime/execution-profiles.md`
- The standard tool surface should reuse `pi`'s built-in coding substrate and add only a small Skarbot-specific layer
- Thread and session storage should follow `pi-coding-agent` session files and `pi-agent-core` or `pi-ai` message types rather than a custom transcript schema
- Filesystem-first runtime state, tasks, and memory
- Repo docs plus repo-backed system capabilities under `~/skarbot/core/capabilities/` define the static Skarbot baseline; learned memory and user-promoted capabilities live under `~/skarbot/state`
- One shared approval system handles both user approvals and admin approvals, with delivery surface determined by approver type

## Doc Map

- [Providers](providers.md)
- [Channels](channels.md)
- [Web UI](web-ui.md)
- [Users and Threads](users-and-threads.md)
- [Runtime Platform and Setup](runtime/platform-and-setup.md)
- [Runtime Execution Profiles](runtime/execution-profiles.md)
- [Runtime Tools](runtime/tools.md)
- [Runtime Approvals](runtime/approvals.md)
- [Runtime Autonomy](runtime/autonomy.md)
- [Runtime Tasks](runtime/tasks.md)
- [Runtime Memory](runtime/memory.md)
- [Reference Research](research/reference-projects.md)
- [exe.dev Research](research/exe-dev.md)
