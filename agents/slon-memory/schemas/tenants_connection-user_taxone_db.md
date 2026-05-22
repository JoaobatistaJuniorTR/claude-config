---
service: tenants_connection (via wsl docker exec tenant-provisioning-db)
schema: user_taxone_db
collected_at: 2026-04-30T10:00:00-03:00
postgres_version: "15.15"
environment: development (0 rows in all tables)
---

# Schema: user_taxone_db @ tenants_connection

## Infraestrutura
- Container: tenant-provisioning-db (postgres:15-alpine)
- Database: tenants_connection (NOT tenant_connections — esse é o catálogo)
- Schema: user_taxone_db
- Todas as tabelas com 0 live tuples (dev environment)

## Tabelas (18 total)

### Core — Conciliação Fiscal Trilateral
| Tabela | Colunas | PK | Propósito |
|--------|---------|-----|-----------|
| sessions | 20 cols | uuid | Sessão de conciliação (empresa, período, status, iterações) |
| profiles | 11 cols | uuid | Perfil de regras de matching (tolerâncias, key levels, fontes) |
| documents_conciliation | 18 cols | uuid | Documentos normalizados das 3 fontes (A/B/C) para matching |
| documents_inbox | 9 cols | uuid | Inbox de documentos brutos vindos de Kafka topics |
| match_results_trilateral | 18 cols | uuid | Resultado do matching trilateral (A-B, A-C, B-C, A-B-C) |
| iteration_logs | 12 cols | uuid | Log de cada iteração de matching |
| key_mappings | 13 cols | uuid | Mapeamento semântico de campos entre fontes e ERPs |
| session_prompts | 11 cols | uuid | Prompts de linguagem natural que ajustam regras de matching |

### Chat — Configuração de Sessão via Chat
| Tabela | Colunas | PK | Propósito |
|--------|---------|-----|-----------|
| session_chats | 7 cols | uuid | Chat conversacional para criar sessão de conciliação |
| session_chat_messages | 7 cols | uuid | Mensagens do chat de criação de sessão |

### Chat — Configuração de Perfil via Chat
| Tabela | Colunas | PK | Propósito |
|--------|---------|-----|-----------|
| profile_chat_sessions | 9 cols | uuid | Chat conversacional para configurar perfil de matching |
| profile_chat_messages | 9 cols | uuid | Mensagens do chat de configuração de perfil |

### SAF — Registros Tributários (Reforma Tributária IBS/CBS)
| Tabela | Colunas | PK | Propósito |
|--------|---------|-----|-----------|
| saf_3007 | ~108 cols | uuid | NF consolidada — cabeçalho fiscal com totais IBS/CBS/IS |
| saf_3008 | ~113 cols | uuid | NF item mercadoria — detalhe por item com impostos |
| saf_3009 | ~85 cols | uuid | NF item serviço — detalhe por item de serviço com impostos |
| saf_3042 | ~44 cols | uuid | Apuração cabeçalho — totais de apuração IBS/CBS por documento |
| saf_3043 | ~60 cols | uuid | Apuração item — detalhe de apuração por item |

### Controle
| Tabela | Colunas | PK | Propósito |
|--------|---------|-----|-----------|
| migrations | 3 cols | serial | TypeORM migrations log |

## Foreign Keys
```
profiles ←─── sessions (profile_id, NO ACTION)
sessions ←─── documents_conciliation (session_id, CASCADE)
sessions ←─── iteration_logs (session_id, CASCADE)
sessions ←─── match_results_trilateral (session_id, CASCADE)
sessions ←─── session_prompts (session_id, CASCADE)
documents_inbox ←─── documents_conciliation (inbox_id, SET NULL)
documents_conciliation ←─── match_results_trilateral (doc_a_id, SET NULL)
documents_conciliation ←─── match_results_trilateral (doc_b_id, SET NULL)
documents_conciliation ←─── match_results_trilateral (doc_c_id, SET NULL)
session_chats ←─── session_chat_messages (chat_id, CASCADE)
profile_chat_sessions ←─── profile_chat_messages (session_id, CASCADE)
```

## Modelo Trilateral (A/B/C)
- Source A = FISCAL_SOLUTION (Mastersaf/TAX ONE)
- Source B = FISCO_API (NF-e do governo, DFe)
- Source C = ERP (SAP, Oracle, JDE, EBS, OCLOUD)

## Check Constraints Notáveis
- source_code IN ('A', 'B', 'C')
- source_type IN ('FISCAL_SOLUTION', 'FISCO_API', 'ERP')
- match_status IN ('PENDING', 'MATCHED', 'UNMATCHED')
- erp_system IN ('SAP', 'ORACLE', 'JDE', 'EBS', 'OCLOUD')
- match_scope IN ('A_B', 'A_C', 'B_C', 'A_B_C')
- trilateral_status IN ('FULL_MATCH', 'FISCAL_MATCH', 'ERP_MATCH', 'EXTERNAL_MATCH', 'PARTIAL_AB/AC/BC', 'ALL_DIFFERENT', 'ORPHAN_A/B/C', 'PENDING')
- session status IN ('CREATED', 'LOADING', 'LOADING_SIDE_A/B/C', 'READY', 'RUNNING', 'PAUSED', 'ITERATION_COMPLETE', 'COMPLETED', 'ERROR', 'CANCELLED')

## Índices Notáveis (parciais)
- idx_docs_conciliation_session_pending: session_id WHERE match_status = 'PENDING'
- idx_docs_conciliation_erp: (session_id, erp_system) WHERE source_code = 'C'
- idx_match_results_review: session_id WHERE needs_review = true
- idx_match_results_doc_a/b/c: parciais com WHERE doc_X_id IS NOT NULL
- idx_saf_3007_movto_norm_docto: parcial para movto_e_s='9', norm_dev='1', cod_docto='55'
- idx_saf_3007_vlr_cbs: parcial WHERE vlr_cbs > 0
- idx_saf_3008/3009_cst_cclass_vlr: parcial para cst_ibs_cbs='001' AND cclass_ibs_cbs='000001' AND vlr_cbs > 0

## Tabelas SAF — sem FK para tabelas de conciliação
As tabelas saf_30XX não possuem foreign keys para sessions ou documents_conciliation.
São tabelas de staging/referência carregadas independentemente.
