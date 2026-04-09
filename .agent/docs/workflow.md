# Workflow

Read this when normal task execution needs more detail than the top-level `AGENTS.md`.

## Default Task Flow

1. Determine goal and acceptance criteria.
2. Determine constraints and risks.
3. Identify files, commands, and tests to inspect.
4. Perform read-only inspection first.
5. Apply the execution gate before editing.
6. Present only blocking findings, questions, and tradeoffs for user resolution.
7. Propose a short execution plan.
8. Wait for agreement before editing.
9. Execute with small, reviewable diffs.
10. Verify with relevant project commands.
11. Update impacted documentation.

## Execution Gate

- If the task has concrete anchors and clear acceptance criteria, execute directly.
- If the task is broad, underspecified, or mostly architectural, stop after a short clarification or planning checkpoint before editing.
- Do not treat prompts like `make it better`, `clean this up`, or other broad requests as direct implementation prompts without first tightening scope.

## Findings Protocol

- Use a numbered list only for distinct blocking decisions, risks, or questions that require explicit user resolution.
- Merge closely related concerns instead of splitting one issue into multiple numbers.
- Keep numbering stable across back-and-forth.
- Keep the list as short as possible, usually 3-5 items.
- Use short prose for non-blocking observations and routine status updates.
- Do not begin execution until blocking numbered items are resolved.

## Research

Use web search when behavior is unfamiliar, version-sensitive, security-relevant, or when an error suggests a known fix.

Rules:

- Use targeted queries.
- Prefer primary sources.
- Check versions and dates.
- Cross-check non-primary sources before relying on them.
- Summarize conclusions in your own words.
- Cite sources when they materially affect a decision.
- Never search secrets or proprietary identifiers verbatim.

## Verification

A task is done when:

- the requested change is implemented or the question is answered
- impact is explained
- impacted documentation is updated
- relevant verification is provided
- follow-ups or omissions are called out explicitly

For source code changes, run as applicable:

- build
- lint
- test
- typecheck

Do not claim completion from inspection alone when the task can be verified with commands, tests, or observable behavior.
Prefer evidence over confidence.

If a failure is clearly pre-existing, call it out explicitly and propose whether to fix or defer it.
