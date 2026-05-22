---
name: rt-fix
description: Workflow completo para fix em projetos da Reforma Tributaria (rt-*) - analisa issue ADO, cria branch de develop, implementa fix, roda testes, abre PR. Detecta automaticamente o tipo de projeto (NestJS backend ou Angular frontend).
user_invocable: true
arguments: issue_id_or_command
---

# /rt-fix — Workflow de Fix para projetos RT (Reforma Tributaria)

Voce recebeu uma issue do Azure DevOps para implementar um fix em um projeto da Reforma Tributaria. Siga este workflow rigorosamente.

## Invocacao

```
/rt-fix 12345
/rt-fix https://dev.azure.com/tr-ggo/Mastersaf%20Fiscal%20Solutions/_workitems/edit/12345
/rt-fix close 12345
```

---

## Input Parsing

Parse os argumentos:

1. Se primeiro argumento e `close` → **Close Mode** (Fase 7 apenas). Segundo argumento e o work item ID.
2. Se argumento e um numero → **Fix Mode** (Fases 1-6). O numero e o work item ID.
3. Se argumento e uma URL contendo `/_workitems/edit/` → **Fix Mode**. Extrair o ID com: `echo "$URL" | grep -oP '/_workitems/edit/\K\d+'`
4. Caso contrario → informar: "Usage: `/rt-fix <id>` ou `/rt-fix <url>` ou `/rt-fix close <id>`" e PARAR.

---

## Fase 0: Detectar Tipo de Projeto

Antes de qualquer coisa, identifique em qual tipo de projeto esta trabalhando.

**Step 1 — Detectar:**

| Arquivo na raiz | Tipo | Stack |
|---|---|---|
| `angular.json` | Frontend Angular | ng build, ng test |
| `nest-cli.json` ou `src/main.ts` com NestJS imports | Backend NestJS | npm run build, npm test |
| `package.json` sem Angular/Nest | Backend Node generico | npm run build, npm test |

**Step 2 — Confirmar:**

Apresente ao usuario o que detectou:

> "Detectei projeto **{tipo}** ({stack}). Prosseguindo no modo {tipo}."

Se detectar ambiguidade (ex: monorepo com front + back), pergunte:

> "Detectei ambos Angular e NestJS neste repositorio. Qual e o alvo do fix?"

**Step 3 — Definir comandos:**

| Comando | Frontend Angular | Backend NestJS/Node |
|---|---|---|
| build | `ng build` | `npm run build` |
| test | `ng test --watch=false` | `npm test` |
| lint | `ng lint` | `npm run lint` |

---

## Fix Mode (Fases 1–6)

### Fase 1: Captura (Ler Issue do ADO)

Busque o Bug/Task no ADO e apresente um resumo estruturado.

**Step 1 — Carregar e validar ambiente:**

As variáveis ADO estão em `~/.claude/settings.json` sob a chave `env`. O terminal do VS Code NÃO as injeta automaticamente — é necessário carregá-las antes de qualquer chamada.

Siga a seção "Loading Variables" do skill `ado-client` para carregar `ADO_PAT`, `ADO_ORG` e `ADO_PROJECT` a partir do arquivo de settings.

Após carregar, valide que as três estão definidas. Se alguma estiver vazia, PARAR e informar o usuário.

**Alternativa: Use as ferramentas MCP do Azure DevOps se disponiveis:**
1. `mcp__azure-devops__wit_get_work_item` com `expand: all` para detalhes completos
2. `mcp__azure-devops__wit_list_work_item_comments` para comentarios adicionais

**Step 2 — Extrair campos:**

| Campo ADO | Uso |
|---|---|
| `System.Title` | Titulo do bug |
| `Microsoft.VSTS.TCM.ReproSteps` | Passos de reproducao (HTML → texto limpo) |
| `Microsoft.VSTS.TCM.SystemInfo` | Info do sistema |
| `System.Description` | Descricao (fallback se ReproSteps vazio) |
| `Microsoft.VSTS.Common.Priority` | Prioridade (1=Critical, 4=Low) |
| `Microsoft.VSTS.Common.Severity` | Severidade (1=Critical, 4=Low) |
| `System.State` | Estado atual |

