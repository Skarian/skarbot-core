# Continuity Ledger

Read this only in repos that actively use `.agent/CONTINUITY.md`.
Treat continuity as a recovery artifact, not a per-turn ritual.

## Purpose

- `.agent/CONTINUITY.md` is the canonical short briefing for future sessions.
- Do not rely on earlier chat or tool output unless it is reflected there.

## When To Read

Read `.agent/CONTINUITY.md`:

- at session start
- after compaction or other context loss
- when resuming paused work
- when the user references prior decisions or state

Do not reread it every turn unless there is a specific reason.

## When To Update

Update only when there is a meaningful delta in:

- goal or success criteria
- constraints or invariants
- decisions
- state: done, now, next
- open questions
- working set
- important tool outcomes or verification results

## Format Rules

- Facts only, no transcripts.
- Every entry includes a date or timestamp.
- Every entry includes a provenance tag:
  - `[USER]`
  - `[CODE]`
  - `[TOOL]`
  - `[ASSUMPTION]`
- If unknown, write `UNCONFIRMED`.
- If something changes, supersede it explicitly instead of silently rewriting history.

## Size Limits

- Keep the file short and high-signal.
- Keep `Snapshot` concise.
- Keep `Done (recent)` bounded.
- Keep `Working set` bounded.
- Keep `Receipts` focused on recent durable outcomes.

If a section grows too large, compress older items into milestone bullets with pointers instead of pasting raw logs.

## In Replies

- Use a brief ledger snapshot only when useful.
- Print the full ledger only on request or after meaningful changes.
