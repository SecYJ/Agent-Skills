---
name: overlay-host-architecture
description: Consolidate duplicated dialog, drawer, modal, or side-panel shells into one feature-owned Overlay Host with swappable content. Use when implementing or refactoring overlays that repeat the same outer infrastructure.
---

# Overlay Host Architecture

One stable outer host, replaceable inner content. Follow the project's existing framework, primitives, and state patterns; do not introduce a specific library.

## Inspect First

Find every related overlay and compare the outer layer: mounting/portal, backdrop, container, size and position, animation, close lifecycle, keyboard, focus, and accessibility semantics. Separate what is truly shared from what varies by content.

Decide from this evidence and proceed. Ask only when a product or behavior choice cannot be inferred and a wrong guess would be risky.

## The Pattern

```text
Overlay Host   (shared outer infrastructure)
`-- Current Content
    |-- Content A
    |-- Content B
    `-- Content C
```

- **Host owns**: mounting, backdrop and outside clicks, container, dimensions and responsive layout, transitions, open/close/escape, focus, and accessibility.
- **Content owns**: its own UI, actions, data, and validation.
- Switch content with the simplest mechanism the project already uses.
- Keep the host mounted across content changes; do not rebuild the shell per view.

## Scope

- Scope the host to the feature or flow, not the whole app.
- If overlays differ materially in role, positioning, animation, or lifecycle, use separate hosts instead of a heavily configurable one.
- A single simple overlay needs no host.

## State

- Prefer local state, props, or composition. Content switching alone does not justify a store, context, or router.
- Add feature-scoped shared state only when content views coordinate state or it must survive transitions; keep it per-instance so separate overlays do not share state.
- Do not pull app-wide state into the host.

## Implementation

1. Extract the duplicated shell into the host; keep each content view as its own unit, without one big conditional component or needlessly tiny extractions.
2. Keep feature logic next to the content that uses it.
3. Avoid unrelated abstractions or framework changes.
4. Verify open/close, content transitions, state retention or reset, keyboard and focus, responsive layout, and instance isolation.