**Step 3 — Buscar attachments:**

Siga `ado-client.md` Operations 2 e 3:
- Listar attachments do array `relations`
- Baixar imagens (PNG, JPG, GIF) para `/tmp/`
- Usar `Read` para visualizar imagens (Claude e multimodal)

**Step 4 — Apresentar resumo:**

```
## Bug #{ID}: {titulo}
**Estado**: {state} | **Prioridade**: {priority} | **Severidade**: {severity}

### Repro Steps
{texto limpo do HTML}

### System Info
{system info se disponivel, ou "Nao informado"}

### Screenshots
{descricao de cada imagem apos visualiza-las}
```

---

### Fase 2: Triage (Compreensao)

Avalie se o bug tem informacao suficiente para prosseguir.

**Criterios minimos:**
- **O que acontece** — descricao clara do comportamento errado
- **Onde acontece** — tela, modulo ou caminho no sistema (suficiente para buscar no codigo)

Screenshot NAO e obrigatorio. Se presente, ajuda. Se ausente, descricao textual clara e suficiente.

**Se entendido → Fase 3.**

**Se NAO entendido → Fase 2a: Perguntar ao dev na conversa.**

> "Li o Bug #{ID} mas preciso de mais informacoes:
> - {ponto especifico que falta}
> - {outro ponto}
> Voce sabe responder isso?"

Aguardar resposta do dev.

- Se o dev responde → reavaliar triage com a nova info.
- Se o dev diz "nao sei" → **Fase 2b: Escalar para ADO.**

**Fase 2b — Postar comentario no ADO:**

Use `mcp__azure-devops__wit_add_work_item_comment` ou siga `ado-client.md` Operation 4:

```html
<b>Preciso de mais informacoes para corrigir este bug:</b><br>
<ul>
<li>{ponto 1}</li>
<li>{ponto 2}</li>
</ul>
<br><em>Comentario gerado via Claude Code</em>
```

Informar o usuario:
> "Comentei no Bug #{ID} no ADO pedindo mais informacoes. Quando tiver resposta, rode `/rt-fix {ID}` novamente."

**PARAR execucao.**

---

### Fase 3: Criar Task no ADO

Apos triage positiva, registre o trabalho no ADO.

**Step 1 — Criar Task filha:**

Use `mcp__azure-devops__wit_add_child_work_items` ou siga `ado-client.md` Operation 6:
- Titulo: `Fix: {descricao curta em ingles}`
- Original Estimate: estimar horas baseado na complexidade (simples=1h, medio=2h, complexo=4h)
- Parent link: o ID da work item do Bug

**Step 2 — Informar o dev:**

> "Task #{NEW_TASK_ID} criada no ADO, linkada ao Bug #{BUG_ID}. Estimativa: {hours}h."

---

### Fase 4: Branch e Setup

Siga o skill `git-workflow` (global) — Moment 1.

Regras criticas (resumo inline):
- Base branch: `develop`
- Prefix: `bugfix/` (para bugs), `feature/` (para features), `hotfix/` (para producao)
- Descricao em ingles, kebab-case, derivada do titulo da issue
- Exemplos: `bugfix/null-pointer-tenant-refresh`, `bugfix/pagination-labels`, `feature/export-button`

---

### Fase 5: Implementar

**Step 1 — Analisar o codigo:**

1. Busque no codebase pelos arquivos relevantes ao bug (use Grep, Glob, Read)
2. Entenda a causa raiz
3. Identifique os arquivos que precisam ser alterados

**Step 2 — Se for projeto Frontend Angular:**

