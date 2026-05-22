---
name: rt-ado-deploy
description: Creates a User Story in Azure DevOps for deploying a RT service artifact to QA, with JFrog artifact link. Invoked via /rt-ado-deploy <artifact> <version>.
---

# /rt-ado-deploy — Deploy Task para projetos RT (Reforma Tributaria)

Cria uma User Story no Azure DevOps para deploy de um artefato da Reforma Tributaria no ambiente de QA, com link para o artefato no JFrog.

## Invocacao

```
/rt-ado-deploy <artefato> <versao> [ambiente]
```

**Exemplos:**
```
/rt-ado-deploy portal 1.0.3
/rt-ado-deploy portal 1.0.3 HML
/rt-ado-deploy rt-credito-api 1.0.1 PROD
/rt-ado-deploy rt-obrigacao-api 2.4.0
```

---

## Input Parsing

Parse os argumentos:

1. Se menos de 2 argumentos → informar: "Usage: `/rt-ado-deploy <artefato> <versao> [ambiente]`" e PARAR.
2. Primeiro argumento = nome do artefato
3. Segundo argumento = versao (formato X.Y.Z)
4. Terceiro argumento (opcional) = ambiente. Se nao informado, assume `QA`
   - Valores aceitos: `QA`, `HML`, `PROD` (case-insensitive, converter para uppercase)
5. Validar que a versao segue o formato semver basico (digitos separados por pontos)

---

## Fase 1: Montar URL do JFrog

**Base URL:** `https://tr1.jfrog.io/artifactory/generic-local/reforma-tributaria`

**Regra de mapeamento de pasta:**

| Artefato | Pasta no JFrog | Arquivo |
|----------|---------------|---------|
| `portal` | `frontend/` | `portal-{versao}.zip` |
| Qualquer outro | `{artefato}/` | `{artefato}-{versao}.zip` |

**Construir a URL:**

```
Se artefato == "portal":
  URL = https://tr1.jfrog.io/artifactory/generic-local/reforma-tributaria/frontend/portal-{versao}.zip
Senao:
  URL = https://tr1.jfrog.io/artifactory/generic-local/reforma-tributaria/{artefato}/{artefato}-{versao}.zip
```

**Apresentar ao usuario:**

> Artefato: `{artefato}` v`{versao}`
> JFrog: `{url}`
>
> Criando User Story no ADO...

---

## Fase 2: Criar User Story no ADO

**Projeto:** Mastersaf Fiscal Solutions (fixo)

**Step 1 — Validar ambiente:**

Verificar que as variaveis ADO estao configuradas conforme `ado-client` skill (ADO_PAT, ADO_ORG, ADO_PROJECT).

**Step 2 — Criar User Story:**

Use `mcp__azure-devops__wit_create_work_item` com:

- **Work Item Type:** `User Story`
- **System.Title:** `[TAXREFORM][{ambiente}] - Atualização do {artefato}`
- **System.AreaPath:** `Mastersaf Fiscal Solutions\MFS\TAX ONE\TechOps`
- **System.AssignedTo:** `de Souza, Alexandre Eduardo`
- **Custom.Component:** `TaxOne`
- **Custom.RequestType:** `Apoio - Interno`
- **System.Description** (formato Markdown):

```markdown
## Deploy para QA

**Artefato:** {artefato}
**Versao:** {versao}
**Ambiente:** QA

### Link do Artefato
[{artefato}-{versao}.zip]({url_jfrog})

---
*Work item gerado via Claude Code — /rt-ado-deploy*
```

**Step 3 — Confirmar ao usuario:**

> User Story **#{ID}** criada no ADO!
> **Titulo:** [TAXREFORM][{ambiente}] - Atualização do {artefato}
> **Assigned To:** de Souza, Alexandre Eduardo
> **Area:** Mastersaf Fiscal Solutions\MFS\TAX ONE\TechOps
> **Link ADO:** {link_para_work_item}
> **Artefato JFrog:** {url_jfrog}

---

## Fase 3: Notificar no Teams

Envia uma mensagem no canal do Teams com @mention para o Alexandre.

**Prerequisito:** Variavel `TEAMS_WEBHOOK_URL_TAXREFORM_DEV_TECHOPS` configurada em `~/.claude/settings.json` com a URL do Incoming Webhook do canal.

**Step 1 — Validar webhook:**

```bash
if [ -z "$TEAMS_WEBHOOK_URL_TAXREFORM_DEV_TECHOPS" ]; then
  echo "AVISO: TEAMS_WEBHOOK_URL_TAXREFORM_DEV_TECHOPS nao configurada. Notificacao no Teams pulada."
  echo "Configure em ~/.claude/settings.json para habilitar."
fi
```

Se nao configurada, **apenas avisar e continuar** (Teams e opcional, ADO e obrigatorio).

**Step 2 — Enviar Adaptive Card com @mention:**

**IMPORTANTE:** Usar `--data-binary` com heredoc e `charset=utf-8` no header. Evitar acentos (ã, ç, é) no JSON — usar equivalentes ASCII (ex: "Atualizacao" em vez de "Atualização") para evitar erros de encoding do Power Automate.

```bash
curl -s -H "Content-Type: application/json; charset=utf-8" --data-binary @- "$TEAMS_WEBHOOK_URL_TAXREFORM_DEV_TECHOPS" <<'JSONEOF'
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
          "text": "🚀 [TAXREFORM][{ambiente}] - Atualização do {artefato}"
        },
        {
          "type": "TextBlock",
          "text": "<at>Alexandre</at>, nova solicitação de deploy criada.",
          "wrap": true
        },
        {
          "type": "FactSet",
          "facts": [
            { "title": "Artefato:", "value": "{artefato} {versao}" },
            { "title": "Ambiente:", "value": "QA" },
            { "title": "User Story:", "value": "#{ID}" }
          ]
        },
        {
          "type": "TextBlock",
          "text": "[Abrir no ADO]({link_work_item}) | [Baixar Artefato]({url_jfrog})",
          "wrap": true
        }
      ],
      "msteams": {
        "entities": [{
          "type": "mention",
          "text": "<at>Alexandre</at>",
          "mentioned": {
            "id": "AlexandreEduardo.deSouza@thomsonreuters.com",
            "name": "Alexandre Eduardo de Souza"
          }
        }]
      }
    }
  }]
}' "$TEAMS_WEBHOOK_URL_TAXREFORM_DEV_TECHOPS"
```

**Step 3 — Confirmar ao usuario:**

Se enviou com sucesso:
> Notificacao enviada no Teams com @Alexandre.

Se webhook nao configurado:
> Teams pulado (TEAMS_WEBHOOK_URL_TAXREFORM_DEV_TECHOPS nao configurada).

---

## Error Handling

| Situacao | Acao |
|---|---|
| ADO API retorna 401 | PAT expirado. Dizer: "Atualize o ADO_PAT em ~/.claude/settings.json" |
| ADO API retorna 403 | PAT sem permissao. Dizer: "O PAT precisa do scope Work Items (Read, Write)" |
| Versao em formato invalido | Dizer: "Versao '{input}' nao parece valida. Use o formato X.Y.Z (ex: 1.0.3)" |
| Argumentos faltando | Mostrar usage e PARAR |

---

## Skills Relacionadas

- `ado-client` (global) — Padroes de API do ADO referenciados neste skill
- `rt-fix` (global) — Workflow de fix que pode sugerir este skill ao final
