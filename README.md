# Claude Code — Custom Configuration Backup

Backup completo da configuração customizada do Claude Code, organizada por categoria.

> **IMPORTANTE**: `settings.json` e `mcp-servers.json` foram sanitizados — tokens (`ADO_PAT`, `SONARQUBE_TOKEN`, webhooks Teams, etc) foram substituídos por placeholders. Nunca commite secrets reais.

---

## Estrutura do Repositório

```
claude-config/
├── agents/                          # Agentes globais (~/.claude/agents/)
│   ├── sincerao.md                  # Crítico brutal — valida ideias e código
│   ├── oraculo.md                   # Especialista Oracle Database
│   ├── slon.md                      # Especialista PostgreSQL
│   ├── oraculo-memory/              # Memória persistente do oráculo
│   └── slon-memory/                 # Memória persistente do slon
├── skills/                          # Skills globais (~/.claude/skills/)
│   ├── ado-client/SKILL.md          # REST API ADO (utilitária)
│   ├── close-release/SKILL.md       # Fechar release/hotfix
│   ├── create-hotfix/SKILL.md       # Criar hotfix branch
│   ├── create-release/SKILL.md      # Criar release branch
│   ├── git-workflow/SKILL.md        # Git branches + commits + PR
│   ├── rt-ado-deploy/SKILL.md       # User Story de deploy QA
│   ├── rt-fix/SKILL.md              # Workflow de fix Reforma Tributária
│   ├── rt-investigation/SKILL.md    # Investigação antes de fix RT
│   ├── taxone-fix/skill.md          # Workflow de fix TaxOne
│   ├── taxone-review-pr/SKILL.md    # Solicitar review via Teams
│   └── saffron/                     # Skills Saffron
│       ├── saffron-a11y/            # WCAG/ARIA
│       ├── saffron-angular-integration/
│       ├── saffron-tokens/
│       ├── saffron-design-system-source/
│       ├── saffron-figma-angular/
│       └── saffron-mcp-saffron/
├── config/                          # Configuração global
│   ├── CLAUDE.md                    # Instruções globais (idioma, PRs, Sincerão, agents)
│   ├── settings.json                # Settings sanitizado
│   ├── mcp-servers.json             # MCP servers sanitizado
│   └── mcp-servers.md               # Documentação dos MCP servers
├── docs/                            # Documentação de referência
│   └── architecture-saffron-multi-agent.md
├── memories/                        # Memórias por projeto (11 projetos)
│   ├── taxonert-frontend/           # 19 memórias
│   ├── rt-agents-conciliation/      # 13 memórias
│   ├── taxone-ecosystem-mcp/        # 4 memórias
│   ├── taxonecomp-RT-DataScanner/   # 4 memórias
│   ├── user-global/                 # 5 memórias (perfil do user)
│   ├── rt-input-service-api/        # 3 memórias
│   ├── echo/                        # 4 memórias
│   ├── taxonecomp-Taxone-Admin/     # 3 memórias
│   ├── sql-reforma-DDL-rt-conciliation/  # 3 memórias
│   ├── taxsami-artifacts/           # 4 memórias
│   └── taxsami-ReferenceUIApplication/   # 1 memória
└── projects/                        # Agents e skills de projetos específicos
    ├── taxonert-frontend/
    │   ├── agents/                  # 7 agentes Saffron multi-agent
    │   ├── skills/                  # 13 skills do projeto
    │   └── CLAUDE.md
    ├── rt-agents-conciliation/
    │   └── CLAUDE.md
    └── taxone-ecosystem-mcp/
        └── skills/write-story/
```

---

## Agentes

### Globais (`agents/`)

| Agente | Arquivo | Descrição |
|--------|---------|-----------|
| **sincerao** | `sincerao.md` | Crítico brutal que valida ideias, código e planos com honestidade afiada. Loop iterativo de até 7 rodadas. Ativado automaticamente em brainstorming (ver `config/CLAUDE.md`). |
| **oraculo** | `oraculo.md` | Especialista em modelagem de dados Oracle DB. Decisões de modelagem, índices, particionamento, performance. Filosofia "só acredito vendo". |
| **slon** | `slon.md` | Especialista em modelagem de dados PostgreSQL. Mesmas responsabilidades do oraculo, mas pro Postgres. Família Sincerão. |

> **Memória persistente:** `oraculo-memory/` e `slon-memory/` contêm decisões, findings e schemas acumulados pelos agentes ao longo de sessões.

