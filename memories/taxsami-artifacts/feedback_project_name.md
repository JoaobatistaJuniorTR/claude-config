---
name: Projeto ADO é Mastersaf Fiscal Solutions
description: O projeto no Azure DevOps para taxsami_artifacts é sempre "Mastersaf Fiscal Solutions" - nunca perguntar ou omitir
type: feedback
originSessionId: 68a80cd7-2f31-438e-82a6-74920f8c1155
---
Sempre usar "Mastersaf Fiscal Solutions" como nome do projeto no Azure DevOps para qualquer operação com work items no contexto do taxsami_artifacts.

**Why:** O usuário já informou isso múltiplas vezes e ficou frustrado por ter que repetir. A informação está em memórias e contextos anteriores.

**How to apply:** Em toda chamada de API do ADO (wit_get_work_item, wit_update_work_item, etc.), sempre passar `project: "Mastersaf Fiscal Solutions"` sem perguntar.
