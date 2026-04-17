# Saffron Accessibility

WCAG-oriented accessibility for Saffron components in Angular.

## When to use

When building or reviewing forms, dialogs, interactive controls, data grids, or when the user mentions a11y, ARIA, or screen readers.

---

## MCP Tool Usage

```
get-saffron-a11y-attributes({ components: ["saf-button", "saf-text-field", "saf-dialog"], framework: "angular" })
```

---

## Core Rules

### Every interactive control needs an accessible name

- Visible label (preferred)
- `aria-label` attribute
- `aria-labelledby` pointing to a visible element's ID

### Icon Buttons (icon-only)

```html
<saf-button appearance="secondary" icon-only a11y-aria-label="Export data">
  <saf-icon icon-name="arrow-up-from-bracket" presentation></saf-icon>
</saf-button>
```

- `a11y-aria-label` on the button (Saffron custom attribute)
- `presentation` attribute on the icon (removes from a11y tree)

### Decorative Icons

```html
<saf-icon icon-name="chevron-right" aria-hidden="true"></saf-icon>
```

### Icons with Meaning (not in a button)

```html
<saf-icon icon-name="warning" a11y-aria-label="Alerta: valor excede o limite"></saf-icon>
```

### Never use emoji as icons

Use `<saf-icon icon-name="...">` instead.

### Never use color alone

Always add text, icon, or pattern alongside color to convey meaning (colorblind users).

---

## Keyboard Interaction Requirements

- All interactive elements: focusable with visible focus indicator
- Buttons/links: Enter and Space to activate
- Dialogs/drawers/tooltips: Escape to close
- Tab: navigate between focusable elements
- Arrow keys: navigate within tabs, radio groups, menus
- Focus trap: dialogs must trap focus within themselves

---

## Component-Specific Patterns

### saf-anchor

```html
<!-- Navigation link -->
<saf-anchor href="/details/123">View Details</saf-anchor>

<!-- Click handler — MUST have href -->
<saf-anchor href="javascript:void(0)" (click)="navigate()">View Details</saf-anchor>
```

NEVER use `<saf-anchor (click)="...">` without `href`.

### saf-select

```html
<saf-select aria-label="Selecionar empresa" (change)="onCompanyChange($event)">
  <saf-option value="1">Empresa A</saf-option>
</saf-select>
```

### saf-dialog

- Must have a title (via heading slot or `aria-label`)
- Must have a close button
- Focus trap is built-in by Saffron

### saf-divider

```html
<saf-divider role="separator"></saf-divider>
```

### wj-flex-grid (Wijmo)

```html
<wj-flex-grid
  aria-labelledby="tableTitle"
  aria-describedby="tableDescription"
>
```

### Charts / SVG (D3)

```html
<div role="img" aria-label="Grafico gauge mostrando 75% debito vs 25% credito">
  <!-- D3 SVG content -->
</div>
```

---

## Prefer Saffron Built-in Semantics

Saffron components already provide correct roles and ARIA attributes. Only add manual ARIA when:

- Saffron doesn't cover the specific use case
- Wrapping non-Saffron elements (D3 charts, custom HTML)
- Adding descriptions that Saffron can't infer