### Projeto: taxonert-frontend (`projects/taxonert-frontend/agents/`)

| Agente | Função |
|--------|--------|
| **saffron-orchestrator** | Coordena pipeline, detecta cenário, delega para agentes adequados |
| **saffron-figma-designer** | Analisa Figma via MCP, produz Blueprint de implementação |
| **saffron-visual-designer** | Analisa imagens/texto, produz Blueprint visual |
| **saffron-specialist** | Especialista Saffron, produz Saffron Guide com componentes e tokens |
| **saffron-angular-developer** | Implementa código Angular com Saffron após Blueprint + Guide |
| **saffron-auditor** | Valida checklist A1-A10 de qualidade e corrige |
| **angular-auditor** | Auditor geral Angular (fora do pipeline Saffron) |

> Arquitetura documentada em [`docs/architecture-saffron-multi-agent.md`](docs/architecture-saffron-multi-agent.md).

---

## Skills

### Globais (`skills/`)

| Skill | Invocável? | Descrição |
|-------|-----------|-----------|
| **ado-client** | Não (utilitária) | Padrões curl da REST API do Azure DevOps. Usada por `/rt-fix`, `/taxone-fix`. |
| **close-release** | `/close-release` | Automatiza PR para main e edita GitHub Release com narrativa. |
| **create-hotfix** | `/create-hotfix` | Cria branch de hotfix via GitHub Actions. |
| **create-release** | `/create-release` | Cria branch de release via GitHub Actions. |
| **git-workflow** | Não (utilitária) | Gitflow + commits conventional + PR via gh CLI. |
| **rt-ado-deploy** | `/rt-ado-deploy <artifact> <version>` | Cria User Story de deploy QA com link JFrog. |
| **rt-fix** | `/rt-fix` | Workflow completo de fix Reforma Tributária (rt-*). Detecta backend (NestJS) ou frontend (Angular). |
| **rt-investigation** | Não (sub-skill) | Investigação antes de fix RT backend. Lê código, encontra padrões, consulta ecosystem. |
| **taxone-fix** | `/taxone-fix {id}` | Workflow completo de fix TaxOne. Branch `fix/MFS{id}/...` ← `rc`, bump versão, PR dev + PR rc (draft). |
| **taxone-review-pr** | `/taxone-review-pr [pr-number]` | Solicita review do PR via Teams (Adaptive Card). |

### Globais Saffron (`skills/saffron/`)

| Skill | Descrição |
|-------|-----------|
| **saffron-a11y** | WCAG/ARIA para componentes Saffron — usa MCP `get-saffron-a11y-attributes`. |
| **saffron-angular-integration** | Web components FAST (`saf-*`) em Angular — CUSTOM_ELEMENTS_SCHEMA, kebab-case, property binding. |
| **saffron-design-system-source** | Navegar monorepo fonte do Saffron Design System. |
| **saffron-tokens** | Mapear CSS values → tokens Saffron (`--saf-*`) via MCP. |
| **saffron-figma-angular** | Fluxo completo Figma → Angular com Saffron. |
| **saffron-mcp-saffron** | Opera ferramentas MCP do Saffron (code, tokens, a11y, equivalents). |

### Projeto: taxonert-frontend (`projects/taxonert-frontend/skills/`)

13 skills específicas (front-engineer, git-workflow do time, investigation, bugfix, saffron-*, etc).

### Projeto: taxone-ecosystem-mcp (`projects/taxone-ecosystem-mcp/skills/`)

| Skill | Descrição |
|-------|-----------|
| **write-story** | Escrever histórias ADO com impact analysis do ecossistema. |

---

## MCP Servers

5 servidores configurados em `~/.claude.json`. Detalhes completos em [`config/mcp-servers.md`](config/mcp-servers.md).

| Server | Propósito |
|--------|-----------|
| **azure-devops** | Work items, pipelines, wikis, repos (org `tr-ggo`). |
| **SONAR_MCP** | SonarQube TR — quality gate, métricas, issues. |
| **saffron-mcp-server** | Saffron Design System — code, tokens, ARIA, mapeamento HTML → saf-*. |
| **taxone-ecosystem** | Grafo do ecossistema TaxOne — 46 componentes, impact analysis. |
| **tr-code-scan-mcp** | Security scanning — SQL injection, XSS, path traversal, remediation. |

