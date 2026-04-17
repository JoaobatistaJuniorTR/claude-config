---
name: feedback_layout_saffron
description: Como usar saf-layout-grid corretamente e padrões de layout Saffron aprendidos no painel-gerenciamento
type: feedback
---

Usar `saf-layout-grid` + `saf-layout-grid-item` para layouts de página. A primeira tentativa falhou, mas a segunda (baseada nos exemplos existentes) funcionou perfeitamente.

**Why:** Na primeira tentativa, usei `md="0"` para dividers e `grid-d-flex grid-align-baseline` direto no grid-item, quebrando o layout. Na segunda tentativa, segui os padrões de `painel-inconsistencia` e `apuracao` que já estavam no projeto, e tudo funcionou.

**How to apply — padrões que funcionam:**

1. **Alinhamento via props nativas do saf-layout-grid:**
   - `justify-content="space-between"` para item esquerda + item direita
   - `justify-content="end"` para alinhar à direita
   - `justify-content="center"` para centralizar
   - NUNCA usar classes CSS (`grid-d-flex grid-justify-end`) no grid-item para alinhamento — o shadow DOM impede. Usar a prop `justify-content` no `saf-layout-grid`.

2. **Space-between (item esquerda + item direita):**
   - `<saf-layout-grid justify-content="space-between">` com dois grid-items sem xs fixo
   - Alternativa: `md="6"` + `md="6"`, lado direito com nested `<saf-layout-grid justify-content="end">` e `<saf-layout-grid-item xs="fit">`
   - Para textos inline na mesma linha, usar `div.inline-baseline` com `display: flex; align-items: baseline; flex-wrap: nowrap` DENTRO do grid-item

3. **3 colunas iguais:**
   - `md="4"` cada (padrão dos tributo-cards em `painel-inconsistencia`)
   - `spacing="3"` no grid para gap entre colunas

4. **Dividers verticais entre colunas:**
   - NÃO usar grid-item separado para dividers (consome colunas e desalinha)
   - Usar `border-left: 1px solid var(--saf-color-border-subtle)` com `padding-left` para equalizar espaçamento
   - No responsivo (`max-width: 960px`): trocar para `border-top` + `padding-top`

5. **Evitar `saf-container` dentro de `saf-tab-panel`** — adiciona padding extra indesejado

6. **Flexbox ainda é válido** para conteúdo interno de cards (detalhes-header, valores, legendas) — o grid é para a estrutura da página/seções
