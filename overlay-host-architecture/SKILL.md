---
name: overlay-host-architecture
description: Consolidate duplicated dialog, drawer, modal, side-panel, and similar overlay shells into a feature-owned Overlay Host while keeping differing content separate. Use when implementing or refactoring overlays that repeat their outer infrastructure across content views; adapt to the project's existing framework and primitives.
---

# Overlay Host Architecture

Solve duplicated overlay infrastructure with the smallest feature-owned architecture that remains easy to read. The primary pattern is one stable outer host with replaceable or selectable content.

Do not bind the implementation to a particular UI framework, state library, dialog library, or routing mechanism. Follow the current project's stack and conventions.

## Work From Evidence Without Repeated Questions

Inspect the relevant code before implementing:

- Find every related dialog, drawer, modal, side panel, or overlay view.
- Compare their outer structure and behavior, including the backdrop, portal or equivalent mounting mechanism, container, dimensions, positioning, animation, close lifecycle, keyboard handling, focus behavior, and accessibility semantics.
- Identify what is truly shared and what varies by content or workflow state.
- Inspect nearby project primitives and existing state patterns before introducing anything new.

Make the architecture decision from this evidence and continue. Do not ask the user to answer a standard checklist whenever the skill triggers. Ask only when a material product or behavior choice cannot be discovered from the code or request and making an assumption would be risky.

## Prefer an Overlay Host for Duplicated Shells

When related overlays repeat substantially the same outer layer and differ mainly in content, consolidate that outer layer into one feature-owned Overlay Host:

```text
Overlay Host
|-- Shared outer infrastructure
`-- Current Content
    |-- Content A
    |-- Content B
    `-- Content C
```

The host owns the concerns common to all content views:

- mounting or portal behavior
- backdrop and outside interaction
- dialog, drawer, modal, or panel container
- shared dimensions, positioning, and responsive behavior
- animation and transition lifecycle
- open, close, escape, focus, and accessibility behavior

Content views own only their content-specific UI, actions, data, validation, and behavior. Switch, inject, compose, or route content using the simplest mechanism already natural to the project.

Keep the host mounted while content changes when that preserves shared lifecycle, focus, animation, or state. Do not recreate the full outer overlay for every internal view.

## Keep the Host Appropriately Scoped

Prefer a host owned by the feature or related flow. Do not turn it into a universal application overlay framework merely because several features use dialogs or drawers.

Share only genuinely common outer behavior. If related overlays have materially different interaction semantics, accessibility roles, positioning, animation, or lifecycle, separate hosts may be clearer than a heavily configurable host.

For a single simple overlay with no duplicated shell or content transitions, keep a normal local implementation. Do not introduce a host abstraction just because it is possible.

## State and Content Coordination

Changing content does not automatically require a store, context, router, or state machine. Prefer the project's simplest existing mechanism when local state, inputs, or direct composition remain readable.

Use feature-scoped shared state only when multiple content views coordinate meaningful state or that state must survive content transitions. Keep it instance-scoped when the framework and project support that pattern so separate overlay instances do not leak state into each other. Choose the project's established provider, dependency-injection, store, or ownership mechanism rather than prescribing a particular library.

Do not move application-wide state into the Overlay Host merely to make the overlay self-contained.

## Boundaries and Abstractions

Use separate units for meaningful content views or responsibilities. Do not build one large implementation dominated by conditional branches, but do not extract tiny pieces solely to reduce line count.

Keep feature-specific logic near the content that uses it. Extract a utility only when it is reusable, represents a meaningful domain concept, substantially improves readability, or follows an established project abstraction. Prefer obvious code over clever generalization.

## Implementation Behavior

After inspecting the code:

1. Identify the duplicated outer shell and the content-specific differences.
2. Briefly explain the host boundary when it is not obvious.
3. Implement one shared host and preserve focused content ownership.
4. Avoid unrelated abstractions or framework changes.
5. Validate opening and closing, content transitions, state retention or reset, keyboard and focus behavior, responsive layout, and instance isolation where relevant.

This is a refactoring and ownership pattern, not a rigid template. Use it when it removes real shell duplication and makes the feature easier to understand.