**Não documentados aqui** (mas aparecem em sessão):
- `plugin_figma_figma` — vem do plugin `figma@claude-plugins-official`.
- `claude_ai_*` — MCPs OAuth hospedados no claude.ai (HighQ, Microsoft 365, HubSpot, Pendo, SalesForce, etc).

---

## Memórias

Memórias persistentes em `~/.claude/projects/<encoded-path>/memory/`, organizadas por projeto:

| Projeto | Memórias | Foco |
|---------|----------|------|
| **taxonert-frontend** | 19 | Saffron (cards, dialogs, layout, eventos), Wijmo grids, pipeline orchestrator. |
| **rt-agents-conciliation** | 13 | Decisões MVP conciliação, regras inbox/conciliation, Docker WSL, PostgreSQL porta 3005. |
| **taxone-ecosystem-mcp** | 4 | Contexto do ecossistema (46 componentes), projeto ADO Story Writer. |
| **taxonecomp-RT-DataScanner** | 4 | DataScanner RT. |
| **user-global** | 5 | Perfil do user (memórias de `~/.claude/projects/C--Users-6045076/`). |
| **rt-input-service-api** | 3 | API de input services RT. |
| **echo** | 4 | Projeto echo. |
| **taxonecomp-Taxone-Admin** | 3 | Taxone Admin. |
| **sql-reforma-DDL-rt-conciliation** | 3 | DDLs SQL reforma RT conciliation. |
| **taxsami-artifacts** | 4 | Taxsami artifacts. |
| **taxsami-ReferenceUIApplication** | 1 | Reference UI app. |

**Total:** 63 memórias em 11 projetos.

---

## Configuração Global

### `config/CLAUDE.md`

Instruções que se aplicam a **todos** os projetos:

- **Idioma**: Comunicação sempre em pt-BR; código e nomes técnicos em inglês.
- **Estilo de PR**: pt-BR informal com humor técnico. Estrutura obrigatória (O que rolou, R.I.P., Antes vs Depois, Commits, Como testar, Risco, citação irônica).
- **Estilo de Release**: template orientado a impacto por versão (Destaques, Correções, Removido, Áreas impactadas, Risco, PRs incluídos).
- **Padrão de localização de agentes**: `~/.claude/agents/<nome>.md` lowercase, com frontmatter YAML, memória em `<nome>-memory/`.
- **Sincerão automático**: ativado em contextos de ideação/brainstorming, loop iterativo de até 7 rodadas sem pedir permissão.

### `config/settings.json`

- **Plugins**: Superpowers + Figma.
- **Env vars sanitizadas**: `ADO_PAT`, `ADO_MCP_AUTH_TOKEN`, `ADO_ORG`, `ADO_PROJECT`, `TEAMS_WEBHOOK_URL_*`.
- **Hooks**: Pixel Agents em todos os eventos + AI Tracker em Write/Edit/MultiEdit (PreToolUse + PostToolUse).
- **Outros**: `autoUpdatesChannel: latest`, `skipDangerousModePermissionPrompt: true`.

### `config/mcp-servers.json` + `config/mcp-servers.md`

JSON sanitizado dos 5 MCP servers + documentação de como restaurar cada um.

---

## Como Restaurar

```bash
# 1. Copiar agents e skills globais
cp -r agents/* ~/.claude/agents/
cp -r skills/* ~/.claude/skills/

# 2. Copiar config global
cp config/CLAUDE.md ~/.claude/CLAUDE.md
cp config/settings.json ~/.claude/settings.json
# IMPORTANTE: editar settings.json e substituir <<YOUR_*_HERE>> pelos tokens reais

# 3. Configurar MCP servers
# Editar ~/.claude.json e adicionar bloco "mcpServers" de config/mcp-servers.json
# Substituir <AZURE_DEVOPS_PAT>, <SONARQUBE_TOKEN>, <USER> pelos valores reais
# Ver detalhes de cada server em config/mcp-servers.md

# 4. Copiar memórias (paths dependem do diretório do projeto)
# Formato: ~/.claude/projects/<path-encoded>/memory/
# Ex: C:\repositories\rt-agents-conciliation → C--repositories-rt-agents-conciliation

# 5. Copiar agents e skills de projeto
cp -r projects/taxonert-frontend/agents/* /path/to/taxonert_frontend/.claude/agents/
cp -r projects/taxonert-frontend/skills/* /path/to/taxonert_frontend/.claude/skills/
```

---

> *"A memória é a única coisa que sobrevive ao `rm -rf` do tempo."* — Dev Anônimo
