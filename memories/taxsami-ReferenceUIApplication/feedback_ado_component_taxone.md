---
name: ADO Component sempre TaxOne
description: O campo Custom.Component no ADO para projetos TaxOne é SEMPRE "TaxOne" - nunca perguntar ao usuário
type: feedback
originSessionId: 0bc4802a-a312-49b1-9bdc-efee8e5fdaa1
---
Ao criar qualquer work item (User Story, Task, Bug) no ADO para projetos TaxOne, o campo `Custom.Component` deve ser sempre `"TaxOne"`.

**Why:** O usuário ficou frustrado por ter que informar isso manualmente. É um valor fixo que nunca muda para este contexto.

**How to apply:** Sempre preencher `Custom.Component: "TaxOne"` automaticamente em qualquer work item criado via skill taxone-fix ou manualmente. Nunca perguntar ao usuário qual é o Component.
