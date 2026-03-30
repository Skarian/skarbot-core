# Deployment

This document defines the host contract for running Skarbot. It covers the platform assumption, setup flow, auth, config, and integrations. Product behavior lives in [Product](product.md). Persistent data layout lives in [Architecture](architecture.md).

## Standard deployment target

Skarbot is designed for one Linux VM on `exe.dev`.

That choice is deliberate:

- `exe.dev` supplies the web proxy
- `exe.dev` supplies the authenticated web identity
- `exe.dev` supplies the inbound admin email surface
- `exe.dev` can supply GitHub access without storing long-lived repo tokens on the VM

`vm-dotfiles` is the expected machine bootstrap. Local development on macOS or WSL is useful, but the normal operating loop is on-box.

## Host roots

```text
~/skarbot/core
~/skarbot/state
~/skarbot/workspaces
~/.skarbot/auth.json
```

Ownership:

- `~/skarbot/core` — product code and repo-backed system capabilities
- `~/skarbot/state` — canonical runtime state
- `~/skarbot/workspaces` — thread workspaces and scratch
- `~/.skarbot/auth.json` — deployment-wide Codex auth

Model auth is host-managed deployment state, not portable Skarbot state.

## Setup flow

Setup supports two modes:

- interactive first-run setup
- non-interactive reruns for automation and recovery

The setup flow is responsible for:

1. validating the expected directory roots
2. validating repo-backed system capability roots
3. creating runtime state directories
4. writing generated runtime config
5. creating the initial admin user record
6. writing the protected admin config
7. validating integrations and host assumptions
8. starting the long-running runtime processes
9. wiring scheduled jobs and local cleanup commands
10. printing health and status endpoints

The initial admin user is created directly in `~/skarbot/state/users/active/`.

## Provider contract

Skarbot uses one model provider for the deployment:

- `openai-codex`

The admin logs into Codex once on the VM. That subscription auth is shared by all Skarbot sessions on that VM.

### Model auth rules

- auth lives in `~/.skarbot/auth.json`
- auth is not stored in the repo
- auth is not written into generated `TOML`
- portable runtime state under `~/skarbot/state` does not contain raw model credentials

### Model selection rules

- the deployment default model is `openai-codex/gpt-5.4`
- new threads start on that default
- thread-level model switches persist per thread
- thread-level model switches do not rewrite the deployment default
- Skarbot does not define model fallbacks in the deployment contract

## Integrations

Skarbot expects a narrow set of integrations.

| Integration | Role |
| --- | --- |
| `exe.dev` proxy | Web app exposure |
| `exe.dev` authenticated email | Web identity |
| `exe.dev` inbound email | Admin email ingress |
| ClickSend | SMS delivery and inbound SMS |
| `exe.dev` GitHub integration | Repository access without storing tokens on the VM |

Non-model integrations should prefer `exe.dev` integrations or proxy-backed secrets over raw secrets stored on disk.

## Configuration

Generated instance config lives under:

```text
~/skarbot/state/config/
```

The main generated config is one `TOML` file. The repo may contain templates and defaults, but it does not contain the live instance config.

Environment variables are for temporary overrides and automation. They are not the primary configuration system.

## Operations

Skarbot runs as a long-lived service plus scheduled local maintenance.

Operational expectations:

- the scheduler reads only active task files
- task workspace cleanup is deterministic and zero-token
- task workspace deletion requires both an inactive task file and an empty `.deleteworkspace` marker in the workspace root
- failed task workspaces are retained
- cleanup runs at startup and on a local cron schedule

## Security model

Skarbot assumes one trusted admin controlling one dedicated VM.

Security posture:

- keep product code under git, not under runtime mutation
- keep secrets out of the repo
- prefer deployment-managed integrations over ad-hoc local secrets
- require approval for actions that cross owned-root or host boundaries

## See also

- [Architecture](architecture.md)
- [Runtime](runtime.md)
- [appendix/exe-dev](appendix/exe-dev.md)
