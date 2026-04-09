# Implementation

Read this when implementing or reviewing code changes.

## Code Standards

- Prefer self-explanatory code.
- Avoid comments unless they are truly necessary.
- Refactor for readability before adding commentary.
- Prefer small, cohesive modules with stable seams.
- Treat large files as a design signal, not an automatic violation.
- Keep entrypoints stable.
- Isolate change-prone logic behind small seams.
- Fail loudly and diagnostically.
- Do not swallow errors.
- Design UI for the end-user, not the schema.

## Editing Rules

- Make the smallest durable change that solves the problem at the right abstraction level.
- Preserve sound existing style and architecture, but do not preserve weak or accidental structure just because it exists.
- Prefer focused diffs over broad rewrites, but make modest structural improvements when they materially reduce future churn, especially in small or early-stage codebases.
- When the existing structure is causing symptom-level patches to accumulate, prefer a small architectural correction over another local workaround.
- Keep mechanical edits separate from logic changes when practical.
- Commit messages should follow Conventional Commits, for example `type(scope): summary`.
- Do not reference `.agent` artifacts in commit messages.

## Dependencies

Do not reinvent standardized functionality without checking whether a mature dependency is the better choice.

When the dependency decision is non-trivial:

- ask the user whether they want a library
- present a short evaluation
- recommend one option or a short list of viable options
- ask for selection when the choice is material

The evaluation should cover:

- maintenance health
- adoption signals
- license fit
- security posture
- compatibility
- integration cost

When adding a dependency:

- justify why it is needed
- note licensing and security considerations
- prefer the smallest dependency surface that solves the problem
