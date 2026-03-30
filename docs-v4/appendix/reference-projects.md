# Reference projects appendix

These projects influence Skarbot, but none of them define it. The goal is to borrow shape and discipline, not surface area.

## Primary influences

### pi-mono

Main source of truth for the underlying agent philosophy:

- small core
- strong SDK seam
- extensions, skills, and packages instead of hard-coded product complexity

### openclaw

Useful as a product reference:

- onboarding
- channel routing
- security defaults
- real-world SDK integration shape

Skarbot borrows ideas from it without copying its breadth.

### nanoclaw and nanobot

Useful as anti-bloat references:

- keep the implementation small
- prefer understandable architecture over elaborate frameworks
- treat skills and focused capabilities as leverage

## Secondary influences

### nemoclaw

Relevant for runtime hardening and sandbox ideas when they directly serve the dedicated-VM model.

### oh-my-pi

Relevant for optional ergonomics and extension ideas, but not a default dependency target.

## Synthesis

The intended shape is simple:

- closest to pi-mono in philosophy
- informed by openclaw for product structure
- kept honest by nanoclaw and nanobot on code size and complexity
- selectively informed by nemoclaw and oh-my-pi only when the gain is real

If an idea makes Skarbot larger without making the household assistant materially better, it does not belong.
