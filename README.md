# Claude Code - Custom Configuration Backup

Backup completo de toda a configuracao customizada do Claude Code, organizada por categoria.

> **IMPORTANTE**: O `settings.json` foi sanitizado — tokens (`ADO_PAT`, `ADO_MCP_AUTH_TOKEN`) foram substituidos por placeholders. Nunca commite secrets reais.

---

## Estrutura do Repositorio

```
claude-config/
├── agents/                          # Agentes globais (~/.claude/agents/)
├── skills/                          # Skills globais (~/.claude/skills/)
│   └── saffron/                     # Skills Saffron (subpastas)
├── config/                          # Configuracao global
│   ├── CLAUDE.md                    # Instrucoes globais para todos os projetos
│   └── settings.json                # Settings sanitizado (sem secrets)
├── memories/                        # Memorias por projeto
│   ├── taxonert-frontend/
│   ├── rt-frontend-conciliation/
│   ├── rt-agents-conciliation/
│   └── taxone-ecosystem-mcp/
└── projects/                        # Agents e skills de projetos especificos
    ├── taxonert-frontend/
    │   ├── agents/                  # 7 agentes Saffron multi-agent
    │   ├── skills/                  # 14 skills do projeto
    │   └── CLAUDE.md
    ├── rt-agents-conciliation/
    │   └── CLAUDE.md
    └── taxone-ecosystem-mcp/
        └── skills/write-story/
```

---

## Agentes

### Globais (`agents/`)

| Agente | Arquivo | Descricao |
|--------|---------|-----------|
| **Sincerao** | `sincerao.md` | Critico brutal que valida ideias, codigo e planos com honestidade afiada, humor seco e analise estrategica. Opera em loop iterativo (max 7 rodadas) ate emitir veredito: APROVADO, PRECISA DE AJUSTE, ou REPROVADO. Ativado automaticamente em brainstorming ou via `/sincerao`. |
| **Saffron Template** | `AGENTS-SAFFRON-TEMPLATE.md` | Template de arquitetura multi-agente (6 agentes) para implementar UIs com Saffron Design System em Angular 19. Pipeline: orchestrator -> designer -> specialist -> developer -> auditor. |

### Projeto: taxonert-frontend (`projects/taxonert-frontend/agents/`)

| Agente | Arquivo | Funcao |
|--------|---------|--------|
| **saffron-orchestrator** | `saffron-orchestrator.md` | Coordena pipeline, detecta cenario, delega para agentes adequados |
| **saffron-figma-designer** | `saffron-figma-designer.md` | Analisa Figma via MCP, produz Blueprint de implementacao |
| **saffron-visual-designer** | `saffron-visual-designer.md` | Analisa imagens/texto, produz Blueprint visual |
| **saffron-specialist** | `saffron-specialist.md` | Especialista Saffron, produz Saffron Guide com componentes e tokens |
| **saffron-angular-developer** | `saffron-angular-developer.md` | Implementa codigo Angular com Saffron apos Blueprint + Guide |
| **saffron-auditor** | `saffron-auditor.md` | Valida checklist A1-A10 de qualidade e corrige |
| **angular-auditor** | `angular-auditor.md` | Auditor geral Angular (fora do pipeline Saffron) |

---

## Skills

### Globais (`skills/`)

| Skill | Arquivo | Invocavel? | Descricao |
|-------|---------|-----------|-----------|
| **ado-client** | `ado-client.md` | Nao (utilitaria) | Referencia REST API do Azure DevOps — curl patterns para work items (fetch, create, update, comment, attachments). Usada por outras skills como `/taxone-fix`. |
| **taxone-fix** | `taxone-fix.md` | Sim (`/taxone-fix {id}`) | Workflow completo de fix TaxOne: le issue ADO -> cria branch `fix/MFS{id}/...` a partir de `rc` -> bump versao -> implementa -> testa -> commit com `AB#{id}` -> abre PR dev + PR rc (draft) -> resolve conflitos com branch `-conflict`. |
| **sincerao** | `sincerao.md` | Sim (`/sincerao`) | Wrapper que spawna o agente Sincerao com contexto da conversa atual e executa loop iterativo. |

### Globais Saffron (`skills/saffron/`)

| Skill | Pasta | Descricao |
|-------|-------|-----------|
| **saffron-accessibility** | `saffron-accessibility/` | WCAG/ARIA para componentes Saffron — usa MCP `get-saffron-a11y-attributes` |
| **saffron-angular-custom-elements** | `saffron-angular-custom-elements/` | Web components FAST (`saf-*`) em Angular — CUSTOM_ELEMENTS_SCHEMA, kebab-case, property binding |
| **saffron-design-system-source** | `saffron-design-system-source/` | Navegar monorepo fonte do Saffron Design System |
| **saffron-design-tokens** | `saffron-design-tokens/` | Mapear CSS values -> tokens Saffron (`--saf-*`) via MCP |
| **saffron-figma-angular** | `saffron-figma-angular/` | Fluxo completo Figma -> Angular com Saffron |
| **saffron-mcp-saffron** | `saffron-mcp-saffron/` | Opera ferramentas MCP do Saffron (code, tokens, a11y, equivalents) |

### Projeto: taxonert-frontend (`projects/taxonert-frontend/skills/`)

