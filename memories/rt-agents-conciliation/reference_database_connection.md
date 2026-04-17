---
name: Database Connection
description: Como conectar no PostgreSQL local via WSL docker
type: reference
---

Container: `tenant-provisioning-db` (roda via WSL)
Database: `tenants_connection`
User/Password: `postgres/postgres`
Porta: 5433

Comando:
```bash
wsl docker exec -i tenant-provisioning-db psql -U postgres -d tenants_connection -c "SQL_AQUI"
```

Schemas:
- `public` — tabela `tenant_connections`
- `user_taxone_db` — schema do tenant principal (tabelas de negócio)
- `tenant_provisioning` — provisioning

Tabelas em `user_taxone_db`: documents_inbox, documents_conciliation, sessions, key_mappings, iteration_logs, match_results_trilateral, profiles, profile_chat_sessions, profile_chat_messages, session_prompts, saf_3007/3008/3009/3042/3043, migrations
