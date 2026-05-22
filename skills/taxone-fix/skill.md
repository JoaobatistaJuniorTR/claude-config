---
name: taxone-fix
description: Workflow completo para fix em projetos TaxOne - analisa issue ADO, cria branch, implementa fix, roda testes, abre PRs (dev + rc draft) seguindo convencoes do time
user_invocable: true
arguments: issue_id
---

# /taxone-fix — Workflow de Fix para projetos TaxOne

Voce recebeu uma issue do Azure DevOps para implementar um fix. Siga este workflow rigorosamente.

## Entrada

O usuario fornece o **ID da work item** do ADO (ex: `1080462`). Pode vir como numero puro ou com prefixo MFS.

## Passo 0: Ler a Issue no ADO

1. Use `mcp__azure-devops__search_workitem` para localizar a issue e descobrir o projeto
2. Use `mcp__azure-devops__wit_get_work_item` com `expand: all` para pegar detalhes completos (descricao, acceptance criteria, relations, tags)
3. Use `mcp__azure-devops__wit_list_work_item_comments` para pegar comentarios adicionais
4. Extraia:
   - **Numero MFS**: o ID da work item (ex: `1080462`)
   - **Titulo**: para gerar breve descricao do branch
   - **Descricao**: para entender o fix necessario
   - **Projeto ADO**: para referencias futuras

## Passo 1: Analisar o Contexto

1. Identifique em qual repositorio voce esta trabalhando (verifique `git remote -v`)
2. Determine se e uma **lib** ou um **servico**:
   - Se for **lib** (ex: multitenancy, framework): pergunte ao usuario qual projeto sera usado para testar
   - Se for **servico** (ex: extractor, publisher): o teste e no proprio projeto
3. Analise o codigo relevante para entender a causa raiz do bug
4. Identifique os arquivos que precisam ser alterados

## Passo 2: Planejar e Criar Tasks no ADO

> **Filosofia:** Planejar ANTES de executar. As tasks sao o plano de trabalho — criadas agora, fechadas conforme cada passo for concluido.

Apos a analise (Passo 1), voce ja sabe o que precisa ser feito. Crie as tasks filhas da User Story **ANTES** de comecar a implementacao.

### 2.1 Definir o plano de tasks

Com base na analise, determine quais tasks serao necessarias. Tasks tipicas:

| Task | Quando criar | Horas tipicas |
|------|-------------|---------------|
| Analise e investigacao | Sempre — leitura da issue, analise do codigo, identificacao da causa | 0.5h (simples) a 2h (complexo) |
| Implementacao | Sempre — codigo, bump de versao | 0.5h (1 arquivo, poucas linhas) a 4h (multiplos arquivos, logica complexa) |
| Testes | Quando testes serao escritos ou executados | 0.5h a 2h |
| Correcao de seguranca (Snyk) | Criar apenas SE o scan encontrar vulnerabilidades | 0.5h a 1h |
| Correcao de qualidade (SonarQube) | Criar apenas SE issues forem encontradas | 0.5h a 1h |
| Code Review / PRs | Sempre — criacao dos PRs, resolucao de conflitos | 0.5h (sem conflito) a 1h (com conflito) |

> **Nota:** A task de "Analise e investigacao" ja pode ser criada como **Closed** neste ponto, pois a analise acaba de ser concluida.

### 2.2 Criar as tasks no ADO

Crie cada task usando `mcp__azure-devops__wit_create_work_item` com:

```
workItemType: "Task"
fields:
  - System.Title: "[MFS{numero}] {descricao da task}"
  - System.AssignedTo: {mesmo usuario da US}
  - System.IterationPath: {mesma iteracao da US}
  - System.AreaPath: {mesmo area path da US}
  - Microsoft.VSTS.Scheduling.OriginalEstimate: {horas estimadas}
  - Custom.Component: "TaxOne"   ← SEMPRE este valor, NUNCA perguntar
```

Apos criar, vincule cada task como filha da US usando `mcp__azure-devops__wit_work_items_link`:
```
type: "child"
id: {id da US}
linkToId: {id da task criada}
```

### 2.3 Atualizar tasks durante a execucao

- **Ao iniciar um passo**: atualize a task correspondente para `State: "Active"`
- **Ao concluir um passo**: atualize a task para `State: "Closed"` com:
  - `Custom.ResolutionType: "Fixed"`
  - `Microsoft.VSTS.Scheduling.CompletedWork: {horas reais gastas}`
- **Tasks condicionais** (Snyk, SonarQube): crie-as apenas se/quando o scan encontrar issues para corrigir

> **CRITICO:** `Custom.Component` e SEMPRE `"TaxOne"`. Nunca pergunte ao usuario.

## Passo 3: Criar Branch

> **Task ADO:** Marque a task "Analise e investigacao" como **Closed** (ja foi concluida no Passo 1).

