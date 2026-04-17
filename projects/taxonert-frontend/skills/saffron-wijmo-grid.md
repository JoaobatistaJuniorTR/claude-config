# Saffron Wijmo FlexGrid

Implement data grids using Wijmo FlexGrid with Saffron styling in Angular. This skill contains the complete, production-validated patterns extracted from `saldo-credor-posterior` and `historico-sessoes` — the project's reference implementations.

## When to use

When the screen requires a data table/grid with sorting, filtering, pagination, selection, or large datasets.

---

## CRITICAL: ViewEncapsulation

Wijmo FlexGrid requires `ViewEncapsulation.None` for Saffron CSS to reach Wijmo internal elements. Without this, all grid styling silently fails.

```typescript
import { ViewEncapsulation } from '@angular/core';

@Component({
  // ...
  encapsulation: ViewEncapsulation.None,
})
```

---

## Required Imports

```typescript
// Core Wijmo
import { CollectionView, DataType } from '@mescius/wijmo';

// Grid core — import what you need
import {
  FlexGrid,
  AllowSorting,
  AllowResizing,
  AllowMerging,  // only if using merged headers
  Row,           // only if using merged headers
  GridPanel,
  CellType,
} from '@mescius/wijmo.grid';

// Filtering
import { FlexGridFilter } from '@mescius/wijmo.grid.filter';

// Selection (only if using checkbox selection)
import { Selector } from '@mescius/wijmo.grid.selector';

// Angular modules — add to component imports array
import { WjGridModule } from '@mescius/wijmo.angular2.grid';
import { WjGridFilterModule } from '@mescius/wijmo.angular2.grid.filter';
```

Component setup:
```typescript
@Component({
  standalone: true,
  imports: [CommonModule, WjGridModule, WjGridFilterModule],
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
  encapsulation: ViewEncapsulation.None,
})
```

---

## DataSource Pattern

```typescript
const DEFAULT_PAGE_SIZE = 15;

// Declaration
itemsSource = new CollectionView<MyModel>([], {
  pageSize: DEFAULT_PAGE_SIZE,
});

// Loading data from service (async)
private loadData(): void {
  this.myService.getData().subscribe((data) => {
    this.itemsSource.sourceCollection = data;
    this.itemsSource.refresh();
    this.itemsSource.moveToFirstPage();
  });
}

// Loading mock data (sync, in constructor)
constructor() {
  this.itemsSource = new CollectionView<MyModel>(this.mockData, {
    pageSize: DEFAULT_PAGE_SIZE,
  });
}
```

---

## Grid Initialization

```typescript
@ViewChild('gridRef', { static: true }) gridRef!: FlexGrid;
@ViewChild('filterRef', { static: true }) filterRef!: FlexGridFilter;

// Track state
pageIndex = 0;
isCompact = false;

initGrid(grid: FlexGrid): void {
  grid.autoGenerateColumns = false;
  grid.allowResizing = AllowResizing.Columns;
  grid.allowSorting = AllowSorting.SingleColumn;
  grid.selectionMode = 0; // None

  // Saffronize filter icons on layout update
  grid.updatedLayout.addHandler(() => {
    this.saffronizeFilterIcon(grid);
  });

  // Update item count on data load
  grid.loadedRows.addHandler(() => {
    this.updateItemCount();
  });

  // Keyboard shortcut: Alt+Arrow to open column filter
  grid.hostElement.parentElement?.addEventListener('keydown', (e) => {
    if (
      !e.ctrlKey && !e.metaKey && !e.shiftKey &&
      e.altKey &&
      (e.key === 'ArrowUp' || e.key === 'ArrowDown')
    ) {
      e.stopImmediatePropagation();
      if (grid.selection && this.filterRef) {
        this.filterRef.editColumnFilter(grid.columns[grid.selection.col]);
      }
    }
  });
}
```

---

## Template Structure (Complete Reference)

