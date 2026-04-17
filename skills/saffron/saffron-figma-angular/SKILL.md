---
name: saffron-figma-angular
description: >-
  Implements Saffron UIs in Angular from Figma or design specs. Guides component
  choice, MCP usage (get-saffron-code, tokens, a11y, HTML equivalents), and
  alignment with Storybook. Use when the user mentions Figma, Saffron, design
  specs, or building screens with saf-* components in Angular.
---

# Saffron: Figma → Angular implementation

## Goal

Turn frames and component instances in Figma into **Angular** templates and styles that use **Saffron** (`saf-*` custom elements) and **design tokens**, not ad-hoc CSS.

## Workflow

1. **Identify building blocks**  
   Match Figma components to Saffron primitives (buttons, fields, dialogs, cards, etc.). If the design shows plain HTML patterns (`button`, `input`, `select`), call **get-saffron-code-equivalent** with `framework: "angular"` to get the `saf-*` tag names.

2. **Pull canonical usage**  
   Call **get-saffron-code** with `components: ["SafButton", ...]` (PascalCase names as required by the tool) and **`framework: "angular"`**. Use the returned snippets as the source of truth for imports, attributes, and patterns.

3. **Map visuals to tokens**  
   For colors, spacing, font families/sizes, and shadows from inspect/CSS, call **get-saffron-tokens** with those raw values. Prefer `var(--saf-...)` (or token-backed properties) in SCSS over literals.

4. **Lock in accessibility**  
   Call **get-saffron-a11y-attributes** with kebab-case component tags (e.g. `saf-text-field`) and `framework: "angular"` when the tool accepts it. Apply labels, roles, and keyboard behavior per the tool output and Saffron docs.

5. **Implement in Angular**  
   Follow **saffron-angular-custom-elements**: `CUSTOM_ELEMENTS_SCHEMA`, kebab-case tags, correct property vs attribute binding, custom events, slots.

6. **Verify against Storybook**  
   When the `saffron_design_system` repo is available locally, locate the component’s stories under `core-packages/core-components` for variants, states, and API edge cases that Figma may not show.

## Figma-specific tips

- **Instance properties** in Figma often map to **attributes/properties** on `saf-*` elements (e.g. appearance, size, disabled). Prefer Storybook/docs naming over guessing.
- **Auto layout gap/padding**: map to **spacing tokens**, not arbitrary `px`.
- **Icons**: Saffron uses Font Awesome (sharp); global icons in Figma may omit labels—still follow accessibility rules (aria-label where needed). See icon README in core-components.
- **Unknown composite**: decompose into atomic Saffron components; avoid raw HTML that duplicates a `saf-*` component.

## Optional visual check

If a running Storybook or app URL is available, **get-saffron-image** can capture a selector for comparison with Figma (requires `uri` and `id`).

## Do not

- Ship hardcoded hex/spacing when **get-saffron-tokens** returns a match.
- Skip a11y attributes for interactive or informational components.
- Use React-only patterns (JSX wrappers, `className`); use Angular templates and `class` / `ngClass`.

## See also

- [saffron-mcp-saffron](../saffron-mcp-saffron/SKILL.md) — MCP tool reference
- [saffron-angular-custom-elements](../saffron-angular-custom-elements/SKILL.md) — Angular integration details
- [saffron-design-tokens](../saffron-design-tokens/SKILL.md) — token workflow
- [saffron-accessibility](../saffron-accessibility/SKILL.md) — WCAG-focused usage
