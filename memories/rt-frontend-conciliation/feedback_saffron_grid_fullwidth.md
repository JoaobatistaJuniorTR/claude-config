---
name: Saffron grid - filtros devem ocupar largura total
description: Ao implementar linhas de filtros (selects + botões) com saf-layout-grid, os campos devem se estender até o final da página (12 colunas totais), não ficar comprimidos no centro.
type: feedback
---

Quando implementar formulários de filtro com saf-layout-grid no taxonert_frontend, os selects e botões devem ocupar toda a largura disponível (12 colunas).

**Why:** O design do Figma mostra os filtros se estendendo de ponta a ponta dentro do container. Usar md="4" para cada item (4+4+4) deixa os campos estreitos demais. O padrão correto é dar mais espaço aos selects e menos aos botões.

**How to apply:** Para uma linha com 2 selects + botões, usar md="5" + md="5" + md="2" = 12 colunas. Os botões devem ficar com `justify-content="end"` para alinhar à direita. Exemplo:
- Select Empresa: `xs="12" md="5"`
- Select Estabelecimento: `xs="12" md="5"`
- Botões (Limpar/Pesquisar): `xs="12" md="2"` com `justify-content="end"`