```html
<!-- Grid title for a11y -->
<span id="table-header" class="grid-mb-3">
  <saf-text font="heading-6">Titulo da Tabela</saf-text>
</span>
<span id="table-description" class="sr-only">Descricao para acessibilidade</span>

<!-- Toolbar -->
<saf-toolbar>
  <div slot="top-row-start">
    <saf-button appearance="primary">
      <saf-icon slot="start" icon-name="plus" appearance="light"></saf-icon>
      Adicionar
    </saf-button>
  </div>
  <div slot="top-row-end">
    <saf-search-field
      [value]="filter"
      (input)="onSearch($event)"
      (clear)="onSearch($event)"
    ></saf-search-field>
  </div>
  <div slot="bottom-row-start">
    <saf-checkbox
      [checked]="allSelected"
      [indeterminate]="someSelected"
      (change)="selectAll()"
    >
      Selecionar todos
    </saf-checkbox>
    {{ selectedCount }} selecionados
  </div>
  <div slot="bottom-row-end">
    <span>({{ itemCount }})</span>
  </div>
</saf-toolbar>

<!-- Grid -->
<div class="table-wrapper">
  <wj-flex-grid
    #gridRef
    [itemsSource]="itemsSource"
    selectionMode="CellRange"
    [showMarquee]="true"
    [anchorCursor]="true"
    [isReadOnly]="true"
    headersVisibility="Column"
    [alternatingRowStep]="1"
    (initialized)="initGrid(gridRef)"
    aria-labelledby="table-header"
    aria-describedby="table-description"
  >
    <!-- Standard data column with sort icon -->
    <wj-flex-grid-column binding="nome" header="Nome" width="*">
      <ng-template wjFlexGridCellTemplate [cellType]="'ColumnHeader'" let-col="col">
        <div style="display: flex; justify-content: space-between; width: 100%;">
          <span>{{ col.header }}</span>
          <span [ngStyle]="{
            'display': 'inline-block',
            'height': isCompact ? '24px' : '32px',
            'width': isCompact ? '24px' : '32px',
            'text-align': 'center',
            'align-content': 'center',
            'margin-left': '16px',
            'margin-right': '4px'}">
            <saf-icon
              [iconName]="col.currentSort === '+' ? 'arrow-up' : col.currentSort === '-' ? 'arrow-down' : 'arrow-up-arrow-down'"
              appearance="light">
            </saf-icon>
          </span>
        </div>
      </ng-template>
    </wj-flex-grid-column>

    <!-- Action column (icon-only, no sorting) -->
    <wj-flex-grid-column [width]="40" [isReadOnly]="true" [allowSorting]="false" header=" ">
      <ng-template wjFlexGridCellTemplate [cellType]="'Cell'" let-item="item">
        <saf-button size="small" appearance="ghost"
          a11y-aria-label="Abrir detalhes"
          (click)="abrirDetalhes(item)">
          <saf-icon icon-name="chevron-right" appearance="light"></saf-icon>
        </saf-button>
      </ng-template>
    </wj-flex-grid-column>

    <!-- Filter -->
    <wj-flex-grid-filter #filterRef
      [filterColumns]="['nome', 'tipo', 'status']">
    </wj-flex-grid-filter>
  </wj-flex-grid>
</div>

<!-- Pagination (OUTSIDE grid wrapper) -->
<saf-pagination
  [totalItemCount]="itemsSource.totalItemCount"
  [itemsPerPage]="itemsSource.pageSize"
  [currentPageIndex]="itemsSource.pageIndex + 1"
  (change)="setPageIndex($event)"
  (items-per-page-change)="setItemsPerPage($event)"
  items-label="itens"
></saf-pagination>
```

### Key template attributes:
- `headersVisibility="Column"` (NOT "All" — project standard)
- `[anchorCursor]="true"` (always include)
- `[alternatingRowStep]="1"` (alternating row colors)
- `[cellType]="'ColumnHeader'"` (binding with quotes — NOT `cellType="ColumnHeader"`)
- `width="*"` (fluid width for data columns)
- Action columns: `[width]="40" [isReadOnly]="true" [allowSorting]="false" header=" "`

---

## Column Width Patterns

| Pattern | Usage |
|---|---|
| `width="*"` | Fluid — fills remaining space (most data columns) |
| `[width]="120"` | Fixed pixel width |
| `[width]="40"` | Icon-only action columns |
| `[minWidth]="100"` | Minimum width with resize |

---

## Merged Headers (Multi-Row Headers)

For grouped column headers like "Dados processados" spanning multiple columns:

```typescript
import { AllowMerging, Row } from '@mescius/wijmo.grid';

// Constants for merge range
const DADOS_START_COL = 6;
const DADOS_END_COL = 9;

initGrid(grid: FlexGrid): void {
  // ... standard init ...

  // Enable column header merging
  grid.allowMerging = AllowMerging.ColumnHeaders;

  // Insert a new header row at position 0
  const headerRow = new Row();
  headerRow.allowMerging = true;
  grid.columnHeaders.rows.insert(0, headerRow);

  // Set merged header text for grouped columns
  for (let c = 0; c < grid.columns.length; c++) {
    if (c >= DADOS_START_COL && c <= DADOS_END_COL) {
      grid.columnHeaders.setCellData(0, c, 'Dados processados');
    } else {
      // Copy original header to row 0 for non-merged columns
      const originalHeader = grid.columnHeaders.getCellData(1, c, false);
      grid.columnHeaders.setCellData(0, c, originalHeader);
    }
  }

  // CRITICAL: refresh after header modification
  setTimeout(() => grid.refresh(), 50);
}
```