**Padrao de nome:**
```
fix/MFS{numero}/{breve-descricao-em-kebab-case}
```

Exemplos:
- `fix/MFS1080462/add-dbschema-connection-config`
- `fix/MFS1095000/null-pointer-tenant-refresh`

**Sempre criar a partir do branch `rc`:**
```bash
git checkout rc
git pull origin rc
git checkout -b fix/MFS{numero}/{breve-descricao}
```

> Para features, use `feat/MFS{numero}/{breve-descricao}` e para refactors, `refactor/MFS{numero}/{breve-descricao}`.

## Passo 4: Bump de Versao

Edite o `pom.xml` do projeto:
- **Fix**: bump **patch** (ex: 5.2.0 -> 5.2.1)
- **Feature**: bump **minor** (ex: 5.2.0 -> 5.3.0)
- **Breaking change**: bump **major** (ex: 5.2.0 -> 6.0.0)

Altere APENAS a tag `<version>` do projeto (nao do parent).

## Passo 5: Implementar o Fix

> **Task ADO:** Marque a task "Implementacao" como **Active** ao iniciar.

1. Faca as alteracoes de codigo necessarias baseado na analise da issue
2. Siga os padroes existentes do projeto (imports, formatacao, convencoes)
3. NAO adicione codigo desnecessario, docstrings extras ou refactors nao solicitados
4. Se a issue mencionar testes necessarios, implemente-os

## Passo 6: Testes

> **Task ADO:** Se existir task de "Testes", marque como **Active**.

1. Atualize/crie testes unitarios que cubram o cenario do fix
2. Execute os testes: `mvn test` (ou o comando equivalente do projeto)
3. Todos os testes DEVEM passar antes de prosseguir
4. Se algum teste falhar, investigue e corrija

## Passo 7: Commit e Push

> **Task ADO:** Marque as tasks de "Implementacao" e "Testes" (se existir) como **Closed** com `Custom.ResolutionType: "Fixed"` e `CompletedWork` preenchido.

**Mensagem de commit:**
```
[Multitenancy] - Fix: breve descricao do que foi corrigido

AB#1080462
```

O prefixo entre colchetes deve ser o nome do modulo/projeto. O `AB#{numero}` no corpo do commit linka automaticamente com o ADO.

```bash
git add <arquivos-especificos>
git commit -m "mensagem"
git push -u origin fix/MFS{numero}/{breve-descricao}
```

## Passo 8: Scan de Seguranca (Pre-PR)

> **Objetivo:** O codigo gerado pela skill deve sair limpo — zero vulnerabilidades, zero issues novas no SonarQube, zero bloqueio no Quality Gate.

> **Regra de commits (CRITICO):** NUNCA misture naturezas diferentes no mesmo commit. Cada tipo de alteracao tem seu proprio commit:
> - Commit de **implementacao**: o fix/feat em si (Passo 5)
> - Commit de **seguranca (Snyk)**: `[Modulo] - Security: correcao de vulnerabilidades Snyk`
> - Commit de **qualidade (SonarQube)**: `[Modulo] - Quality: correcao de issues SonarQube`
> - Todos com `AB#{numero}` no corpo do commit

### 8.1 Snyk Code Scan (tr-code-scan)

Antes de qualquer PR, rode o scan de seguranca do Snyk no projeto:

1. Execute `mcp__tr-code-scan-mcp__scan_project` com `project_directory` apontando para a raiz do repositorio
2. **Se o scan retornar issues:**
   - Filtre por tipo: consulte o resource `vulnerability-types://snyk` para listar os tipos encontrados
   - Para cada tipo, consulte `list-vulnerabilities-by-type://snyk/{tipo}` para ver os detalhes
   - Consulte `remediation-knowledge://{tipo_normalizado}/{linguagem}` para orientacao de fix (tipo normalizado: lowercase com underscores, ex: `sql_injection`; linguagem: `java` para projetos TaxOne)
   - **Corrija TODAS as vulnerabilidades encontradas** — vulnerabilidade e vulnerabilidade, independente de ter sido introduzida por voce ou ser pre-existente
   - Faca commit **exclusivo** para as correcoes de seguranca (ver regra de commits abaixo)
   - Re-execute o scan ate que esteja limpo
3. **Se o scan retornar limpo:** prossiga para 8.2

> **Nota:** O scanner e o Snyk (SAST + dependencias). Diferente do SonarQube, aqui corrigimos tudo — nao apenas o que introduzimos.

### 8.2 SonarQube (Pre-PR)

Antes de abrir os PRs, verifique a saude geral do projeto no SonarQube:

1. Descubra a **project key** do repositorio:
   - **Projetos Maven (backend):** leia o `pom.xml` e monte `{groupId}:{artifactId}` (ex: `com.thomsonreuters.taxone:taxone-admin`)
   - **Projetos frontend:** procure `sonar.projectKey` em `.github/workflows/sonar.yml` ou `sonar-project.properties`
   - Se nao funcionar, pergunte ao usuario qual e a project key
