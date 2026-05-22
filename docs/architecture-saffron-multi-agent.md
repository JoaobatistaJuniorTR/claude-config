# Agente Saffron — Multi-Agent Architecture

## Missao

Implementar interfaces com **Saffron Design System** em **Angular 19**, a partir de **Figma**, **imagens**, **descricoes textuais** ou **codigo existente**, usando uma arquitetura de 6 agentes especializados.

## Quando ativar

Aplica este modo de trabalho quando a tarefa envolver: Saffron, Figma, Thomson Reuters UI, `saf-*`, `@saffron/core-components`, `@saffron/core-styles`, design tokens, ou migracao de HTML/Angular para componentes Saffron.

## Agentes (`.claude/agents/`)

| Agente | Funcao | Quando |
|---|---|---|
| `saffron-orchestrator` | Coordena pipeline, detecta cenario, delega | Sempre — ponto de entrada |
| `saffron-figma-designer` | Analisa Figma via MCP, produz Blueprint | URL Figma fornecida |
| `saffron-visual-designer` | Analisa imagens/texto, produz Blueprint | Imagem ou descricao textual |
| `saffron-specialist` | Especialista Saffron, produz Saffron Guide | Consultado por designers e developer |
| `saffron-angular-developer` | Implementa codigo Angular | Apos Blueprint + Guide prontos |
| `saffron-auditor` | Valida A1-A10, corrige | Apos codigo gerado |

## Pipeline

```
Usuario → Orchestrator → Designer → Specialist → Developer → Auditor → Entrega
```

## Skills (`.claude/skills/`)

| Skill | Usada por |
|---|---|
| `saffron-patterns` | specialist, developer |
| `saffron-angular-integration` | specialist, developer |
| `saffron-tokens` | specialist, auditor |
| `saffron-a11y` | specialist, auditor |
| `saffron-wijmo-grid` | developer |
| `saffron-screen-from-figma` | figma-designer |
| `saffron-screen-from-image` | visual-designer |
| `saffron-migration` | developer |
| `saffron-prompt-to-design` | visual-designer |

## Skills globais (`~/.claude/skills/`)

| Skill | Descricao |
|---|---|
| `saffron-figma-angular` | Fluxo Figma → Angular |
| `saffron-angular-custom-elements` | CUSTOM_ELEMENTS_SCHEMA, templates, slots |
| `saffron-mcp-saffron` | Ferramentas MCP Saffron |
| `saffron-design-tokens` | Mapeamento CSS → tokens |
| `saffron-accessibility` | ARIA, WCAG com Saffron |
| `saffron-design-system-source` | Navegacao no repo fonte |

## MCP Saffron (saffron-mcp-server)

- `get-saffron-code` — snippets Angular (`framework: "angular"`)
- `get-saffron-tokens` — mapear valores CSS → tokens
- `get-saffron-a11y-attributes` — acessibilidade por componente
- `get-saffron-code-equivalent` — HTML → equivalentes saf-*
- `get-saffron-image` — capturas visuais

## Como usar

1. Invoque o `saffron-orchestrator` com sua tarefa
2. O orchestrator detecta o cenario e despacha os agentes adequados
3. Valide os blueprints e codigo nas pausas oferecidas
4. Receba o codigo final auditado (A1-A10)
