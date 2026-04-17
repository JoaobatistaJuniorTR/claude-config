---
name: saffron-mcp-saffron
description: >-
  Operates the user-saffron-mcp-server tools for Saffron Design System: code
  snippets, token lookup, accessibility specs, HTML-to-Saffron mapping, and
  screenshots. Use when implementing Saffron UIs, when MCP is configured, or when
  the user asks for Saffron APIs, tokens, or component mappings.
---

# Saffron MCP server (Cursor: user-saffron-mcp-server)

Always read the tool schema in the MCP descriptor before calling if parameters differ in your environment.

## Default for Angular work

For **`get-saffron-code`**, **`get-saffron-code-equivalent`**, and **`get-saffron-a11y-attributes`** (when a framework is relevant), pass **`framework: "angular"`** unless the user explicitly wants another target.

## Tools

### get-saffron-code

- **Purpose**: Production-oriented snippets (imports, usage) for named Saffron components.
- **Arguments**: `components` (array of strings, e.g. `["SafButton", "SafDialog"]`), `framework` (`"angular"` | `"react"`).
- **Tip**: Component names in the array use **PascalCase** `Saf*` as required; generated Angular markup uses **kebab-case** tags.

### get-saffron-tokens

- **Purpose**: Map **CSS values** (colors, spacing, fonts, etc.) to Saffron token names.
- **Arguments**: `cssValues` (array of strings, e.g. `["#1a1a1a", "16px", "Source Sans 3"]`).
- **Framework**: Not applicable.

### get-saffron-a11y-attributes

- **Purpose**: Accessibility attributes, ARIA patterns, and options per component.
- **Arguments**: `components` (array). For Angular, prefer **kebab-case** names like `saf-button`. Use `framework: "angular"` when supported for naming context.

### get-saffron-code-equivalent

- **Purpose**: Map **HTML elements** (`button`, `input`, …) to Saffron component names for the target framework.
- **Arguments**: `components` (HTML tag names), `framework` (`"angular"` for `saf-*` style names).

### get-saffron-image

- **Purpose**: Screenshot a live URL for a specific element.
- **Arguments**: `uri` (full page URL), `id` (CSS selector or element id).
- **Use**: Visual comparison with Figma or regression notes; requires a running Storybook or app.

## Suggested call order (implementation task)

1. **get-saffron-code-equivalent** — if starting from generic HTML or Figma “native” widgets.
2. **get-saffron-code** — canonical Angular usage for chosen components.
3. **get-saffron-tokens** — replace literals from design inspect.
4. **get-saffron-a11y-attributes** — finalize labels, roles, keyboard-related attributes.

## Related skills

- [saffron-figma-angular](../saffron-figma-angular/SKILL.md)
- [saffron-angular-custom-elements](../saffron-angular-custom-elements/SKILL.md)