2. Verifique o **Quality Gate** e **ratings**: `mcp__SONAR_MCP__get_project_measures` com metricas `alert_status`, `quality_gate_details`, `reliability_rating`, `security_rating`, `sqale_rating`
   - `alert_status` retorna OK/ERROR/WARN
   - `quality_gate_details` retorna JSON com cada condicao, threshold, valor atual e status individual
3. Consulte as **metricas gerais**: `mcp__SONAR_MCP__get_project_measures` com metricas `coverage`, `bugs`, `vulnerabilities`, `code_smells`
4. Consulte o **resumo de issues**: `mcp__SONAR_MCP__get_project_issues_summary` com a project key
5. Busque issues nos **arquivos alterados**: `mcp__SONAR_MCP__search_issues` com `componentKeys` apontando para os arquivos que voce alterou (formato: `{projectKey}:caminho/do/arquivo.java`) e `statuses: ["OPEN"]`
   - Se encontrar issues nos arquivos que voce mexeu, **corrija-as** antes de abrir o PR
   - Faca novo commit, push, e re-execute este passo

> **Importante:** NAO use `get_quality_gate_status` — ele retorna UNKNOWN para Quality Gates customizados. Use `get_project_measures` com `quality_gate_details` que retorna o JSON completo com condicoes e status.

**Decisao pre-PR:**

- **Quality Gate OK e sem issues nos arquivos alterados**: prossiga para os PRs
- **Quality Gate FALHOU**: reporte ao usuario com detalhes e pergunte se deseja corrigir ou prosseguir
- **Issues nos arquivos alterados**: corrija automaticamente, commit, push, e re-valide

## Passo 9: Pull Requests

> **Task ADO:** Marque a task "Code Review / PRs" como **Active**.

### 9.1 PR para branch `dev`

**Titulo:** `[DEV] - MFS{numero} - Breve titulo descritivo`

Exemplo: `[DEV] - MFS1080462 - Fix dbSchema ausente no ConnectionConfig`

**Descricao:** Use o padrao narrativo definido no CLAUDE.md global (tom informal PT-BR, com humor tecnico, secoes obrigatorias). OBRIGATORIO incluir no corpo:

```
ADO: AB#{numero}
```

Isso cria o link automatico com a work item do ADO.

**Target branch:** `dev`

### 9.2 Verificacao SonarQube (Pos-PR)

Apos criar o PR para `dev`, o SonarQube roda a analise de PR automaticamente. As issues dessa analise so ficam disponiveis via parametro `pullRequest` (NAO `branch` — sao mecanismos separados).

1. Aguarde a analise do SonarQube completar (aguarde ~2 minutos apos o push, NAO pergunte ao usuario — faca automaticamente)
2. Busque issues do PR: `mcp__SONAR_MCP__search_issues` com `projectKeys: ["{projectKey}"]` e `pullRequest: "{numeroPR_dev}"`
3. **Se encontrar issues introduzidas pelas suas alteracoes:**
   - Corrija todas (CRITICAL e MAJOR sao obrigatorias; INFO sao recomendadas)
   - Faca novo commit e push no branch
   - Re-execute a busca ate zerar as issues introduzidas
4. **Se todas as issues forem pre-existentes** (nao introduzidas por voce): prossiga normalmente
5. So prossiga para 9.3 (PR rc) quando o PR estiver limpo

> **Meta:** O PR nao deve introduzir NENHUMA issue nova no SonarQube. Issues pre-existentes do projeto nao sao responsabilidade desta skill.

### 9.3 PR para branch `rc` (DRAFT)

**Titulo:** `[RC] - MFS{numero} - Breve titulo descritivo`

Mesmo conteudo do PR de dev, mas:
- Criar como **draft/rascunho**
- **Target branch:** `rc`
- Criar **somente apos** a verificacao SonarQube (9.2) estar limpa

### 9.4 Workaround obrigatoria do WIP App (Pos-PR rc)

> **Bug conhecido:** Apos criar o PR rc DRAFT (9.3) apontando para o mesmo HEAD que o PR dev (9.1), o GitHub WIP App fica eternamente `in_progress` com titulo "draft mode override" — porque o commit pertence a dois PRs simultaneamente, e o WIP App entra em modo draft-override considerando o "pior caso" (o PR draft). O check fica travado em **AMBOS** os PRs, e o PR dev nunca consegue ser merged.

**Workaround OBRIGATORIA (executar SEMPRE apos 9.3, antes do Passo 11):**

```bash
# Toggle draft->ready do PR dev para forcar novo check WIP exclusivo daquele PR
gh pr ready {numeroPR_dev} --undo
gh pr ready {numeroPR_dev}
```

