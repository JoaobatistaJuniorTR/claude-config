---
name: saffron-design-system-source
description: >-
  Locates Saffron Design System source in the saffron_design_system monorepo:
  FAST components, Storybook stories, COMMON-API, and docs. Use when the MCP
  output is insufficient, when verifying slots/events/parts, or when aligning
  Figma variants with story variants.
---

# Navigating saffron_design_system

## High-signal paths

- **Components**: `core-packages/core-components/src/components/<name>/` — templates (`*.template.ts`), definitions, specs (`*.spec.md`).
- **Stories**: co-located or under Storybook config in the same package; search for `<name>.stories.ts` when implementing a specific `saf-*` tag.
- **Shared API conventions**: `docs/COMMON-API.md` — slot names (`start`, `end`, default), `::part` names (`root`, `control`, `label`, …), common custom events (`change`, `input`, `close`, …).
- **Architecture**: `docs/FAST-COMPONENT-PATTERNS.md` — how FAST/Saffron composes web components and shadow DOM.
- **Web components mental model**: `docs/WEB-COMPONENTS-FAQ.md` — shadow DOM vs Angular encapsulation, events.

## How to use with Figma

1. Resolve the **Saffron component** name from the Figma instance or MCP **get-saffron-code-equivalent**.
2. Open the component folder and **story file** to list **appearances, sizes, states**; mirror those in Angular templates with the same attributes/properties.
3. If documentation in README is **GENERATED**, still trust **stories + source** for edge cases.

## Build note (local full build)

From repo root: `yarn build` — **core-styles** must build before **core-components** (per GETTING-STARTED).

## Related

- [saffron-figma-angular](../saffron-figma-angular/SKILL.md)
- [saffron-angular-custom-elements](../saffron-angular-custom-elements/SKILL.md)
