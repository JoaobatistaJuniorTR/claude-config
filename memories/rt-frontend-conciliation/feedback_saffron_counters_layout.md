---
name: saffron-counters-layout
description: Contadores numéricos lado a lado devem usar flex:1 com texto centralizado, números heading-3xl e labels body-default-lg
type: feedback
---

Ao implementar contadores/resumos numéricos lado a lado (ex: "5 Total de sessões | 2 Concluídas | 2 Em execução | 1 Pausadas"), usar:

- Números grandes com `appearance="heading-3xl"` (não heading-2xl)
- Labels com `appearance="body-default-lg"` (não body-default-sm)
- Cada bloco com `flex: 1` para ocupar largura igual
- Conteúdo centralizado (`align-items: center; justify-content: center`)
- Padding generoso em cada bloco (`padding: var(--saf-spacing-4) var(--saf-spacing-3)`)
- Separados por `saf-divider orientation="vertical"` com `align-self: stretch`

**Why:** A primeira versão usava heading-2xl, body-default-sm e gap fixo sem flex:1, resultando em contadores pequenos e desproporcionais comparados ao Figma. O design espera blocos espaçosos e equilibrados.

**How to apply:** Sempre que implementar cards com contadores/KPIs numéricos do Figma, priorizar números grandes (heading-3xl+) e blocos de largura igual com centralização.
