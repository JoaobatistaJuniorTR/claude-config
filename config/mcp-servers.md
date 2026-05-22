# MCP Servers

Servidores MCP configurados no `~/.claude.json` (Windows: `C:\Users\<user>\.claude.json`).

A config sanitizada está em [`mcp-servers.json`](mcp-servers.json) — placeholders precisam ser substituídos por valores reais ao restaurar.

---

## Servidores configurados

### 1. `azure-devops`

**Propósito:** Integração completa com Azure DevOps (work items, pipelines, wikis, repos).

**Como rodar:** `npx -y @azure-devops/mcp tr-ggo -d core work work-items search wiki pipelines`

**Requer:**
- `AZURE_DEVOPS_PAT` — Personal Access Token do ADO (Tools-team usa `tr-ggo` org).
- Como gerar: ADO → User settings → Personal Access Tokens → New Token com escopo `vso.work_write`, `vso.code_write`, `vso.build_execute`.

---

### 2. `SONAR_MCP`

**Propósito:** Consultar SonarQube (quality gate, métricas, issues) do TR.

**Como rodar:** `uvx --from git+https://github.com/JoaobatistaJuniorTR/mcp-sonar.git mcp-sonarqube`

**Requer:**
- `SONARQUBE_URL` = `https://sonar.qa.thomsonreuters.com`
- `SONARQUBE_TOKEN` — token gerado no SonarQube em `My Account → Security → Generate Tokens`.
- `uv`/`uvx` instalado (via scoop: `scoop install uv`).

---

### 3. `saffron-mcp-server` (HTTP)

**Propósito:** Saffron Design System — code snippets, tokens, ARIA, mapeamento HTML → saf-*.

**URL:** `https://aideployment-prod.plexus.gcs.int.thomsonreuters.com/207870/ask-saffron-mcp/`

**Requer:** rede TR (VPN ou estação interna).

---

### 4. `taxone-ecosystem` (local)

**Propósito:** Grafo do ecossistema TaxOne — 46 componentes (34 serviços + 12 libs), impact analysis, dependency graph.

**Como rodar:** `node C:/repositories/taxone-ecosystem-mcp/dist/src/index.js`

**Requer:**
- Clone de `taxone-ecosystem-mcp` em `C:/repositories/`.
- Build feito (`npm run build`).
- `KNOWLEDGE_PATH` apontando pra `C:/repositories/taxone-ecosystem-mcp/knowledge`.

---

### 5. `tr-code-scan-mcp` (local)

**Propósito:** Security scanning — identificar e remediar vulnerabilidades (SQL injection, XSS, path traversal, etc).

**Como rodar:** binário local `C:\Users\<USER>\TR\tr-code-scan-windows.exe mcp-server`.

**Requer:** binário fornecido pelo time TR (não é open-source).

---

## Como restaurar

```bash
# 1. Editar ~/.claude.json (ou usar `claude mcp add` se a CLI suportar):
#    - Copiar bloco "mcpServers" de mcp-servers.json
#    - Substituir <AZURE_DEVOPS_PAT>, <SONARQUBE_TOKEN>, <USER>

# 2. Validar:
claude mcp list

# 3. Reiniciar o Claude Code.
```

---

## MCPs que NÃO estão neste arquivo (mas aparecem na sessão)

- **plugin_figma_figma** — vem do plugin `figma@claude-plugins-official` (gerenciado pelo Claude Code, não precisa configurar manualmente).
- **claude_ai_***  — MCPs hospedados em claude.ai (CoCounsel, HighQ, HubSpot, Microsoft 365, Pendo, SalesForce, etc). Conectados via OAuth automaticamente, não vivem em `~/.claude.json`.
