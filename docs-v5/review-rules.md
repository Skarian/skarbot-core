# Review rules

This page defines the working rules for the current `docs-v5` review pass

## What this pass is

- `docs-v5` is the active editing set for the current review
- the goal is to refine `docs-v4` into a cleaner canonical doc set without losing implementation-relevant detail
- edits happen one file at a time in reading order
- no file is edited until the intended changes for that file are discussed and approved

## Editorial rules

- avoid the term `surfaces`
- avoid the term `sidecar`
- avoid the term `substrate`
- prefer `channels` when describing communication paths
- prefer `UX` when the real point is user experience
- prefer `user thread` over `direct thread`
- avoid terminal periods on bullets and list items
- preserve user-authored wording unless there is a clear consistency, clarity, or correctness reason to change it
- keep the docs canonical and implementation-relevant without over-specifying one area far beyond the others
- keep wording direct, plain, and product-specific rather than generic or inflated
- do not rely on concepts before they are introduced clearly enough for a human reader
- treat user approvals over web and SMS as an intentional product change during the `docs-v5` review
- when a file format, message structure, or runtime behavior is inherited from `pi`, include a pinned GitHub permalink to the relevant `pi` source or docs at first mention

## Working style

- review pages in this order: `README`, `product`, `architecture`, `runtime`, `deployment`, then appendices
- after each agreed edit pass, the user may make additional tweaks before moving on
- the rules on this page are for the `docs-v5` editing session and can be folded into the main docs later if they become durable project rules
