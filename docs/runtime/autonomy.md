# Runtime Autonomy

Status: Research

## Git and Product Code

- `~/skarbot/core/` should remain product code under the user's control
- Changes to `~/skarbot/core/` should come through normal git workflow rather than runtime agent edits
- The user merges to `main`; the running agent does not self-merge

## Approval Boundary

- Approval policy and UX should be defined centrally in `docs/runtime/approvals.md`
- User approvals should resolve in the user's direct thread
- Admin approvals should resolve through the admin `exe.dev` email path in v1
- Admin approvals originating from user-owned work should require explicit user escalation from the direct thread
- SMS is an escalation channel, not an approval channel
- `exe.dev` sharing and integration changes remain human-controlled
- Outbound SMS is allowed only to explicitly allowlisted numbers

## Dedicated VM Model

- Because this is a dedicated VM, the agent is autonomous by default only inside Skarbot-owned runtime roots such as `~/skarbot/state/` and `~/skarbot/workspaces/`
- Host package installs should require admin approval
- `~/skarbot/core/` should be treated as product code and should not be edited by the running agent
- V1 should not add additional sandboxing or isolation inside the dedicated VM beyond the current approval rules and owned-root boundaries
- Destructive actions outside the configured owned roots still require approval