Delegue ao pipeline `front-engineer` (investigation):
- O contexto do bug da Fase 1 (titulo, repro steps, screenshots, analise) e a entrada
- O `front-engineer` aciona o skill `investigation` do front (cascade Angular/Saffron)
- **CRITICO:** NAO implemente sem diagnostico confirmado pelo usuario

**Step 3 — Se for projeto Backend NestJS/Node:**

Siga o skill `rt-investigation` (global):
- Passe o contexto do bug da Fase 1 como entrada
- O skill guia pelas 5 fases: Understand → Investigate → Consult → Diagnose → Implement
- A cascade de consulta e especifica pro backend: CLAUDE.md do projeto → codebase patterns → TaxOne Ecosystem MCP → memories → WebSearch → SonarQube
- **CRITICO:** NAO implemente sem diagnostico confirmado pelo usuario (Iron Law)

**Step 4 — Testes:**

1. Atualize/crie testes unitarios que cubram o cenario do fix
2. Execute os testes com o comando do profile detectado
3. Todos os testes DEVEM passar antes de prosseguir
4. Se algum teste falhar, investigue e corrija

**Step 5 — Lint:**

1. Execute o linter com o comando do profile detectado (Angular: `ng lint` / NestJS/Node: `npm run lint`)
2. **Zero erros obrigatorio** — corrija todos os erros antes de prosseguir
3. Se o linter reportar erros nos arquivos que voce alterou, corrija imediatamente
4. Se o linter reportar erros em arquivos que voce NAO alterou, corrija tambem — o validate-build vai bloquear o PR se houver qualquer erro de lint no projeto
5. Apos corrigir, re-execute o linter para confirmar que esta limpo

**Step 6 — Build:**

1. Execute o build com o comando do profile detectado
2. **Zero erros e zero warnings obrigatorio**
3. Se falhar, corrija antes de prosseguir

---

### Fase 6: Commit, Push e PR

Siga o skill `git-workflow` (global) — Moment 2.

Regras criticas (resumo inline):
- Stage apenas arquivos relevantes (NUNCA `git add .`)
- Conventional commits em ingles: `fix: {descricao}`
- PR para `develop` com descricao no estilo CLAUDE.md global (PT-BR, narrativo, ironico)
- OBRIGATORIO incluir `ADO: AB#{numero}` no corpo do PR para linkar com o ADO

**Step 4 — Comentar no ADO:**

Use `mcp__azure-devops__wit_add_work_item_comment` no Bug E na Task:

```html
PR aberto para revisao: <a href="{PR_URL}">{PR_URL}</a><br>
Branch: <code>bugfix/{branch-name}</code><br>
<br><em>Comentario gerado via Claude Code</em>
```

**Step 5 — Informar o usuario:**

> "PR criado: {PR_URL}
> Comentei no Bug #{ID} e na Task #{TASK_ID} no ADO com o link do PR.
> Apos sua revisao e aprovacao, rode `/rt-fix close {ID}` para fechar no ADO."

**PARAR — Fase de review e assincrona.**

---

## Close Mode (Fase 7) — Enviar para QA

Invocado via `/rt-fix close {ID}`. So roda apos o dev ter revisado e aprovado o PR.

**Step 1 — Perguntar horas ao dev:**

> "Quantas horas gastar nesse fix? (minha estimativa foi {X}h)"

Aguardar resposta do dev.

**Step 2 — Fechar Task no ADO:**

Use `mcp__azure-devops__wit_update_work_item` ou siga `ado-client.md` Operation 5:
- Set `Completed Work` = horas confirmadas pelo dev
- Set `Remaining Work` = 0

**IMPORTANTE — Transicao de estado da Task:**
O ADO exige transicoes validas. NAO e possivel ir direto de `New` para `Closed`.
Siga esta sequencia:
1. Se estado atual e `New` → primeiro mover para `Active`
2. Depois mover de `Active` para `Closed`
3. O campo `Custom.ResolutionType` e **obrigatorio** para fechar. Usar valor `"No further action required"`.

