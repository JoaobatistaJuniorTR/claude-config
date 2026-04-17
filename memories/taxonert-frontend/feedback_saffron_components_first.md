---
name: feedback_saffron_components_first
description: Sempre verificar componentes e props Saffron disponíveis antes de criar soluções custom - usar MCP tools e grep no node_modules
type: feedback
---

Sempre verificar se existe um componente ou prop Saffron nativo antes de criar implementações custom (divs, CSS manual, classes utilitárias, etc).

**Why:** Criei accordion custom com divs quando existia `saf-accordion` + `saf-accordion-item` no pacote. Também usei `saf-disclosure` quando o correto era `saf-accordion`. Tentei alinhar grid-items com classes CSS (`grid-d-flex grid-justify-end`) e divs wrapper quando o `saf-layout-grid` já tinha prop nativa `justify-content="end"`. O shadow DOM impede que classes externas afetem o layout interno dos web components.

**How to apply:**
1. ANTES de qualquer implementação, carregar skills: `saffron-angular-custom-elements` e `saffron-mcp-saffron`
2. Seguir a ordem do MCP: `get-saffron-code-equivalent` → `get-saffron-code` → `get-saffron-tokens` → `get-saffron-a11y-attributes`
3. Sempre fazer `grep` nos `.d.ts` do `node_modules/@saffron/core-components` para confirmar existência real (MCP pode estar incompleto)
4. **Quando precisar de alinhamento/layout, verificar PRIMEIRO as props nativas do componente** (ex: `justify-content`, `spacing`, `orientation`) antes de usar classes CSS ou divs wrapper
5. Prioridade: prop nativa do componente > componente Saffron custom > CSS custom
6. Verificar o que já existe no `main.ts` registrado e o que precisa ser adicionado
7. Nunca criar divs/CSS custom para algo que o Saffron já fornece (accordion, disclosure, tabs, justify-content, etc.)
8. **REGRA CRITICA para ajustes visuais**: NUNCA tentar CSS custom para alinhar/posicionar componentes saf-*. SEMPRE consultar o saffron-specialist PRIMEIRO, mesmo para ajustes que pareçam simples. O specialist consulta o código fonte do Saffron e os stories oficiais para encontrar a solução correta. Exemplo: `align-self: center` no saf-layout-grid-item é padrão validado pelos stories oficiais, não CSS custom inventado.
