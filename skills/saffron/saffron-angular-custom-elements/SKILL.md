---
name: saffron-angular-custom-elements
description: >-
  Embeds Saffron FAST web components (saf-*) in Angular: CUSTOM_ELEMENTS_SCHEMA,
  kebab-case tags, property and event binding, slots, and styling with tokens and
  ::part. Use when implementing or reviewing Angular code that uses @saffron/core-components.
---

# Saffron web components in Angular

Saffron is implemented as **custom elements** (`saf-*`). Angular treats them as unknown elements unless you declare schema support.

## Module or standalone component

Import and register:

```typescript
import { CUSTOM_ELEMENTS_SCHEMA } from '@angular/core';

// @NgModule({ ... schemas: [CUSTOM_ELEMENTS_SCHEMA] })
// or
@Component({
  // ...
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
})
```

Use **`schemas: [CUSTOM_ELEMENTS_SCHEMA]`** on the NgModule or standalone component that declares templates containing `saf-*` tags.

## Templates

- Use **kebab-case** tags: `<saf-button>`, `<saf-text-field>`, not PascalCase.
- **Slots**: put slotted children on light-DOM elements with the matching `slot` attribute, e.g. `slot="start"`, `slot="end"`, per component docs and [COMMON-API](https://github.com/tr/saffron_design_system/blob/main/docs/COMMON-API.md) slot names (`start`, `end`, default, etc.).

## Properties vs attributes

- Many Saffron APIs are **element properties**. Prefer Angular **property binding** when the value is non-string or updated dynamically: `[appearance]="'primary'"`, `[disabled]="isDisabled"`.
- Use **`attr.*`** when you intentionally set HTML attributes only or when the component reflects via attributes and you need string values.
- If a control does not update as expected, check Storybook or source for whether the API is attribute-reflected or property-only.

## Events

Listen for **DOM custom events** the component dispatches (e.g. `change`, `input`, `close`). In templates:

```html
<saf-text-field (change)="onChange($event)"></saf-text-field>
```

Use `$event` or `$event.detail` depending on what the component emits (verify in docs or Storybook).

## Forms

Form-associated custom elements integrate with `<form>` when supported. Prefer Saffron form components (`saf-text-field`, etc.) for validated patterns; wire **ControlValueAccessor** wrappers only when the project already uses that pattern.

## Styles

- Prefer **design tokens** (`var(--saf-...)`) in component SCSS.
- **Shadow DOM**: global styles do not pierce component internals. Use documented **`::part(...)`** selectors only where the component exposes parts (see design system docs on parts vs slots).
- Do not rely on **ViewEncapsulation** to style inside Saffron’s shadow tree.

## Packages and global styles

- Install **`@saffron/core-components`** and **`@saffron/core-styles`** per the version your app uses; follow [saffron_angular-examples](https://github.com/tr/saffron_angular-examples) for version-specific setup.
- Register **Font Awesome** CSS from `@saffron/core-styles` in `angular.json` styles when using `saf-icon` (see icon component README in the design system).

## File layout (demo / feature parity)

For feature or demo pages mirroring design-system DAT conventions:

- `src/app/.../saf-my-feature-demo.component.ts`
- `saf-my-feature-demo.component.html`
- `saf-my-feature-demo.component.scss`

Use strict TypeScript, scoped SCSS, and demonstrate variants/states when building from Figma.

## Debugging

- If Angular reports unknown element `saf-*`, add **`CUSTOM_ELEMENTS_SCHEMA`** to the declaring component/module.
- If bindings seem inert, confirm custom element **upgrade** (bundle imports components before route render) and check property vs attribute.