---

## Pagination Wiring

CRITICAL: `saf-pagination` uses **1-based** indexing, `CollectionView.pageIndex` uses **0-based**.

```typescript
setPageIndex(event: any): void {
  const index = (event.detail || event) - 1;  // fallback for different event types
  this.itemsSource.moveToPage(index);
  this.pageIndex = index;  // track state
}

setItemsPerPage(event: any): void {
  const size = event.detail || event;
  this.itemsSource.pageSize = size;
  this.itemsSource.moveToFirstPage();  // CRITICAL: reset to page 0
}
```

---

## saffronizeFilterIcon — Complete Implementation

Replace Wijmo's default filter icons with Saffron icons and add keyboard accessibility:

```typescript
private saffronizeFilterIcon(grid: FlexGrid): void {
  const panel = grid.columnHeaders;
  const columns = panel.columns;

  columns.forEach((column: any) => {
    const cell = panel.getCellElement(0, column.index);
    if (!cell) return;

    const filterButton = cell.querySelector<HTMLButtonElement>(
      '.wj-btn.wj-btn-glyph.wj-elem-filter'
    );

    // Keyboard accessibility for header cells
    cell.addEventListener('keydown', (e: KeyboardEvent) => {
      if (e.key === 'Tab') {
        e.stopImmediatePropagation();
      } else if (e.key === 'Enter' && e.altKey && filterButton) {
        e.preventDefault();
        filterButton.click();
        // Focus first input in filter dialog
        setTimeout(() => {
          const filterDialog = document.querySelector('.wj-form-control');
          const firstFocusable = filterDialog?.querySelector(
            'input, button, select, textarea'
          );
          if (firstFocusable) {
            (firstFocusable as HTMLElement).focus();
          }
        }, 100);
      } else if (e.key === 'Enter' && filterButton) {
        cell.click();
        e.stopImmediatePropagation();
      }
    });

    // Focus management for filter button
    if (filterButton) {
      cell.addEventListener('focusin', () => {
        filterButton.tabIndex = 0;
      });
      cell.addEventListener('focusout', () => {
        filterButton.tabIndex = -1;
      });
      filterButton.addEventListener('keydown', (e: KeyboardEvent) => {
        if (e.key === 'Enter') {
          filterButton.click();
          e.stopImmediatePropagation();
        }
      });
      // Move button to end of cell for proper tab order
      filterButton.parentElement!.appendChild(filterButton);
    }
  });
}
```

---

## SCSS Styling — Complete Reference

These are the exact styles used in the project's production grids. Copy and adapt as needed.

```scss
// === GRID CONTAINER ===

.table-wrapper {
  .wj-flexgrid {
    border: 1px solid var(--saf-color-border-stronger);
    overflow: hidden;
  }
}

// === HEADER CELLS ===

.wj-flexgrid .wj-header {
  background-color: var(--saf-flexgrid-color-background-header, var(--saf-color-background-subtle));
  color: var(--saf-color-text-heavy);
  font: var(--saf-type-body-default-md-strong-standard);
  padding: var(--saf-spacing-2) var(--saf-spacing-4);
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-bottom: 1px solid var(--saf-color-border-strong);
}

// === DATA CELLS ===

.wj-flexgrid .wj-cell {
  font: var(--saf-type-body-default-md-regular-standard);
  color: var(--saf-color-text-heavy);
  padding: var(--saf-spacing-2) var(--saf-spacing-4);
  display: flex;
  align-items: center;
  border-right: 1px solid var(--saf-color-border-default);
}

// === ALTERNATING ROWS ===

.wj-flexgrid .wj-row.wj-alt:not(.wj-state-selected) {
  background-color: var(--saf-color-background-subtle);
}

// === HOVER STATE ===

.wj-flexgrid .wj-cell:hover {
  background: var(--saf-color-background-hover);
}

// === SELECTED STATE ===

.wj-flexgrid .wj-cell.wj-state-selected {
  background-color: var(--saf-color-interactive-background-selected);
  color: var(--saf-color-interactive-on-selected);
}

// === ACTIVE/FOCUS STATE ===

.wj-flexgrid .wj-cell.wj-state-active {
  border: 2px solid var(--saf-color-interactive-focus);
  outline: none;
}

// === COMPACT MODE ===
// Wrap grid in a div with class saf-flexgrid--compact
// Use CSS custom property for density:
// <div style="--saf-density: 0;">

// === PAGINATION SPACING ===

saf-pagination {
  margin-top: var(--saf-spacing-3);
}
```

