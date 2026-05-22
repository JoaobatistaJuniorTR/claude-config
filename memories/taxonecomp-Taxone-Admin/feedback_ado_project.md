---
name: ado-project-default
description: Defaults obrigatorios para Azure DevOps - projeto e Component - nunca perguntar ao usuario
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 628ee5d4-62d6-474d-be00-c7d6a3f44521
---

Sempre passar `project: "Mastersaf Fiscal Solutions"` em TODAS as chamadas ao Azure DevOps MCP.
Ao criar work items, sempre usar `Custom.Component: "TaxOne"`.
**Why:** O usuario se irrita quando o ADO pede o projeto repetidamente. O campo Component e obrigatorio e o valor padrao e "TaxOne".
**How to apply:** Em toda chamada mcp__azure-devops__*, incluir `project: "Mastersaf Fiscal Solutions"`. Ao criar US/Task/Bug, incluir `Custom.Component: "TaxOne"`.
