---
name: Campos obrigatorios ADO para fechar work items
description: Valores e campos obrigatorios ao fechar tasks e user stories no Azure DevOps - Custom.ResolutionType e Custom.Component
type: reference
originSessionId: 1aa20dbe-90d7-45c2-a0c7-52aaeaacbef3
---
Ao fechar work items no projeto **Mastersaf Fiscal Solutions**, os seguintes campos sao obrigatorios:

- **Custom.ResolutionType**: Obrigatorio para fechar Tasks e User Stories. Valores comuns: "Done", "No further action required"
- **Custom.Component**: Obrigatorio ao CRIAR Tasks e User Stories. Valor mais usado na area de RT: "TAX ONE"
- **Custom.RequestType**: Obrigatorio ao CRIAR User Stories. Valores comuns: "Sustentação" (para fixes/manutenção), "Evolução" (para features novas)

O campo `Custom.ResolutionType` nao aceita "Completed" — usar "Done" como padrao.

## Campos obrigatórios na CRIAÇÃO de User Story
Confirmado em 2026-05-21 (US 1131079): `Custom.Component` e `Custom.RequestType` são obrigatórios na criação. Sem eles, retorna TF401320 "Rule Error for field ... Required, HasValues, AllowsOldValue, InvalidEmpty".

## Remaining Work em tasks fechadas

Ao fechar uma Task, o campo **RemainingWork** é automaticamente limpo pelo ADO (regra do processo).
- NAO tentar setar RemainingWork ao fechar — causa erro "InvalidNotEmpty"
- NAO tentar setar RemainingWork=0 antes de fechar — tambem causa erro
- O fluxo correto é: criar a task com CompletedWork e OriginalEstimate, e ao fechar o ADO limpa o RemainingWork sozinho
- Resultado final: OriginalEstimate=X, CompletedWork=X, RemainingWork=vazio (nulo)
