# ExecPlan Index

This file tracks all ExecPlans for this repository. It is required by `.agent/PLANS.md`.

## Conventions

- Draft plans live in: `.agent/execplans/draft/`
- Active plans live in: `.agent/execplans/active/`
- Archived plans live in: `.agent/execplans/archive/`
- Plan filename format: `EP-YYYY-MM-DD__slug.md`
- Plan header fields live inside each plan file and must match the index entry.

## Index entry format (use this consistently)

For each plan, add a single bullet in the appropriate section:

- `EP-YYYY-MM-DD__slug` — `<Title>` — `Status:<DRAFT|ACTIVE|BLOCKED|DONE|ARCHIVED>` — `Created:YYYY-MM-DD` — `Updated:YYYY-MM-DD` — `Path:<repo-relative path>` — `Owner:<UNCONFIRMED|name>` — `Summary:<one line>` — `Links:<optional>`

For archived plans, also include:

- `Archived:YYYY-MM-DD` — `Outcome:<one line>`

Keep entries short, greppable, and consistent.

If the user request does not specify plan type, ask: `What type is this ExecPlan: draft, active, or archive?` before adding or moving entries.

## Draft ExecPlans

- `EP-2026-03-28__skarbot-foundation` — `Skarbot Foundation and Project Definition` — `Status:DRAFT` — `Created:2026-03-28` — `Updated:2026-03-28` — `Path:.agent/execplans/draft/EP-2026-03-28__skarbot-foundation.md` — `Owner:UNCONFIRMED` — `Summary:Tracks reference imports and the split docs discovery process during the research phase.` — `Links:docs/overview.md,docs/research/reference-projects.md`

## Active ExecPlans

- (none yet)

## Archived ExecPlans

- (none yet)
