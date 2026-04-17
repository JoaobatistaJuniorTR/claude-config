---
name: Wijmo FlexGrid merged headers
description: Como criar headers mergeados (grupo de colunas com titulo) no Wijmo FlexGrid usando MergeManager customizado + header row extra programatico.
type: feedback
---

Para criar headers de grupo (ex: "Dados processados" cobrindo 4 sub-colunas) no Wijmo FlexGrid:

**Why:** Templates customizados de ColumnHeader (`wjFlexGridCellTemplate [cellType]="'ColumnHeader'"`) impedem o `AllowMerging` nativo. `AllowMerging.ColumnHeaders` so faz merge horizontal, nao vertical. A unica forma confiavel e um `MergeManager` customizado.

**How to apply:**

1. **Remover ColumnHeader templates** das colunas que participam do merge
2. **Criar MergeManager customizado**:
   ```typescript
   class CustomMergeManager extends MergeManager {
     override getMergedRange(panel: any, r: number, c: number, clip = true): CellRange | null {
       if (panel.cellType === CellType.ColumnHeader) {
         // Merge vertical: colunas que ocupam 2 linhas
         if (c <= 5) return new CellRange(0, c, 1, c);
         // Merge horizontal: grupo na linha 0
         if (r === 0 && c >= 6 && c <= 9) return new CellRange(0, 6, 0, 9);
       }
       return null;
     }
   }
   ```
3. **No initGrid**, adicionar header row extra e setar dados:
   ```typescript
   const panel = grid.columnHeaders;
   panel.rows.insert(0, new Row());
   // Colunas normais: header na linha 0
   for (let c = 0; c <= 5; c++) {
     panel.setCellData(0, c, grid.columns[c].header || '');
   }
   // Grupo: mesmo texto na linha 0 para merge horizontal
   for (let c = 6; c <= 9; c++) {
     panel.setCellData(0, c, 'Dados processados');
   }
   grid.mergeManager = new CustomMergeManager();
   grid.allowMerging = AllowMerging.ColumnHeaders;
   ```
4. **Centralizar header do grupo** via `formatItem`:
   ```typescript
   grid.formatItem.addHandler((s, e) => {
     if (e.panel.cellType === CellType.ColumnHeader && e.row === 0 && e.col >= 6) {
       e.cell.style.textAlign = 'center';
     }
   });
   ```

**Imports necessarios:** `AllowMerging, Row, CellRange, CellType, MergeManager` de `@mescius/wijmo.grid`

**IMPORTANTE:** Metodo `getMergedRange` precisa do modificador `override`.

**Distribuicao de largura:** Colunas com conteudo fixo (datas, periodos) usar `[width]` fixo. Colunas restantes usar `width="*"` com `[minWidth]` para distribuir igualmente e preencher toda a largura do grid.
