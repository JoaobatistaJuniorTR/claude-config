# Saffron Screen From Image

Create Angular screens from screenshots, mockups, or images using Saffron Design System.

## When to use

When the user provides an image (screenshot, print do Figma, mockup, wireframe) and wants to generate an Angular component from it.

---

## Flow

### Step 1 — Analyze the Image

Read the image and identify:
- **Layout structure**: columns, rows, sections, cards, containers
- **UI components**: buttons, inputs, selects, tables, tabs, icons, text blocks, checkboxes, switches
- **Visual properties**: colors, spacing, typography, borders, shadows
- **Hierarchy**: header, content areas, sidebars, footers

### Step 2 — Confirm Interpretation

**ALWAYS** describe what you identified to the user before generating code:

```
Interpretacao da imagem:
- Layout: 2 colunas (4/8) dentro de um card
- Componentes: 3 text-fields, 1 select, 2 buttons (primary + secondary)
- Tabela: 5 colunas com paginacao
- Cores: fundo cinza claro, texto escuro, botao azul primario
Confirma? Quer ajustar algo?
```

If image quality is low, ask for a better image or text description.

### Step 3 — Map to Saffron

1. **HTML -> Saffron**: Call `get-saffron-code-equivalent` with identified HTML elements
   ```
   get-saffron-code-equivalent({ components: ["button", "input", "select", "table"], framework: "angular" })
   ```

2. **Angular snippets**: Call `get-saffron-code` for each mapped component
   ```
   get-saffron-code({ components: ["SafButton", "SafTextField", "SafSelect"], framework: "angular" })
   ```

3. **Tokens**: Call `get-saffron-tokens` with extracted visual values
   ```
   get-saffron-tokens({ cssValues: ["#333333", "#f8f9fa", "16px", "24px"] })
   ```

4. **Accessibility**: Call `get-saffron-a11y-attributes` for interactive components
   ```
   get-saffron-a11y-attributes({ components: ["saf-button", "saf-text-field", "saf-select"], framework: "angular" })
   ```

5. **MCP fallback**: If MCP returns empty/error, search `saffron_design_system` repo. Save findings to `saffron-learnings.md`.

### Step 4 — Generate Template (PAUSE)

Generate `.component.html` and `.component.scss`:

- Use `saf-layout-grid` with breakpoints (`xs`, `md`, `lg`) for responsive layout
- Use only `var(--saf-*)` tokens in SCSS — zero hardcoded values
- Follow template pattern from **@saffron-patterns**
- Follow integration rules from **@saffron-angular-integration**
- If table/grid detected → use **@saffron-wijmo-grid** skill

**PAUSE** — Ask user to validate the template before continuing.

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

## Grid Inference

Map visual layout to 12-column Saffron grid:

| Visual Layout | Grid Mapping |
|---|---|
| Full width | `xs="12"` |
| Two equal columns | `xs="12" md="6"` |
| Sidebar + content | `xs="12" md="4"` + `xs="12" md="8"` |
| Three columns | `xs="12" md="6" lg="4"` |
| Four columns | `xs="12" md="6" lg="3"` |

---

## Rules

- **Always** confirm visual interpretation before generating code
- If image quality is low → ask for better image or text description
- Infer 12-column grid from visual layout
- If table/grid detected → invoke @saffron-wijmo-grid
- Follow all patterns from @saffron-patterns
- Follow integration rules from @saffron-angular-integration
- Use tokens from @saffron-tokens — never hardcode
- Apply accessibility from @saffron-a11y
- If user says "gera tudo completo" → skip pauses but NEVER skip audit