### Token Reference for Grid Styles

| Property | Token | Fallback |
|---|---|---|
| Grid border | `var(--saf-color-border-stronger)` | — |
| Header background | `var(--saf-flexgrid-color-background-header)` | `var(--saf-color-background-subtle)` |
| Header text | `var(--saf-color-text-heavy)` | — |
| Header font | `var(--saf-type-body-default-md-strong-standard)` | — |
| Header border | `var(--saf-color-border-strong)` | — |
| Cell text | `var(--saf-color-text-heavy)` | — |
| Cell font | `var(--saf-type-body-default-md-regular-standard)` | — |
| Cell border | `var(--saf-color-border-default)` | — |
| Cell padding | `var(--saf-spacing-2) var(--saf-spacing-4)` | — |
| Alt row background | `var(--saf-color-background-subtle)` | — |
| Hover background | `var(--saf-color-background-hover)` | — |
| Selected background | `var(--saf-color-interactive-background-selected)` | — |
| Selected text | `var(--saf-color-interactive-on-selected)` | — |
| Focus border | `var(--saf-color-interactive-focus)` | — |

---

## Selection Pattern (Checkbox)

Only use when the grid needs row selection:

```typescript
@ViewChild('gridRef', { static: true }) gridRef!: FlexGrid;

initGrid(grid: FlexGrid): void {
  // ... standard init ...

  new Selector(grid, {
    showCheckAll: false,
    itemChecked: () => this.updateSelection(),
  });
}

private updateSelection(): void {
  const selected = this.itemsSource.items.filter((item: any) => item.$isSelected);
  this.selectedCount.set(selected.length);
}
```

---

## Density / Compact Mode

```html
<!-- Normal density -->
<div style="--saf-density: 0;">
  <!-- toolbar + grid + pagination -->
</div>

<!-- Compact mode -->
<div class="saf-flexgrid--compact" style="--saf-density: 1;">
  <!-- toolbar + grid + pagination -->
</div>
```

IMPORTANT: `--saf-density` value must NOT be quoted. Use `style="--saf-density: 1"` not `style="--saf-density: '1'"`.

---

## Status Column Pattern

For columns showing status with colored text:

```html
<wj-flex-grid-column binding="status" header="Status" [width]="120">
  <ng-template wjFlexGridCellTemplate [cellType]="'Cell'" let-item="item">
    <span [class]="'status--' + item.status">
      {{ item.statusLabel }}
    </span>
  </ng-template>
</wj-flex-grid-column>
```

```scss
.status--concluida {
  color: var(--saf-color-status-success-strong);
}
.status--em_execucao {
  color: var(--saf-color-status-info-strong);
}
.status--pausada {
  color: var(--saf-color-status-warning-strong);
}
.status--erro {
  color: var(--saf-color-status-error-strong);
}
```

---

## Accessibility

- `aria-labelledby` pointing to a heading describing the table
- `aria-describedby` for additional context (sr-only span)
- `headersVisibility="Column"` (project standard)
- Keyboard: Alt+Arrow opens column filter, Tab navigates, Enter activates
- Filter button keyboard accessible via saffronizeFilterIcon

---

## Anti-Patterns (NEVER do)

- Use `headersVisibility="All"` (project uses `"Column"`)
- Hardcode hex colors in grid styles (e.g. `border: #D2D2D2 1px solid`)
- Skip `ViewEncapsulation.None` (grid styles won't work)
- Use `cellType="ColumnHeader"` without binding (must be `[cellType]="'ColumnHeader'"`)
- Skip `[alternatingRowStep]="1"` (no alternating rows)
- Forget `moveToFirstPage()` after changing items per page
- Quote `--saf-density` value (`'1'` instead of `1`)
- Register Saffron components in constructor instead of main.ts
- Use `headersFocusability="All"` (not used in project)

---

## Reference Files

- **Best practice**: `features/apuracao/components/saldo-credor-posterior/` (all 3 files)
- **Grid with merged headers**: `features/conciliacao/components/historico-sessoes/` (all 3 files)
- **Kitchen-sink**: `C:\repositories\saffron_design_system\core-packages\core-components\src\components\wijmo\kitchen-sink\kitchen-sink.angular.ts`
- **Saffron Wijmo docs**: `C:\repositories\saffron_design_system\core-packages\core-components\src\components\wijmo\overview.doc.mdx`
- **Mescius API**: https://developer.mescius.com/wijmo/api/
