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

## Passo 2: Criar Branch

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

## Passo 3: Bump de Versao

Edite o `pom.xml` do projeto:
- **Fix**: bump **patch** (ex: 5.2.0 -> 5.2.1)
- **Feature**: bump **minor** (ex: 5.2.0 -> 5.3.0)
- **Breaking change**: bump **major** (ex: 5.2.0 -> 6.0.0)

Altere APENAS a tag `<version>` do projeto (nao do parent).

## Passo 4: Implementar o Fix

1. Faca as alteracoes de codigo necessarias baseado na analise da issue
2. Siga os padroes existentes do projeto (imports, formatacao, convencoes)
3. NAO adicione codigo desnecessario, docstrings extras ou refactors nao solicitados
4. Se a issue mencionar testes necessarios, implemente-os

## Passo 5: Testes

1. Atualize/crie testes unitarios que cubram o cenario do fix
2. Execute os testes: `mvn test` (ou o comando equivalente do projeto)
3. Todos os testes DEVEM passar antes de prosseguir
4. Se algum teste falhar, investigue e corrija

## Passo 6: Commit e Push

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

## Passo 7: Pull Requests

### 7.1 PR para branch `dev`

**Titulo:** `[DEV] - MFS{numero} - Breve titulo descritivo`

Exemplo: `[DEV] - MFS1080462 - Fix dbSchema ausente no ConnectionConfig`

**Descricao:** Use o padrao narrativo definido no CLAUDE.md global (tom informal PT-BR, com humor tecnico, secoes obrigatorias). OBRIGATORIO incluir no corpo:

```
ADO: AB#{numero}
```

Isso cria o link automatico com a work item do ADO.

**Target branch:** `dev`

### 7.2 PR para branch `rc` (DRAFT)

**Titulo:** `[RC] - MFS{numero} - Breve titulo descritivo`

Mesmo conteudo do PR de dev, mas:
- Criar como **draft/rascunho**
- **Target branch:** `rc`

### 7.3 Estrategia de Conflitos (CRITICO)

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

## Passo 8: Se for Lib — Atualizar Projeto de Teste

Quando o fix e em uma **biblioteca** (ex: multitenancy), e necessario atualizar o projeto consumidor para testar:

1. Navegue ate o projeto de teste (ex: taxone-extractor)
2. Siga os mesmos passos 2-7 nesse projeto:
   - Criar branch com MESMO numero MFS: `fix/MFS{numero}/{breve-descricao-do-update}`
   - Bump de versao do projeto (patch)
   - Atualizar a versao da dependencia da lib no `pom.xml`
   - Commit, push, PRs (dev + rc draft)
3. A mesma estrategia de conflitos se aplica

## Passo 9: Atualizar Tasks no ADO

Se existirem tasks filhas da work item no ADO, atualize o status conforme o progresso.

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
