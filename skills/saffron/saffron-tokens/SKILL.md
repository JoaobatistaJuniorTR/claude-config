---
name: saffron-tokens
description: >-
  Maps raw CSS values to Saffron design tokens using MCP and documents usage of
  CSS custom properties (--saf-*). Use when translating Figma styles, replacing
  hardcoded colors/spacing/typography, or migrating custom CSS to Saffron tokens.
---

# Saffron design tokens

## MCP: get-saffron-tokens

Pass **concrete values** from Figma inspect or existing CSS:

- Colors: hex, `rgb()`, `rgba()`
- Spacing: `px`, `rem` (e.g. `8px`, `1rem`)
- Typography: font family names, sizes, weights as strings
- Other: `none`, shadows, borders as they appear in CSS

Example argument shape: `cssValues: ["#000000", "12px", "Inter"]`

Use the tool’s mappings to pick the **canonical token**; prefer that token in code over the raw value.

## In stylesheets

- Prefer **`var(--saf-...)`** (or design-system-documented variables) in SCSS/CSS.
- For CSP-safe dynamic theming patterns, follow internal docs such as `docs/csp/` in **saffron_design_system** when applicable.

## With Figma

- Auto layout and spacing scales should **collapse to token steps**, not arbitrary pixels.
- When no token fits, document the exception; avoid silently inventing new “standard” spacing.

## Related

- [saffron-mcp-saffron](../saffron-mcp-saffron/SKILL.md) — tool reference
- [saffron-figma-angular](../saffron-figma-angular/SKILL.md) — full UI workflow
