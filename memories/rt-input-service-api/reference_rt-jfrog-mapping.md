---
name: reference_rt-jfrog-mapping
description: Mapeamento completo dos 6 repos RT para caminhos no JFrog (reforma-tributaria)
metadata: 
  node_type: memory
  type: reference
  originSessionId: 39bbd102-7bc1-4104-85a5-83eacaa8de31
---

Base URL: `https://tr1.jfrog.io/artifactory/generic-local/reforma-tributaria`

| Repo GitHub | Pasta JFrog | Nome do artefato |
|-------------|-------------|------------------|
| rt-frontend | `frontend/` | `portal-{versao}.zip` |
| rt-input-service-api | `rt-input-service/` | `rt-input-service-api-{versao}.zip` |
| rt-agents-conciliation | `rt-agents-conciliation/` | `rt-agents-conciliation-{versao}.zip` |
| rt-credito-api | `rt-credito-api/` | `rt-credito-api-{versao}.zip` |
| rt-debito-api | `rt-debito-api/` | `rt-debito-api-{versao}.zip` |
| rt-tenant-provisioning | `rt-tenant-provisioning/` | `rt-tenant-provisioning-{versao}.zip` |

**Notas:**
- O repo `rt-input-service-api` publica na pasta `rt-input-service/` (sem `-api`), mas o zip se chama `rt-input-service-api-{versao}.zip` (com `-api`).
- O frontend usa pasta `frontend/` e nome `portal-{versao}.zip`, diferente dos backends.
- Todos os 5 backends (NestJS) empacotam `dist/` + `package.json` + `package-lock.json`.
- O frontend empacota apenas os arquivos do Angular build (`dist/portal/browser/*`).
