---
name: ADO tasks em PT-BR
description: Tasks no ADO devem ser criadas em português brasileiro, não em inglês
type: feedback
originSessionId: 01376094-cd28-47c3-ae08-f9f3faac92ff
---
Tasks criadas no Azure DevOps devem ter título e descrição em **português brasileiro**.

**Why:** Convenção do time — toda comunicação no ADO (tasks, comments, bugs) é em PT-BR. Inglês é só para código, commits e branches.

**How to apply:** Ao usar `wit_create_work_item` ou `wit_add_child_work_items`, escrever `System.Title` e `System.Description` em PT-BR. Commits e branches continuam em inglês.
