---
name: taxone-review-pr
description: Sends a PR review request to the TaxOne Teams channel via Adaptive Card. Auto-detects PR from current branch or accepts PR number as argument. Invoked via /taxone-review-pr [pr-number].
---

# /taxone-review-pr — Solicitar Review de PR no Teams

Envia uma mensagem no canal do TaxOne no Teams solicitando revisao de um Pull Request. Sem @mention — e um pedido aberto pro time.

## Invocacao

```
/taxone-review-pr [numero-pr]
```

**Exemplos:**
```
/taxone-review-pr           # detecta PR da branch atual
/taxone-review-pr 42        # PR especifico por numero
```

---

## Input Parsing

1. Se argumento fornecido → usar como numero do PR
2. Se nenhum argumento → detectar PR da branch atual (Fase 1, Step 1)

---

## Fase 1: Obter dados do PR

**Step 1 — Resolver numero do PR:**

Se nenhum argumento foi passado, detectar da branch atual:

```bash
gh pr view --json number,title,url,author,body,changedFiles,baseRefName,headRefName,additions,deletions,state,isDraft,headRepository --jq '.'
```

Se falhar (nenhum PR aberto para a branch):
> Nenhum PR encontrado para a branch atual. Use: `/taxone-review-pr <numero-do-pr>`

E **PARAR**.

Se argumento foi passado:

```bash
gh pr view {numero} --json number,title,url,author,body,changedFiles,baseRefName,headRefName,additions,deletions,state,isDraft,headRepository --jq '.'
```

Se falhar (PR nao encontrado):
> PR #{numero} nao encontrado. Verifique o numero e tente novamente.

E **PARAR**.

**Step 2 — Extrair e preparar dados:**

Do JSON retornado, extrair:
- `number` — numero do PR
- `title` — titulo
- `url` — link do PR no GitHub
- `author.login` — autor
- `body` — descricao (truncar em 200 caracteres, remover markdown pesado)
- `headRepository.name` — nome do repositorio (servico)
- `changedFiles` — numero de arquivos alterados
- `baseRefName` — branch destino (main, develop, etc.)
- `headRefName` — branch origem
- `additions` — linhas adicionadas
- `deletions` — linhas removidas
- `isDraft` — se e draft

**Step 3 — Apresentar ao usuario:**

> PR **#{number}** — {title}
> Servico: {repoName} | Autor: {author} | {changedFiles} arquivos | +{additions} -{deletions}
> Branch: {headRefName} → {baseRefName}
>
> Enviando solicitacao de review no Teams...

---

## Fase 2: Enviar Adaptive Card no Teams

**Prerequisito:** Variavel `TEAMS_WEBHOOK_URL_TAXONE_EVOLUCAO_TECNICA` configurada em `~/.claude/settings.json` com a URL do Incoming Webhook do canal.

**Step 1 — Validar webhook:**

```bash
if [ -z "$TEAMS_WEBHOOK_URL_TAXONE_EVOLUCAO_TECNICA" ]; then
  echo "ERRO: TEAMS_WEBHOOK_URL_TAXONE_EVOLUCAO_TECNICA nao configurada."
  echo "Configure em ~/.claude/settings.json → env → TEAMS_WEBHOOK_URL_TAXONE_EVOLUCAO_TECNICA"
fi
```

Se nao configurada, **informar e PARAR** (Teams e obrigatorio neste skill).

**Step 2 — Montar e enviar Adaptive Card:**

**IMPORTANTE:** Usar `--data-binary` com heredoc e `charset=utf-8` no header. Evitar acentos (a, c, e) no JSON — usar equivalentes ASCII para evitar erros de encoding do Power Automate.

Montar o status badge:
- Se `isDraft == true` → "📝 Draft"
- Senao → "🔍 Open"

```bash
curl -s -H "Content-Type: application/json; charset=utf-8" --data-binary @- "$TEAMS_WEBHOOK_URL_TAXONE_EVOLUCAO_TECNICA" <<'JSONEOF'
{
  "type": "message",
  "attachments": [{
    "contentType": "application/vnd.microsoft.card.adaptive",
    "contentUrl": null,
    "content": {
      "$schema": "http://adaptivecards.io/schemas/adaptive-card.json",
      "type": "AdaptiveCard",
      "version": "1.4",
      "body": [
        {
          "type": "TextBlock",
          "size": "medium",
          "weight": "bolder",
          "text": "🔍 Code Review Solicitado — PR #{number}"
        },
        {
          "type": "TextBlock",
          "text": "**{title}**",
          "wrap": true
        },
        {
          "type": "FactSet",
          "facts": [
            { "title": "Servico:", "value": "{repoName}" },
            { "title": "Autor:", "value": "{author}" },
            { "title": "Branch:", "value": "{headRefName} → {baseRefName}" },
            { "title": "Arquivos:", "value": "{changedFiles} alterados (+{additions} -{deletions})" },
            { "title": "Status:", "value": "{status_badge}" }
          ]
        },
        {
          "type": "TextBlock",
          "text": "{descricao_truncada}",
          "wrap": true,
          "isSubtle": true,
          "maxLines": 3
        },
        {
          "type": "TextBlock",
          "text": "[Abrir PR no GitHub]({url})",
          "wrap": true
        }
      ]
    }
  }]
}
JSONEOF
```

**Step 3 — Validar resposta:**

- Se curl retornou exit code 0 → sucesso
- Se curl falhou (timeout, conexao recusada) → erro claro

**Step 4 — Confirmar ao usuario:**

Se enviou com sucesso:
> Review solicitado no Teams para PR **#{number}** — {title}
> Link: {url}

Se webhook nao configurado:
> ERRO: `TEAMS_WEBHOOK_URL_TAXONE_EVOLUCAO_TECNICA` nao configurada em `~/.claude/settings.json`.
> Adicione a URL do webhook do canal do Teams na secao `env`.

---

## Error Handling

| Situacao | Acao |
|---|---|
| Branch sem PR aberto (sem argumento) | "Nenhum PR encontrado para a branch atual. Use: `/taxone-review-pr <numero>`" |
| PR nao encontrado (com argumento) | "PR #{numero} nao encontrado. Verifique o numero e tente novamente." |
| Webhook nao configurado | Instruir a configurar `TEAMS_WEBHOOK_URL_TAXONE_EVOLUCAO_TECNICA` em settings.json |
| curl falha (timeout/conexao) | "Falha ao enviar para o Teams. Verifique a URL do webhook." |
| gh CLI nao autenticado | "Execute `gh auth login` para autenticar no GitHub CLI." |

---

## Configuracao Necessaria

### settings.json (~/.claude/settings.json)

Adicionar na secao `env`:

```json
{
  "env": {
    "TEAMS_WEBHOOK_URL_TAXONE_EVOLUCAO_TECNICA": "<URL_DO_WEBHOOK_DO_CANAL>"
  }
}
```

Para obter a URL do webhook:
1. No Teams, va ao canal desejado
2. Clique em "..." → "Workflows" (ou "Connectors")
3. Crie/selecione o workflow "Post to a channel when a webhook request is received"
4. Copie a URL gerada

---

## Skills Relacionadas

- `rt-ado-deploy` (global) — Skill similar que tambem envia Adaptive Card pro Teams
- `git-workflow` (global) — Workflow de branches e PRs
