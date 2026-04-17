---
name: MVP Conciliação - Decisões e Contexto
description: Decisões técnicas e de escopo para o MVP da tela de Conciliação (demanda 1037412)
type: project
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

**Why:** Esclarecimentos do PO/dev em 2026-03-31 (segunda rodada de feedback).
**How to apply:** Usar essas decisões em toda implementação. NUNCA acessar tabelas SAF da conciliação.
