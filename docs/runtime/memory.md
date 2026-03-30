# Runtime Memory

Status: Research

## Memory Model

- The repo baseline in `~/skarbot/core` is not memory; it is the static foundation for what Skarbot is, what skills it has, and how runtime files work
- Learned memory should stay outside git in workspace files
- V1 memory should stay extremely simple:
  - every direct-thread workspace should have a `MEMORY.md`
  - every task-thread workspace should have a `MEMORY.md`
- `MEMORY.md` should hold compact durable notes that help future turns in that workspace
- There should be no dedicated runtime memory tool in v1
- The agent should maintain `MEMORY.md` through the normal coding tools such as `read`, `write`, `edit`, and `grep`
- Routine outputs and run-by-run logs should stay in normal thread history rather than being copied into `MEMORY.md` by default
- Deterministic rules such as SMS allowlists, approval timing, roles, and owned roots must live in code or config rather than in memory
- Automatic thread compaction should not be treated as memory storage
- Compaction summaries should preserve ongoing goals, progress, decisions, next steps, and critical identifiers for future turns, but they should not replace durable `MEMORY.md`
- Full append-only thread history should remain available on disk even after compaction, while `MEMORY.md` stays limited to durable notes
- Searchable thread history is useful, but it should be defined later as a separate thread-history tool rather than as part of memory

## Memory Skill

- The built-in runtime skill name should be `memory`
- The `memory` skill should treat repo docs and skills as the baseline contract, not as mutable memory
- The `memory` skill should teach the agent to use the workspace-local `MEMORY.md` file rather than a dedicated memory tool
- When the user explicitly says to remember something, the `memory` skill should write it to disk
- The `memory` skill should support lightweight recall when the agent needs past context
- The `memory` skill should prefer filesystem operations such as direct reads and targeted search wherever practical

## Consolidation

- `MEMORY.md` should stay compact and should only capture durable facts that are worth seeing again in the same workspace
- Workflow runs are individual executions with terminal states such as `succeeded`, `failed`, or `cancelled`; a successful run does not imply the workflow itself is complete
- The recommended reference pattern is the file-first `MEMORY.md` approach used by `nanobot` and `mom`
- Pre-compaction memory flush is intentionally deferred from the initial compaction design; v1 should rely on rolling compaction summaries plus existing durable memory writes rather than a separate silent flush turn
