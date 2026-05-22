---
name: rt-input-service-gap
description: 5 campos faltando no rt-input-service que impedem matching completo e exibição conforme requisitos
metadata: 
  node_type: memory
  type: project
  originSessionId: 5e0ade3b-dd47-4cba-af88-901d4df92dd2
---

## Gap: rt-input-service-api não envia 5 campos obrigatórios para conciliação

**Why:** O documento de requisitos (MTZ_T1_CONCILIACAO v4 MVP) exige colunas nas grades de inconsistências que dependem de campos que o rt-input-service não inclui nos SELECTs de `fetchCreditoData()` e `fetchDebitoData()` do `documents-inbox.repository.ts`. Sem eles, 3 dos 7 key_mappings de matching são impossíveis de avaliar para Side A.

**How to apply:** Ao retomar, ajustar os SELECTs no rt-input-service, depois alinhar o seed e os repositórios do rt-agents-conciliation.

### 5 campos que PRECISAM ser adicionados ao rt-input-service

| Campo SAF_3007 | Nome no raw_data | Para que serve | Impacto |
|---|---|---|---|
| `cnpj_cpf_pfpj` | `cnpjCpfPfpj` | CNPJ Emitente/Fornecedor | matching key `ISSUER_NI` + coluna grid |
| `cod_docto` | `codDocto` | Tipo Documento (55=NF-e) | matching key `DOC_TYPE` + coluna grid |
| `desc_chave_dfe` | `descChaveDfe` | Chave Acesso 44 dígitos | matching key `ACCESS_KEY` + coluna grid |
| `vlr_tot_ibs` | `vlrTotIbs` | Valor IBS total | matching key `AMOUNT_IBS` + exibição CBS/IBS/IS |
| `vlr_tot_is` | `vlrTotIs` | Valor IS total | exibição CBS/IBS/IS separados |

### Onde alterar no rt-input-service

Arquivo: `src/infrastructure/database/repositories/documents-inbox.repository.ts`
- `fetchCreditoData()` (linha ~169): adicionar 5 campos no SELECT
- `fetchDebitoData()` (linha ~84): adicionar 5 campos no SELECT
- Interfaces `CreditoQueryRow` e `DebitoQueryRow`: adicionar campos
- Mapeamento de retorno (linhas ~137-149 e ~215-232): incluir novos campos

### Depois no rt-agents-conciliation

1. **Ajustar seed-inbox.sql** para compliance com campos reais (remover `cpfCgc`, `codFisJur` fantasmas; usar `cnpjCpfPfpj`, `codDocto`, `descChaveDfe`)
2. **Fix repositories** (`document-inbox.repository.ts` linhas ~482, `match-result.repository.ts` linha ~265): ler `cnpjEmpresa` como acquirerCnpj e `cnpjCpfPfpj` como supplierCnpj para Side A
3. **Fix key_mappings padrão** na migration: `ISSUER_NI` field_side_a deve ser `cnpjCpfPfpj` (não `COD_FIS_JUR`), `ACQUIRER_NI` field_side_a deve ser `cnpjEmpresa` (não `COD_FIS_JUR`)

### Contexto do levantamento (2026-05-20)

- Todos os 5 campos existem na tabela `saf_3007` do rt-input-service (verificado na entity ORM)
- Hoje o matching funciona com apenas 4 dos 7 campos: DOC_NUMBER, EMISSION_DATE, AMOUNT_CBS, ACQUIRER_NI
- Branch de trabalho: `fix/data-driven-matching-engine`
- Referência: [[project_data_model_rules]]
