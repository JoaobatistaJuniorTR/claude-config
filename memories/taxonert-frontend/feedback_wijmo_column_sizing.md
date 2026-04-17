---
name: Wijmo Column Sizing
description: Colunas de acao com headers curtos precisam de 85-95px minimo, especialmente com merged headers
type: feedback
---

Colunas de acao no Wijmo FlexGrid (ex: "Padrao", "Excluir", "Editar") precisam de largura minima de 85-95px, nao 64-72px como parece suficiente.

**Why:** Headers curtos parecem caber em 64-72px, mas com merged headers (row extra do MergeManager) e icones de filtro, o texto e cortado. Foram necessarias 3 iteracoes para acertar na tela de perfis.

**How to apply:** Ao definir `[width]` de colunas de acao com cell templates (radio, icone), usar minimo 85px. Se a coluna tiver merged header group acima, considerar 90-95px.
