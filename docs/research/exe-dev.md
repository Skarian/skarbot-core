# exe.dev Research

`exe.dev` is not just the hosting target for `skarbot-core`; it is part of the system contract for v1.

## Why It Matters

The planned first release runs on a single Linux VM on `exe.dev`, and the platform already provides several things that would otherwise become product work:

- HTTPS exposure for a web service
- inbound email delivery to the VM
- GitHub integration without storing tokens on the VM
- SSH-first remote access and automation hooks

That makes `exe.dev` a valid standardization target for the initial version.

## Confirmed Platform Capabilities

### VM model

- `exe.dev` provides persistent Linux VMs with immediate HTTPS reachability and optional auth defaults.
- VMs share CPU/RAM; pricing is based on underlying resources rather than per-VM billing.

### HTTP proxy / web exposure

- `https://vmname.exe.xyz/` is proxied to the VM.
- `exe.dev` handles certificates and TLS termination.
- Access is private by default and can be made public explicitly.
- Port selection can be changed while keeping the same proxy surface.

### Inbound email

- A VM can receive mail at `*@vmname.exe.xyz`.
- Delivered mail lands in `~/Maildir/new/`.
- Mail uses Maildir format and includes an injected `Delivered-To:` header.
- Processing is the VM's responsibility; a large backlog disables receiving automatically.
- Current limitations include no custom domains yet, 1 MB max message size, and no spam/virus filtering.

### GitHub integration

- `exe.dev` can attach a GitHub integration per repository.
- The integration allows private repo access from the VM without storing GitHub tokens inside the VM.
- Access is scoped to the configured repository.

### Agent-facing docs

- `exe.dev` publishes an agent skill and docs index specifically for coding agents.

## Implications For `skarbot-core`

- Standardize v1 on one Linux VM running on `exe.dev`.
- Treat local development as optional convenience, not a release requirement.
- Use the built-in `exe.dev` HTTP proxy for the initial web UI instead of designing a custom exposure layer.
- Implement inbound email by watching `~/Maildir/new/` on the VM and moving processed mail into `cur/`.
- Prefer `exe.dev` GitHub integrations over PAT management on the box.
- Keep `skarbot-core` portable in code, but platform-opinionated in the first deployment docs and setup flow.

## Source Notes

- `https://exe.dev/llms.txt`
- `https://exe.dev/docs/what-is-exe.md`
- `https://exe.dev/docs/proxy.md`
- `https://exe.dev/docs/receive-email.md`
- `https://exe.dev/docs/integrations-github.md`
- `https://exe.dev/docs/agent-skill.md`
