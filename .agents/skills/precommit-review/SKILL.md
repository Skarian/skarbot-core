---
name: precommit-review
description: Use when the user requests a phased pre-commit review of the worktree.
---

# Pre-Commit Review

Trigger: use when the user asks for a commit-readiness review in phases (source, tests, commit-message prep).

Run phases sequentially. Do not auto-advance from one phase to the next.
After each phase:

- Present findings.
- Pause for user-directed fixes/discussion.
- Continue to the next phase only when the user asks to proceed.

## Phase 1: Source Review

- Start by enumerating the full current worktree (staged, unstaged, and untracked files).
- Treat the full worktree as the primary review scope, not only the most recent edits.
- Review source changes in that scope exhaustively.
- Read full files for changed files and tangentially related files as needed to validate behavior.
- Focus on bugs, regressions, race conditions, broken UX, contract breaks, and risky side effects.
- Use concrete evidence from code paths and call sites, not speculation.

## Phase 2: Test Review

- Review changed tests in the worktree exhaustively.
- Review the broader existing test suite for drift against current behavior.
- Identify stale tests worth removing, missing coverage worth adding, and mismatches between tests and intended behavior.
- Call out whether test gaps are commit-blocking or safe to defer.

## Phase 3: Commit-Prep Output

- Produce commit message options in common commit style based on shipped behavior/features.
- After the user selects one commit message option, provide a commit description/body for that selected option.
- Do not mention planning artifacts (`.agents/*`, ExecPlans, continuity files) in commit text.
- Do not execute `git commit`; provide message options and let the user run the commit.

## Phase-Gated Output Format

Only output sections for the current phase.
Never output future-phase sections.
Never output placeholder `none` lines for phases that have not run yet.

Phase 1 response:

1. `Phase 1 Findings (Source Review)`
2. `Must-fix before Phase 2`
3. `Safe to defer`
4. `Gate status` with: `Phase 1 complete. Ask me to proceed to Phase 2 (test review) when ready.`

Phase 2 response:

1. `Phase 2 Findings (Test Review)`
2. `Must-fix before commit`
3. `Safe to defer`
4. `Gate status` with: `Phase 2 complete. Ask me to proceed to Phase 3 (commit prep) when ready.`

Phase 3 response (step A):

1. `Commit message options`
2. `Gate status` with: `Select one option number and I will provide the commit description.`

Phase 3 response (step B, after user selects an option):

1. `Selected commit message`
2. `Commit description`
3. `Gate status` with: `Commit message package ready. I will not run git commit.`

Formatting rules:

- Every finding/proposal is a numbered item on its own line.
- Never combine multiple findings into one numbered line.
- Prefix each finding with severity: `[critical]`, `[high]`, `[medium]`, or `[low]`.
- Include a file reference for each code/test finding when applicable.
- If a current-phase findings section has no items, write exactly `1. none`.

Commit message option rules:

- Provide 2-4 options.
- Use `type(scope): summary` style.
- Keep each option to one line.
- After the user picks an option, provide a commit description body with 3-6 concise bullet lines focused on shipped behavior and key file-level changes.
- Never execute `git commit` as part of this skill.
