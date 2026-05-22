---
name: saffron-a11y
description: >-
  Applies WCAG-oriented accessibility for Saffron components using
  get-saffron-a11y-attributes and Saffron guidance (icons, labels, keyboard).
  Use when building or reviewing forms, dialogs, interactive controls, or when
  the user mentions a11y, ARIA, or screen readers with Saffron.
---

# Saffron accessibility

## MCP: get-saffron-a11y-attributes

- Pass **`components`** as the `saf-*` (kebab-case) or tool-supported names for each control in scope.
- When the tool supports **`framework`**, use **`"angular"`** for Angular projects.
- Apply returned attributes (e.g. `aria-label`, `role`, `aria-expanded`) in **Angular templates** on the `saf-*` element or slotted content as the spec indicates.

## General rules

- Every **interactive** control needs an **accessible name** (visible label, `aria-label`, or `aria-labelledby`).
- **Icons** without text: use Saffron/global icon guidance—global icons may omit visible labels but still need correct **`aria-hidden`** / **`aria-label`** / **`role`** per component docs.
- Prefer **native semantics** via Saffron components over divs with manual ARIA when a `saf-*` component exists for the pattern.
- Custom events from web components still require **keyboard** support as documented (focus, Enter/Space, escape for overlays, etc.).

## Verification

- Cross-check behavior and roles against **Storybook** stories for the component in **saffron_design_system**.
- For deeper test processes, see `docs/testing/` in the design system (screen reader, VoiceOver, etc.).

## Related

- [saffron-mcp-saffron](../saffron-mcp-saffron/SKILL.md)
- [saffron-figma-angular](../saffron-figma-angular/SKILL.md)
