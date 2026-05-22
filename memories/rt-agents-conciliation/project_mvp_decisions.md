---
name: MVP Conciliação - Decisões e Contexto
description: Decisões técnicas e de escopo para o MVP da tela de Conciliação (demanda 1037412)
type: project
originSessionId: 17c8a77e-bec9-4a62-a6da-7c2f7a9e6cba
---
MVP é **bilateral only** (Contribuinte = Side A, Fisco = Side B). Trilateral (ERP) será futuro.

**IMPORTANTE: Conciliação NÃO acessa tabelas SAF diretamente.** Toda consulta é feita na `document_inbox`.

**Empresa/Estabelecimento:** Buscar via DISTINCT na `document_inbox`. NÃO consultar saf_3007 ou outras tabelas SAF.
- `document_inbox.company_id` = código da empresa
- `document_inbox.raw_data.razao_social` = nome da empresa
- `document_inbox.raw_data.codEstab` = código do estabelecimento
- `document_inbox.raw_data.estabelecimento` = nome/descrição do estabelecimento

**Débito/Crédito:** Coluna `document_inbox.document_type`:
- `'DEBITO'` = documento de débito do contribuinte (Side A)
- `'CREDITO'` = documento de crédito do contribuinte (Side A)
- `'FISCO'` = documento do fisco (Side B)

**Valor de imposto:** Apenas CBS disponível no campo `raw_data.valor_imposto`. IBS e IS **não calculados ainda**. MVP apenas CBS é aceitável.

**Tela sem sessão concluída:** Frontend mostra gráfico vazio e valores com traço (-). Conferir padrão no taxonert_frontend.

**Competência:** Sempre mês/ano. Há bug na IA gerando datas erradas (pede Jan/2026 e gera 25/12/2025-28/01/2026).

**Docs sem inconsistências:** Documentos iguais entre Contribuinte x Fisco = FULL_MATCH.

**Indicadores da tela:** Sempre calculados dos **docs conciliados** (sessões), nunca direto das bases.

**Múltiplas sessões mesma competência:** Usa a última sessão concluída.

**Autenticação:** JWT com tenant. Sem licenciamento. Padrão: rt-credito-api / rt-debito-api. Usa `jsonwebtoken` (já no package.json).

## Doc Regras v4.1 (04/05/2026) — Novas restrições MVP

**Escopo MVP restrito:** Apenas modelo 55 (NF-e), CST_IBS_CBS=000, CCLASS_IBS_CBS=000001, VLR_CBS>0.
- Débito = MOVTO_E_S=9 (Saída)
- Crédito = MOVTO_E_S<>9 (Entrada)
- Excluir: cancelados (SITUACAO=N), devolução (NORM_DEV<>01)

**Correspondência "Documentos Conciliados":** Doc exige 100% em 7 campos: modeloDFe, numeroDFe, dataDFeEmissao, niEmitente, niAdquirente, chaveDfe, valorCBSTotal. **Pendente validação com BA** se nosso matching progressivo (EXACT→RELAXED) atende ou se precisa ser estrito.

**Tela "Dados da operação" (novo):** ~20 campos incluindo Local da operação, Consumidor final, Origem/Destino, Seguro, Frete, Desconto. Muitos campos vêm de SAFX07/SAFX2064/SAFX04 que não ingerimos. **Pendente: está no escopo MVP?**

**IBS e IS:** "Manutenção futura" — apenas CBS tem regras definidas. Colunas IBS/IS podem ficar zeradas.

**Créditos API Fisco:** "A definir" — governo não disponibilizou API de testes.

**Monetização:** Sempre pt-BR, R$ 1.234,56, 2 casas decimais, em todos os campos de valor.

**Why:** Documento v4.1 MVP Regras v2 (18 pags) de 23/04/2026 por Danielle Batista (MFS-1100677). Complementar ao doc de telas.
**How to apply:** Usar essas decisões em toda implementação. NUNCA acessar tabelas SAF da conciliação. Perguntas críticas pendentes para BA antes de implementar filtros SAFX e tela "Dados da operação".
