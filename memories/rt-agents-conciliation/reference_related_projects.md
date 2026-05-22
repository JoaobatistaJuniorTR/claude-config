---
name: Projetos Relacionados RTC
description: Projetos de referência para padrões de implementação (débito, crédito, input service, frontend)
type: reference
---

- **rt-credito-api**: API de créditos. Referência para extração de dados de crédito do document_inbox JSONB e padrão de autenticação JWT.
- **rt-debito-api**: API de débitos. Referência para extração de dados de débito do document_inbox JSONB e padrão de autenticação JWT.
- **rt-input-service-api**: Worker Kafka que ingesta dados do TaxOne e grava nas tabelas. Compartilha banco com rt-agents-conciliation.
- **rt-frontend**: Frontend Angular 19 do RTC. NÃO é rt-frontend-conciliation. Repositório correto: rt-frontend.
- **rt-tenant-provisioning**: Gerencia tenants e conexões.

**How to apply:** Consultar rt-credito-api e rt-debito-api para padrões de JWT, extração de valores, e classificação débito/crédito.
