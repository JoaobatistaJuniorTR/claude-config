# Saffron Patterns

Validated project patterns for taxonert_frontend. Reference component: `saldo-credor-posterior`.

## When to use

This skill is loaded automatically by the saffron-screen-builder agent. Consult it for Angular component structure, template patterns, SCSS patterns, and project conventions.

---

## Component Pattern

```typescript
import { Component, ChangeDetectionStrategy, CUSTOM_ELEMENTS_SCHEMA, signal, computed, inject, input } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'rt-nome-componente',
  standalone: true,
  schemas: [CUSTOM_ELEMENTS_SCHEMA],
  imports: [CommonModule],
  templateUrl: './nome-componente.component.html',
  styleUrl: './nome-componente.component.scss',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class NomeComponenteComponent {
  // Signals para estado local
  isLoading = signal<boolean>(true);
  data = signal<Tipo | null>(null);

  // Inject para DI (nunca constructor injection)
  private service = inject(NomeService);

  // Signal inputs (nunca @Input() decorator)
  id = input.required<string>();

  // Computed values
  total = computed(() => this.data()?.items.length ?? 0);
}
```

---

## Template Pattern

```html
<saf-container>
  <saf-layout-grid spacing="6">
    <saf-layout-grid-item xs="12" md="6" lg="4">
      <saf-card>
        <saf-text appearance="heading-lg">Titulo</saf-text>
        <saf-divider role="separator"></saf-divider>
        <!-- conteudo -->
      </saf-card>
    </saf-layout-grid-item>
  </saf-layout-grid>
</saf-container>
```

Sempre usar breakpoints `xs`, `md`, e `lg` no `saf-layout-grid-item` para responsividade.

---

## SCSS Pattern

```scss
:host {
  display: block;
}

.section-header {
  margin-bottom: var(--saf-spacing-4);
}

.field-label {
  font: var(--saf-type-body-default-sm-regular-standard);
  color: var(--saf-color-text-subtle);
}

.value-text {
  font: var(--saf-type-body-default-md-strong-standard);
  color: var(--saf-color-text-default);
}

// Responsivo via saf-layout-grid, NAO via media queries custom
// Usar saf-layout-grid-item com xs, md, lg
```

---

## Data Formatting

```typescript
formatCurrency(value: number): string {
  return value.toLocaleString('pt-BR', { style: 'currency', currency: 'BRL' });
}

formatCNPJ(cnpj: string): string {
  return cnpj.replace(/^(\d{2})(\d{3})(\d{3})(\d{4})(\d{2})$/, '$1.$2.$3/$4-$5');
}

formatDate(date: string): string {
  return new Date(date).toLocaleDateString('pt-BR');
}
```

---

## Feature Folder Structure

```
features/
  nome-feature/
    nome-feature.component.ts          # Componente principal (container)
    nome-feature.component.html
    nome-feature.component.scss
    nome-feature.routes.ts             # Lazy loading routes
    components/                        # Sub-componentes
      nome-table/
      nome-filters/
      nome-detail/
    models/                            # Interfaces e types
      nome-feature.model.ts
    services/                          # Services especificos
      nome-feature.service.ts
```

---

## main.ts Registration

```typescript
// SEMPRE registrar globalmente no main.ts
import { SafButton, SafCard, SafIcon, SafDialog } from '@saffron/core-components';
SafButton();
SafCard();
SafIcon();
SafDialog();
// NUNCA registrar no constructor de componentes individuais
```

Antes de adicionar, verificar se ja esta registrado para evitar duplicatas.

---

## Breadcrumb Pattern

```typescript
// Na rota
{ path: 'exemplo', component: ExemploComponent, data: { breadcrumb: 'Exemplo' } }
```

---

## Utility Classes

```
grid-mb-{n}          margin-bottom com var(--saf-spacing-{n})
grid-mt-{n}          margin-top com var(--saf-spacing-{n})
grid-ml-{n}          margin-left com var(--saf-spacing-{n})
grid-flex             display: flex
grid-flex-column      flex-direction: column
grid-justify-between  justify-content: space-between
grid-align-center     align-items: center
grid-gap-{n}          gap com var(--saf-spacing-{n})
```

**Bug conhecido:** `.grid-ml-6` em `styles.scss` usa `--saf-spacing-8` (64px) ao inves de `--saf-spacing-6` (40px). NAO propagar este erro.

---

## Rules

- `shared/` somente para componentes reutilizaveis sem dependencia de services de feature
- Sempre usar `takeUntilDestroyed()` ou `toSignal()` para subscriptions — nunca `.subscribe()` sem cleanup
- D3/Charts: ler tokens via `getComputedStyle(el).getPropertyValue('--saf-color-*').trim()`

---

## Anti-Patterns (NUNCA fazer)

- Hardcodar cores hex em SCSS ou TypeScript
- Usar `NO_ERRORS_SCHEMA` (sempre `CUSTOM_ELEMENTS_SCHEMA`)
- Criar CSS framework custom (`.btn`, `.card` custom)
- Usar emojis como icones (usar `saf-icon`)
- Inline styles com valores hardcoded
- `!important` em estilos de componentes Saffron
- Registrar componentes Saffron em constructors de componentes
- Usar `@Input()` decorator (usar `input()` signal)
- Usar `confirm()` ou `alert()` nativos (usar `saf-dialog`)
- Classes com nomes do Figma (`.frame-5841`, `.apples-43`)
- `.subscribe()` sem cleanup
- Componentes especificos de feature em `shared/`
- `import *` (exceto Wijmo que exige)
- Tokens com prefixo errado (`--saffron-*` ao inves de `--saf-*`)