| Skill | Arquivo | Descricao |
|-------|---------|-----------|
| **front-engineer** | `front-engineer.md` | Skill principal de engenharia frontend |
| **git-workflow** | `git-workflow.md` | Workflow Git do time (branches, PRs, conflitos) |
| **investigation** | `investigation.md` | Investigacao de bugs e issues |
| **bugfix** | `bugfix/SKILL.md` | Workflow de bugfix (usa ado-client) |
| **saffron-a11y** | `saffron-a11y.md` | Acessibilidade Saffron (projeto) |
| **saffron-angular-integration** | `saffron-angular-integration.md` | Integracao Angular + Saffron |
| **saffron-migration** | `saffron-migration.md` | Migracao HTML -> componentes Saffron |
| **saffron-patterns** | `saffron-patterns.md` | Padroes de uso Saffron |
| **saffron-prompt-to-design** | `saffron-prompt-to-design.md` | Gerar design a partir de prompt textual |
| **saffron-screen-from-figma** | `saffron-screen-from-figma.md` | Construir tela a partir de Figma |
| **saffron-screen-from-image** | `saffron-screen-from-image.md` | Construir tela a partir de imagem |
| **saffron-tokens** | `saffron-tokens.md` | Tokens Saffron (projeto) |
| **saffron-wijmo-grid** | `saffron-wijmo-grid.md` | Grids Wijmo com Saffron |

### Projeto: taxone-ecosystem-mcp (`projects/taxone-ecosystem-mcp/skills/`)

| Skill | Arquivo | Descricao |
|-------|---------|-----------|
| **write-story** | `write-story/skill.md` | Escrever historias ADO com impact analysis do ecossistema |

---

## MCP Servers

Servidores MCP configurados para integrar com ferramentas externas:

| Server | Proposito | Ferramentas Principais |
|--------|-----------|----------------------|
| **azure-devops** | Azure DevOps completo | Work items (CRUD, comments, attachments), pipelines (build, run, logs), wikis, repos, iterations, backlogs, capacities |
| **saffron-mcp-server** | Saffron Design System | `get-saffron-code` (snippets), `get-saffron-tokens` (CSS -> tokens), `get-saffron-a11y-attributes` (ARIA), `get-saffron-code-equivalent` (HTML -> saf-*), `get-saffron-image` (screenshots) |
| **SONAR_MCP** | SonarQube | Quality gate, metricas (coverage, ncloc, complexity), issues por severidade, scan de projetos |
| **taxone-ecosystem** | Ecossistema TaxOne | `get_dependency_graph`, `get_impact_analysis`, `get_service_details`, `search_services` — 46 componentes (34 servicos + 12 libs) |
| **tr-code-scan-mcp** | Security scanning | `scan_project` (vulnerabilidades), `optimize_fixes` (ILP para minimizar fixes), remediation knowledge base |

---

## Memorias

Memorias persistentes que o Claude usa para manter contexto entre conversas. Organizadas por projeto.

### taxonert-frontend (15 memorias)

Feedbacks sobre Saffron (cards, dialogs, layout, eventos), Wijmo grids (readonly, headers, sizing), pipeline orchestrator, e preferencias de componentes.

### rt-agents-conciliation (11 memorias)

Decisoes do MVP de conciliacao, regras de dados (inbox/conciliation), configs locais (Docker WSL, PostgreSQL porta 3005), e feedbacks criticos (NUNCA deletar documents_inbox).

### rt-frontend-conciliation (3 memorias)

Referencia ao repo principal (taxonert_frontend), layout de grid Saffron, e counters numericos.

### taxone-ecosystem-mcp (3 memorias)

Contexto do ecossistema (46 componentes), projeto ADO Story Writer, e preferencia de idioma PT-BR.

---

## Configuracao Global

### CLAUDE.md (`config/CLAUDE.md`)

Instrucoes que se aplicam a **todos** os projetos:

- **Estilo de PR**: Portugues brasileiro, informal, com humor tecnico. Estrutura obrigatoria com secoes tematicas (O que rolou, R.I.P., O que entrou, Commits, Como testar, Risco) e citacao ironica no final.
- **Sincerao automatico**: Ativado automaticamente em contextos de ideacao/brainstorming. Loop iterativo de ate 7 rodadas sem pedir permissao ao usuario.

### settings.json (`config/settings.json`)

- **Plugins**: Superpowers + Figma
- **Env vars**: ADO_PAT, ADO_ORG, ADO_PROJECT (tokens sanitizados)
- **Hooks**: Pixel Agents hooks em todos os eventos (SessionStart, SessionEnd, Stop, PermissionRequest, Notification, UserPromptSubmit, PreToolUse, PostToolUse, PostToolUseFailure, SubagentStart, SubagentStop)

---

## Como Restaurar

Para restaurar essa configuracao em uma nova maquina:

```bash
# 1. Copiar agents e skills globais
cp -r agents/* ~/.claude/agents/
cp -r skills/* ~/.claude/skills/

# 2. Copiar config global
cp config/CLAUDE.md ~/.claude/CLAUDE.md
cp config/settings.json ~/.claude/settings.json
# IMPORTANTE: editar settings.json e colocar seus tokens reais

# 3. Copiar memorias (ajustar paths dos projetos conforme necessario)
# Os paths de memoria dependem do diretorio do projeto no seu sistema
# Formato: ~/.claude/projects/{path-encoded}/memory/

# 4. Copiar agents e skills de projeto
cp -r projects/taxonert-frontend/agents/* /path/to/taxonert_frontend/.claude/agents/
cp -r projects/taxonert-frontend/skills/* /path/to/taxonert_frontend/.claude/skills/
# Repetir para outros projetos
```

---

> *"A memoria e a unica coisa que sobrevive ao `rm -rf` do tempo."* — Dev Anonimo
