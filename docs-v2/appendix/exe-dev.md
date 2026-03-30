# exe.dev appendix

This appendix explains why `exe.dev` is the standard deployment target for Skarbot. It is background, not the product contract.

## Why `exe.dev` fits

Skarbot wants a single always-on VM with just enough platform help to avoid rebuilding infrastructure. `exe.dev` provides the pieces that matter most:

- HTTPS exposure for the web app
- authenticated web access
- inbound email to the VM
- GitHub integration without normal PAT sprawl
- SSH-first operations

That makes `exe.dev` a strong fit for a small self-hosted assistant.

## Platform features Skarbot depends on

### Web exposure

Skarbot uses the built-in `exe.dev` proxy for the initial web app instead of adding a separate exposure layer.

### Inbound email

Inbound admin email lands in Maildir on the VM. Skarbot processes mail locally and moves handled messages through the normal Maildir lifecycle.

### GitHub access

When repository access is needed, the preferred path is an `exe.dev` GitHub integration rather than tokens stored inside the VM.

### Agent-friendly environment

`exe.dev` is opinionated toward remote coding and agent workflows, which matches Skarbot’s operating model.

## What this means for Skarbot

Standardizing on `exe.dev` keeps the Skarbot codebase smaller. Platform work stays platform work. Product code stays focused on the assistant itself.

That is the whole point of the deployment choice.
