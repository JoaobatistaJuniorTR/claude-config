---
name: Wijmo FlexGrid readonly background fix
description: isReadOnly=true causa fundo cinza em todas as celulas porque Saffron _gauge.css aplica background cinza via aria-readonly. Usar isReadOnly=false + beginningEdit.cancel + override global no styles.scss.
type: feedback
---

Nunca usar `[isReadOnly]="true"` em Wijmo FlexGrid com Saffron. Isso faz todas as celulas ficarem com fundo cinza uniforme, sem alternancia de cores.

**Why:** Saffron `_gauge.css` tem a regra `.wj-cell[aria-readonly=true] { background: var(--saf-color-interactive-read-only-subtle) }` que resolve para `#e5e5e5` (cinza). Quando `isReadOnly=true`, Wijmo adiciona `aria-readonly="true"` em TODAS as celulas, e essa regra sobrescreve os backgrounds de `.wj-cell.wj-alt` (alternating rows) e `.wj-cell` (normal rows).

**How to apply:**

1. **No template HTML**: usar `[isReadOnly]="false"` (como debitos-contribuinte-saffron faz)
2. **No initGrid()**: cancelar edicao programaticamente:
   ```typescript
   grid.beginningEdit.addHandler((s: FlexGrid, e: any) => {
     e.cancel = true;
   });
   ```
3. **Override global ja adicionado** em `styles.scss` (linhas 20-31) que corrige o fundo para grids que usam readonly:
   ```scss
   .wj-cell[aria-readonly=true] { background: var(--saf-color-interactive-background-default) !important; }
   .wj-cell.wj-alt[aria-readonly=true] { background: var(--saf-flexgrid-color-background-alt-rows) !important; }
   .wj-cell.wj-header[aria-readonly=true] { background: var(--saf-flexgrid-color-background-header) !important; }
   ```

**Tokens envolvidos:**
- `--saf-color-interactive-background-default` → branco (celulas normais)
- `--saf-flexgrid-color-background-alt-rows` → `#f2f6f9` (linhas alternadas)
- `--saf-flexgrid-color-background-header` → `#dae4ed` (cabecalho)
- `--saf-color-interactive-read-only-subtle` → `#e5e5e5` (o cinza problematico)

**Referencia de grid funcional:** `debitos-contribuinte-saffron` (usa `[isReadOnly]="false"`)
