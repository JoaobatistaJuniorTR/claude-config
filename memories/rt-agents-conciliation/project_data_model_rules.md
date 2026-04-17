---
name: Regras de dados documents_inbox
description: Como document_type, source_key e classification devem ser preenchidos na documents_inbox e documents_conciliation
type: project
---

## documents_inbox

- **document_type**: sempre `DEBITO` ou `CREDITO`, tanto para lado A (contribuinte) quanto lado B (fisco)
- **source_key**: para o fisco (lado B), preencher com `numDocfis` do `raw_data`
- **source_topic**: identifica a origem — `*taxone*` = contribuinte (lado A), `*fisco*` = fisco (lado B)
- Para o fisco, `document_type` deve estar também no `raw_data` como campo `document_type`

## documents_conciliation

- **classification**: deve ser `DEBITO` ou `CREDITO` (derivado de `raw_data.document_type` ou `document_type` da inbox). NUNCA usar `FISCO` como classification.
- A distinção contribuinte vs fisco vem do `source_code` (A ou B), não da classification.

## Cenário Janeiro/2026 (EMP001)

**Débitos:**
- 1001: match exato (A=B, R$ 2.150)
- 1002: divergência valor (A=R$ 4.800,75 vs B=R$ 4.750)
- 1003: match exato (A=B, R$ 1.875,30)
- 1004: órfão fisco (só B, R$ 3.500)

**Créditos:**
- 000001: match exato (A=B, R$ 750)
- 000002: match exato (A=B, R$ 150)
- 000003: divergência valor (A=R$ 850 vs B=R$ 900)
- 000005: órfão contribuinte (só A, R$ 99.999,99)
- 000006: órfão contribuinte (só A, R$ 0,01)

**Why:** Definição consolidada após ajustes na base para garantir que gráficos de débitos e créditos no dashboard funcionem corretamente.

**How to apply:** Ao ingerir dados ou criar simulações, sempre seguir essas regras. Nunca classificar como FISCO — usar DEBITO/CREDITO e distinguir pelo source_code.