Exemplo de PATCH bodies em sequencia:
```json
// Step 2a: New → Active
[{"op":"add","path":"/fields/System.State","value":"Active"}]

// Step 2b: Active → Closed (com ResolutionType obrigatorio)
[
  {"op":"add","path":"/fields/System.State","value":"Closed"},
  {"op":"add","path":"/fields/Custom.ResolutionType","value":"No further action required"}
]
```

Se a Task ja estiver em `Active`, pular direto para o Step 2b.

**Step 3 — Enviar Bug para QA no ADO:**

NAO fechar o Bug. Em vez disso, mover para QA:
- Set `System.State` = "QA" (ou manter "Active" se "QA" nao for estado valido — verificar board)
- Set `System.AssignedTo` = "de Souza, Emerson" (QA do time)
- Add tag `QA` ao campo `System.Tags` (preservar tags existentes, adicionar com `;` separador)

Exemplo de PATCH body para adicionar tag e reatribuir:
```json
[
  {"op":"add","path":"/fields/System.AssignedTo","value":"Emerson.deSouza@thomsonreuters.com"},
  {"op":"add","path":"/fields/System.Tags","value":"{tags_existentes}; QA"}
]
```

**IMPORTANTE:** Sempre ler o work item antes de atualizar para preservar as tags existentes.

**Step 4 — Postar comentario no Bug:**

```html
Fix implementado e PR aprovado. Enviado para validacao do QA.<br>
<b>Horas:</b> {hours}h<br>
<b>PR:</b> <a href="{PR_URL}">{PR_URL}</a><br>
<b>Assigned to:</b> de Souza, Emerson (QA)<br>
<br><em>Comentario gerado via Claude Code</em>
```

**Step 5 — Informar o usuario:**

> "Bug #{ID} enviado para QA no ADO (de Souza, Emerson). Task #{TASK_ID} fechada. {hours}h lancadas. Tag `QA` adicionada."

---

## Regras

1. **Nunca fechar bug automaticamente** — `/rt-fix close` envia para QA, NAO resolve/fecha o Bug
2. **Nunca pular investigacao** — mesmo que o bug pareca obvio, siga todas as fases
3. **Sempre branch de develop** — nunca commit direto em develop ou main
4. **Conventional commits em ingles** — `fix:`, `feat:`, etc.
5. **PR para develop** — nunca merge direto
6. **Comentarios no ADO em PT-BR** — lingua do time
7. **Nunca fabricar dados** — se algo nao esta no bug, pergunte
8. **Horas sao estimativas** — Claude estima, dev confirma/ajusta
9. **Diagnostico antes de implementar** — tanto pra front quanto pra back, confirme com o dev
10. **PR no estilo CLAUDE.md global** — PT-BR, narrativo, ironico

## Error Handling

| Situacao | Acao |
|---|---|
| ADO API retorna 401 | PAT expirado. Dizer: "Atualize o ADO_PAT em ~/.claude/settings.json" |
| ADO API retorna 404 | ID errado ou projeto errado. Dizer: "Work item nao encontrado. Verifique o ID." |
| Work item nao e Bug | Informar: "Work item #{ID} e tipo {type}, nao Bug. Deseja prosseguir mesmo assim?" |
| `gh` CLI nao autenticado | Pedir ao usuario: `! gh auth login` |
| Build falha | Corrigir erros antes de commitar |
| Dev nao confirma diagnostico | Revisar diagnostico — NAO prosseguir para implementacao |
| Testes falham | Investigar e corrigir. NAO abrir PR com testes quebrados |

## Skills Relacionadas

- `git-workflow` (global) — Branch management, commits, PRs (referenciado nas Fases 4 e 6)
- `rt-investigation` (global) — Investigation workflow para backend NestJS/Node (referenciado na Fase 5)
- `ado-client` (global) — Padroes de API do ADO referenciados ao longo do workflow
- `front-engineer` (taxonert-frontend) — Orquestra o pipeline de investigacao para fixes de frontend
