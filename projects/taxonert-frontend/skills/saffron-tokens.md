# Saffron Design Tokens

Map visual values (colors, spacing, typography) to Saffron design tokens.

## When to use

When translating Figma styles, replacing hardcoded CSS values, or migrating custom CSS to Saffron tokens.

---

## MCP Tool Usage

Always call `get-saffron-tokens` with concrete CSS values:

```
get-saffron-tokens({ cssValues: ["#333333", "16px", "600", "Source Sans 3"] })
```

---

## Token Categories

### Colors (`--saf-color-*`)

| Category | Examples |
|---|---|
| Text | `--saf-color-text-default`, `--saf-color-text-subtle`, `--saf-color-text-disabled` |
| Interactive | `--saf-color-interactive-primary-default`, `--saf-color-interactive-secondary-default`, `--saf-color-interactive-tertiary-default` |
| Status | `--saf-color-status-error-strong`, `--saf-color-status-success-strong`, `--saf-color-status-warning-strong`, `--saf-color-status-info-strong` |
| Border | `--saf-color-border-default`, `--saf-color-border-strong` |
| Background | `--saf-color-background-default`, `--saf-color-background-subtle` |
| Neutral | `--saf-color-neutral-*` (numbered scale) |

### Spacing (`--saf-spacing-*`)

| Token | Value |
|---|---|
| `--saf-spacing-1` | 4px |
| `--saf-spacing-2` | 8px |
| `--saf-spacing-3` | 12px |
| `--saf-spacing-4` | 16px |
| `--saf-spacing-5` | 24px |
| `--saf-spacing-6` | 40px |
| `--saf-spacing-7` | 48px |
| `--saf-spacing-8` | 64px |
| `--saf-spacing-9` | 80px |
| `--saf-spacing-10` | 96px |

### Typography (`--saf-type-*`)

- Body: `--saf-type-body-default-sm-regular-standard`, `--saf-type-body-default-md-strong-standard`, `--saf-type-body-default-lg-regular-standard`
- Heading: `--saf-type-heading-default-lg-strong-standard`, `--saf-type-heading-default-xl-strong-standard`, `--saf-type-heading-default-2xl-strong-standard`
- Usage: `font: var(--saf-type-body-default-md-regular-standard);`

### Border Radius (`--saf-border-radius-*`)

`--saf-border-radius-sm`, `--saf-border-radius-md`, `--saf-border-radius-lg`

### FlexGrid (`--saf-flexgrid-*`)

`--saf-flexgrid-color-background-header` — override at `:root` level for customization.

---

## Rules

1. Correct prefix is `--saf-*` — NEVER `--saffron-*` or Figma names like `--type-heading-default-2xl-*`
2. Prefer semantic tokens over core tokens (`--saf-color-text-default` over `--saf-color-neutral-900`)
3. Never invent tokens that don't exist
4. If no token matches, use closest match and add SCSS comment: `/* No exact token — closest match */`
5. TypeScript token usage (D3 charts):
   ```typescript
   const style = getComputedStyle(this.elementRef.nativeElement);
   const color = style.getPropertyValue('--saf-color-status-error-strong').trim();
   ```

---

## Common Mappings

| Hardcoded Value | Saffron Token |
|---|---|
| `#333`, `#212223` | `var(--saf-color-text-default)` |
| `#666`, `#666666` | `var(--saf-color-text-subtle)` |
| `#999` | `var(--saf-color-text-disabled)` |
| `#D2D2D2`, `#e5e5e5` | `var(--saf-color-border-default)` |
| `#f8f9fa` | `var(--saf-color-background-subtle)` |
| `16px` (margin/padding) | `var(--saf-spacing-4)` |
| `24px` (margin/padding) | `var(--saf-spacing-5)` |
| `40px` (margin/padding) | `var(--saf-spacing-6)` |
| `font-weight: 600` | Use `--saf-type-*-strong-*` variant |
| `font-size: 14px` | Part of typography token, don't set independently |
