# Saffron Screen From Figma

Create Angular screens from Figma designs using Saffron Design System.

## When to use

When the user provides a Figma link, mentions Figma, or wants to create a screen based on a Figma design.

---

## Flow

### Step 1 — Get Figma Data

Try MCP Figma tools first:
- `get_file` — get full file structure
- `get_node` — get specific node/frame
- `get_selection` — get currently selected element

If MCP Figma fails → ask user for a screenshot and switch to **@saffron-screen-from-image** skill.

### Step 2 — Analyze Figma Structure

From Figma data, identify:
- **Component hierarchy**: frames, groups, auto-layout directions
- **Components**: Figma component instances and their properties
- **Styles**: fill colors, stroke colors, spacing (padding, gap), typography (font, size, weight)
- **Layout**: auto-layout direction, spacing, alignment

### Step 3 — Map to Saffron

1. **HTML -> Saffron**: Call `get-saffron-code-equivalent` with identified elements
   ```
   get-saffron-code-equivalent({ components: ["button", "input", "select", "card"], framework: "angular" })
   ```

2. **Angular snippets**: Call `get-saffron-code` for each mapped component
   ```
   get-saffron-code({ components: ["SafButton", "SafTextField", "SafCard"], framework: "angular" })
   ```

3. **Tokens**: Call `get-saffron-tokens` with Figma color/spacing values
   ```
   get-saffron-tokens({ cssValues: ["#0066CC", "#F5F5F5", "8px", "16px", "600"] })
   ```

4. **Accessibility**: Call `get-saffron-a11y-attributes` for interactive components
   ```
   get-saffron-a11y-attributes({ components: ["saf-button", "saf-text-field"], framework: "angular" })
   ```

5. **MCP fallback**: If MCP returns empty/error, search `saffron_design_system` repo. Save findings to `saffron-learnings.md`.

### Step 4 — Generate Template (PAUSE)

Generate `.component.html` and `.component.scss`:

- Respect Figma visual hierarchy in the template structure
- Use `saf-layout-grid` with breakpoints (`xs`, `md`, `lg`)
- Use only `var(--saf-*)` tokens in SCSS — zero hardcoded values
- Follow template pattern from **@saffron-patterns**
- Follow integration rules from **@saffron-angular-integration**
- If table/grid detected → use **@saffron-wijmo-grid** skill

**PAUSE** — Ask user to validate the template.

### Step 5 — Generate Logic (PAUSE)

After template approval, generate `.component.ts`:

- Standalone component with `CUSTOM_ELEMENTS_SCHEMA`
- `ChangeDetectionStrategy.OnPush`
- `signal<T>()` for local state
- `inject()` for DI
- `input()` for inputs (never `@Input()`)
- `computed()` for derived values
- Register new `saf-*` components in `main.ts`

**PAUSE** — Ask user to validate the logic.

### Step 6 — Audit

Run the automatic post-generation audit (10 checks, A1-A10). See agent for full checklist.

---

## Figma-Specific Rules

- If Figma uses a component **without** a Saffron equivalent → warn user and suggest closest alternative
- **Never** use Figma-exported class names (`.frame-5841`, `.group-42`, `.apples-43`)
- Prioritize Saffron components over custom HTML
- If table/grid in design → invoke @saffron-wijmo-grid
- Maintain Figma visual fidelity while using Saffron components
- Figma auto-layout `horizontal` → `saf-layout-grid` row or flexbox
- Figma auto-layout `vertical` → stacked grid items or flex-column

---

## Figma Property Mapping

| Figma Property | Saffron Equivalent |
|---|---|
| Fill color | `get-saffron-tokens` → `var(--saf-color-*)` |
| Stroke color | `get-saffron-tokens` → `var(--saf-color-border-*)` |
| Padding | `get-saffron-tokens` → `var(--saf-spacing-*)` |
| Gap (auto-layout) | `spacing` attribute on `saf-layout-grid` or `var(--saf-spacing-*)` |
| Font family | Part of typography token `var(--saf-type-*)` |
| Font size | Part of typography token — never set independently |
| Font weight | Use `-strong-` or `-regular-` variant of typography token |
| Border radius | `var(--saf-border-radius-*)` |

---

## Rules

- Follow all patterns from @saffron-patterns
- Follow integration rules from @saffron-angular-integration
- Use tokens from @saffron-tokens — never hardcode
- Apply accessibility from @saffron-a11y
- If user says "gera tudo completo" → skip pauses but NEVER skip audit
