# Saffron Migration

Migrate existing Angular screens to Saffron Design System.

## When to use

When the user asks to migrate an existing screen to Saffron, mentions "migrar para Saffron", or wants to replace Bootstrap/Material/custom CSS with Saffron components.

---

## Flow

### Step 1 — Read Existing Screen

Read all files of the existing screen:
- `.component.html` — template
- `.component.ts` — logic
- `.component.scss` — styles

Understand the current implementation before making changes.

### Step 2 — Inventory Elements

Create an inventory of:

**HTML elements used:**
```
<button>, <input>, <select>, <table>, <div class="card">, etc.
```

**CSS properties with hardcoded values:**
```
color: #333333
margin: 16px
font-size: 14px
background: #f8f9fa
border: 1px solid #e5e5e5
```

**Custom CSS classes:**
```
.btn-primary, .card-header, .form-control, etc.
```

### Step 3 — Map to Saffron

1. **HTML -> Saffron**: Call `get-saffron-code-equivalent`
   ```
   get-saffron-code-equivalent({ components: ["button", "input", "select", "table"], framework: "angular" })
   ```

2. **CSS -> Tokens**: Call `get-saffron-tokens`
   ```
   get-saffron-tokens({ cssValues: ["#333333", "#f8f9fa", "16px", "14px", "#e5e5e5"] })
   ```

3. **Angular snippets**: Call `get-saffron-code` for each mapped component
   ```
   get-saffron-code({ components: ["SafButton", "SafTextField", "SafSelect"], framework: "angular" })
   ```

4. **Accessibility**: Call `get-saffron-a11y-attributes`
   ```
   get-saffron-a11y-attributes({ components: ["saf-button", "saf-text-field"], framework: "angular" })
   ```

5. **MCP fallback**: If MCP returns empty/error, search `saffron_design_system` repo. Save findings to `saffron-learnings.md`.

### Step 4 — Generate Migrated Template (PAUSE)

Generate migrated `.component.html` and `.component.scss`:

- Replace HTML elements with Saffron equivalents
- Replace hardcoded CSS with `var(--saf-*)` tokens
- Preserve ALL existing data bindings, `*ngIf`, `*ngFor`, `(click)`, `[class]`, etc.
- Use `saf-layout-grid` for layout structure
- Follow patterns from **@saffron-patterns**

**Present changelog before asking for validation:**

```
CHANGELOG — [component-name] migration
========================================
REPLACED: <button class="btn btn-primary"> → <saf-button appearance="primary">
REPLACED: <input class="form-control"> → <saf-text-field>
REPLACED: color: #333 → var(--saf-color-text-default)
REPLACED: margin: 16px → var(--saf-spacing-4)
ADDED: CUSTOM_ELEMENTS_SCHEMA to decorator
ADDED: SafButton, SafTextField registration in main.ts
KEPT: All service calls and data bindings unchanged
KEPT: *ngIf="isLoading" preserved
REMOVED: .btn-primary custom class (replaced by saf-button appearance)
```

**PAUSE** — Ask user to validate the migrated template.

### Step 5 — Update Logic (PAUSE)

After template approval, update `.component.ts` **only if needed**:

- Add `CUSTOM_ELEMENTS_SCHEMA` to decorator
- Update event handlers if Saffron events have different signatures
  - Example: `(change)` now provides `$event.detail` instead of `$event.target.value`
- Register new `saf-*` components in `main.ts`

**Do NOT change:**
- Services, models, or routing
- Business logic
- Old Angular patterns (NgModules, Observables) unless user explicitly asks

**PAUSE** — Ask user to validate the logic changes.

### Step 6 — Audit

Run the automatic post-generation audit (10 checks, A1-A10). See agent for full checklist.

---

## Migration Rules

### Preserve Business Logic
- Migration is **visual only**, not functional
- Do NOT alter services, models, or routing
- Do NOT refactor old Angular patterns unless user explicitly asks
- Keep all existing data bindings intact

### Common Mappings

| Before | After |
|---|---|
| `<button class="btn btn-primary">` | `<saf-button appearance="primary">` |
| `<button class="btn btn-secondary">` | `<saf-button appearance="secondary">` |
| `<input class="form-control">` | `<saf-text-field>` |
| `<select class="form-select">` | `<saf-select>` |
| `<div class="card">` | `<saf-card>` |
| `<div class="modal">` | `<saf-dialog>` |
| `<table class="table">` | `wj-flex-grid` (use @saffron-wijmo-grid) |
| `<nav class="tabs">` | `<saf-tabs>` |
| `<input type="checkbox">` | `<saf-checkbox>` |
| `<div class="alert">` | `<saf-alert>` |
| `<span class="badge">` | `<saf-badge>` |
| `<div class="spinner">` | `<saf-spinner>` |

### Event Signature Changes

After migration, some events change signature. Update handlers accordingly:

| Before | After |
|---|---|
| `(change)="fn($event.target.value)"` on `<select>` | `(change)="fn($event.detail)"` on `<saf-select>` |
| `(change)="fn($event.target.checked)"` on `<input checkbox>` | `(change)="fn($event.target.checked)"` on `<saf-checkbox>` |
| `(input)="fn($event.target.value)"` on `<input>` | `(input)="fn($event.target.value)"` on `<saf-text-field>` |

---

## Rules

- Follow all patterns from @saffron-patterns
- Follow integration rules from @saffron-angular-integration
- Use tokens from @saffron-tokens — never hardcode
- Apply accessibility from @saffron-a11y
- If user says "gera tudo completo" → skip pauses but NEVER skip audit
- Always present changelog before asking for validation
