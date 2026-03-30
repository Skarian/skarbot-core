# Reference Projects

These repositories are imported as read-only research submodules under `references/`. They are here to mine architecture, setup, and product ideas, not to copy wholesale.

## Imported References

| Project | Local Path | Upstream | Pinned Commit | Why It Matters |
| --- | --- | --- | --- | --- |
| `pi-mono` | `references/pi-mono` | `https://github.com/badlogic/pi-mono` | `576e5e1a2fbe1abbbad96b696f4058cffd8391ca` | Core `pi.dev` runtime, extension model, SDK/RPC surfaces, and the minimal-harness philosophy |
| `openclaw` | `references/openclaw` | `https://github.com/openclaw/openclaw` | `23772bb785731126d723fe1d82cf662ba625ab6c` | Full `pi`-based personal assistant reference with onboarding, gateway, channels, security defaults, and multi-agent routing |
| `nanoclaw` | `references/nanoclaw` | `https://github.com/qwibitai/nanoclaw` | `29839464bff39b7672c0a7504d9bb7649a79c5f1` | Small bespoke claw pattern with strong container isolation and a skills-over-features philosophy |
| `nanobot` | `references/nanobot` | `https://github.com/HKUDS/nanobot` | `c8c520cc9a4dbe619eb3f21200dc40971a36b665` | Ultra-lightweight OpenClaw-inspired implementation with a simpler Python codebase and broad channel support |
| `nemoclaw` | `references/nemoclaw` | `https://github.com/NVIDIA/NemoClaw` | `eb4ba8c486ad97ea3ca67bea388690b641dc134e` | Sandbox and deployment ideas for running an always-on claw more safely, especially around container/runtime boundaries |
| `oh-my-pi` | `references/oh-my-pi` | `https://github.com/can1357/oh-my-pi` | `82b57365d421d05237149a70472444a455bd6d88` | A feature-rich `pi` fork with advanced tooling ideas such as LSP, Python kernel, hash-anchored edits, and commit tooling |

## What To Study

### `pi-mono`

- Keep the base harness small.
- Use packages, skills, and extensions instead of baking everything into the core.
- Favor RPC/SDK seams that let `skarbot-core` stay repo-specific.

### `openclaw`

- Study onboarding flow, gateway structure, channel routing, and security defaults.
- Mine concepts, not the entire surface area.
- Treat its breadth as a warning against overbuilding the first version.

### `nanoclaw`

- Study the bias toward bespoke forks instead of generic frameworks.
- Evaluate container-per-context isolation and skills as a distribution mechanism.
- Keep its "small enough to understand" bar in view while defining scope.

### `nanobot`

- Study how much assistant surface area can fit into a much smaller codebase.
- Inspect its memory simplification, provider layout, and lighter-weight packaging ideas.
- Track security-sensitive dependency choices carefully.

### `nemoclaw`

- Study hardening, sandbox/runtime assumptions, and install flow.
- Avoid inheriting heavyweight infrastructure unless it directly serves the single-user VM target.

### `oh-my-pi`

- Study selective harness upgrades that may improve developer ergonomics without bloating runtime behavior.
- Treat most of it as optional inspiration, not a default dependency target.

## Current Synthesis

- The repo should stay closest to `pi-mono` in core philosophy.
- `openclaw` is the main product/operations reference, not the model for code size.
- `nanoclaw` and `nanobot` are the best counterweights against complexity creep.
- `nemoclaw` is useful for security and sandbox research, not for first-pass footprint.
- `oh-my-pi` is the main reference for optional harness ergonomics if they can stay modular.

## Memory and State Synthesis

- `nanobot` is the strongest v1 reference for durable memory shape: a compact `MEMORY.md` loaded into context plus an append-only history file searched on demand.
- `pi-mono/mom` is the strongest reference for separating prompt context from searchable audit history via `context.jsonl` and `log.jsonl`, plus shared and per-channel `MEMORY.md`.
- `nanoclaw` shows a simpler workspace-and-history approach, but its memory is looser and depends more on workspace files, conversation folders, and Claude auto-memory than on a portable first-class memory format.
- `oh-my-pi` proves that a richer memory pipeline is possible, including `memory_summary.md`, generated skills, and `memory://` access, but it adds more machinery than `skarbot-core` needs in v1.
- `openclaw` provides a powerful retrieval-oriented memory system with indexing and search backends, but it is intentionally broader and heavier than the current Skarbot target.
- The recommended Skarbot direction is to keep deterministic policy outside memory, store private state outside git, and use plain directory shards for users and workflows with `MEMORY.md` plus append-only history.