Apos executar, validar com:
```bash
gh pr view {numeroPR_dev} --json statusCheckRollup --jq '.statusCheckRollup[] | select((.name // .context) == "WIP")'
```

O check WIP deve aparecer como `conclusion: SUCCESS` / "Ready for review". Se ainda estiver pending, aguarde alguns segundos e re-tente.

### 9.5 Estrategia de Conflitos (CRITICO)

Se o PR para `dev` tiver conflitos (porque `dev` tem codigo que nao esta em `rc`):

1. **NUNCA** faca merge de `dev` dentro do branch original
2. Crie um **novo branch** a partir do original:
   ```bash
   git checkout fix/MFS{numero}/{breve-descricao}
   git checkout -b fix/MFS{numero}/{breve-descricao}-conflict
   ```
3. **Neste branch `-conflict`**, faca merge de `dev`:
   ```bash
   git merge origin/dev
   # resolva conflitos
   git push -u origin fix/MFS{numero}/{breve-descricao}-conflict
   ```
4. Crie o PR do branch `-conflict` para `dev`
5. **Mantenha** o PR original (branch limpo) apontando para `rc` como draft
6. Assim o branch que vai pra `rc` fica limpo, sem codigo do `dev` misturado

## Passo 10: Se for Lib — Atualizar Projeto de Teste

Quando o fix e em uma **biblioteca** (ex: multitenancy), e necessario atualizar o projeto consumidor para testar:

1. Navegue ate o projeto de teste (ex: taxone-extractor)
2. Siga os mesmos passos 3-9 nesse projeto:
   - Criar branch com MESMO numero MFS: `fix/MFS{numero}/{breve-descricao-do-update}`
   - Bump de versao do projeto (patch)
   - Atualizar a versao da dependencia da lib no `pom.xml`
   - Commit, push, PRs (dev + rc draft)
3. A mesma estrategia de conflitos se aplica

## Passo 11: Finalizar Tasks no ADO

> **Task ADO:** Marque a task "Code Review / PRs" como **Closed** com `Custom.ResolutionType: "Fixed"`.

Ao final da execucao, garanta que **todas as tasks** criadas no Passo 2 estejam fechadas:

1. Verifique as tasks filhas da US: todas devem estar com `State: "Closed"`
2. Cada task deve ter `CompletedWork` preenchido com as horas reais gastas
3. Se alguma task condicional (Snyk, SonarQube) nao foi necessaria, delete-a ou feche com 0h

> **Recapitulando o ciclo de vida de cada task:**
> - Criada no **Passo 2** como `New` com `OriginalEstimate`
> - Marcada como `Active` ao iniciar o passo correspondente
> - Marcada como `Closed` com `Custom.ResolutionType: "Fixed"` e `CompletedWork` ao concluir

---

## Resumo de Convencoes

| Item | Padrao |
|------|--------|
| Branch fix | `fix/MFS{numero}/{kebab-case}` |
| Branch feat | `feat/MFS{numero}/{kebab-case}` |
| Branch refactor | `refactor/MFS{numero}/{kebab-case}` |
| Origem do branch | Sempre `rc` |
| PR dev titulo | `[DEV] - MFS{numero} - Titulo` |
| PR rc titulo | `[RC] - MFS{numero} - Titulo` (draft) |
| PR desc obrigatoria | `ADO: AB#{numero}` |
| PR desc estilo | Padrao narrativo PT-BR do CLAUDE.md global |
| Versao fix | Patch bump |
| Versao feat | Minor bump |
| Conflito com dev | Branch `-conflict`, NUNCA merge dev no original |
| Commit msg | `[Modulo] - Tipo: descricao\n\nAB#{numero}` |
| Commit implementacao | `[Modulo] - Fix/Feat: descricao` |
| Commit seguranca | `[Modulo] - Security: correcao de vulnerabilidades Snyk` |
| Commit qualidade | `[Modulo] - Quality: correcao de issues SonarQube` |
| Regra de commits | NUNCA misturar naturezas no mesmo commit |
| Scan seguranca | Snyk via `tr-code-scan-mcp` (pre-PR), corrige TUDO |
| SonarQube pre-PR | QG + metricas + issues nos arquivos alterados |
| SonarQube pos-PR | Issues do PR via `pullRequest` param |
| Meta de qualidade | Zero vulnerabilidades, zero issues novas, zero bloqueio no QG |
| Custom.Component | SEMPRE `"TaxOne"` — nunca perguntar ao usuario |
| Tasks ADO | Planejar e criar no Passo 2, fechar conforme execucao |
| Task lifecycle | New → Active → Closed (Fixed) com CompletedWork |
| WIP App workaround | OBRIGATORIO apos 9.3: `gh pr ready X --undo && gh pr ready X` no PR dev |
