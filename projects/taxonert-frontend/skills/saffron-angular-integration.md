# Saffron Angular Integration

Reference for embedding Saffron FAST web components (saf-*) in Angular 19.

## When to use

Consult when implementing or reviewing Angular code that uses `@saffron/core-components`.

---

## CUSTOM_ELEMENTS_SCHEMA

Add to `schemas` array in every component decorator that uses saf-* tags:

```typescript
@Component({
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
  // ...
})
```

Required because Saffron components are web components, not Angular components. Without it, Angular throws unknown element errors.

---

## Property Binding

| Scenario | Syntax | Example |
|---|---|---|
| String literal (static) | No brackets | `appearance="primary"` |
| Dynamic value | Brackets | `[appearance]="myVar"` |
| Quoted string in binding | Brackets + quotes | `[appearance]="'primary'"` |
| Boolean (always true) | Just attribute | `disabled` |
| Boolean (dynamic) | Brackets | `[disabled]="isDisabled"` |

---

## Event Binding

Standard pattern: `(eventname)="handler($event)"`

Saffron custom events wrap payload in `$event.detail`.

### Common Events by Component

| Component | Event | Payload |
|---|---|---|
| `saf-select` | `(change)` | `$event.detail` — selected value |
| `saf-checkbox` | `(change)` | `$event.target.checked` — boolean |
| `saf-switch` | `(change)` | `$event.target.checked` — boolean |
| `saf-text-field` | `(input)` | `$event.target.value` — typing |
| `saf-text-field` | `(change)` | `$event.target.value` — on blur |
| `saf-dialog` | `(close)` | dialog closed |
| `saf-dialog` | `(cancel)` | dialog cancelled |
| `saf-drawer` | `(close)` | drawer closed |
| `saf-tabs` | `(change)` | `$event.detail` — active tab index |
| `saf-pagination` | `(change)` | `$event.detail` — page number (1-based) |
| `saf-pagination` | `(items-per-page-change)` | `$event.detail` — new page size |
| `saf-search-field` | `(input)` | `$event.target.value` — search text |
| `saf-search-field` | `(clear)` | search cleared |
| `saf-date-picker` | `(change)` | `$event.detail` — selected date |

---

## Slot Usage (per COMMON-API)

| Slot | Purpose | Used In |
|---|---|---|
| `slot="start"` | Content before main (icons, prefix) | button, anchor |
| `slot="end"` | Content after main (icons, suffix) | button, anchor |
| `slot="icon"` | Card icon | card |
| `slot="heading"` | Card heading | card |
| `slot="checked-message"` | Switch checked label | switch |
| `slot="unchecked-message"` | Switch unchecked label | switch |
| `slot="top-row-start"` | Toolbar top-left | toolbar |
| `slot="top-row-end"` | Toolbar top-right | toolbar |
| `slot="bottom-row-start"` | Toolbar bottom-left | toolbar |
| `slot="bottom-row-end"` | Toolbar bottom-right | toolbar |
| (default, no attribute) | Main content | all components |

### Example

```html
<saf-button appearance="primary">
  <saf-icon slot="start" icon-name="plus" appearance="light"></saf-icon>
  Add Item
</saf-button>

<saf-switch>
  <span slot="checked-message">Ativo</span>
  <span slot="unchecked-message">Inativo</span>
</saf-switch>
```

---

## Shadow DOM Styling

- Use `::part()` to style shadow DOM elements:
  - `::part(control)` — main interactive element
  - `::part(root)` — component wrapper
  - `::part(content)` — content area
  - `::part(label)` — label element
- Prefer design tokens `var(--saf-*)` over `::part` when possible
- **Never** use `!important` to override Saffron styles

---

## Component Registration (main.ts)

```typescript
import { SafButton, SafCard, SafIcon } from '@saffron/core-components';
SafButton();
SafCard();
SafIcon();
```

- Register **globally** in `main.ts`, NEVER in component constructors
- Check main.ts before adding to avoid duplicates
- Every `saf-*` tag used in templates MUST have its registration call in main.ts

---

## Standalone Component Setup

```typescript
import { Component, CUSTOM_ELEMENTS_SCHEMA } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'rt-my-component',
  standalone: true,
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
  imports: [CommonModule],
  templateUrl: './my-component.component.html',
  styleUrl: './my-component.component.scss'
})
export class MyComponent {}
```
