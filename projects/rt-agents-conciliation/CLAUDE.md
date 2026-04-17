# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Visão Geral

**RTC Agents Conciliation** - Módulo de conciliação fiscal trilateral para o produto RTC (Reforma Tributária do Contribuinte) da Thomson Reuters. Utiliza agentes de IA para matching de documentos entre 3 origens:

| Lado | Origem | Descrição |
|------|--------|-----------|
| **A** | Solução Fiscal (MASTERSAF/SAFX) | Documentos do produto fiscal |
| **B** | API do Fisco (CBS/IBS) | Declarações governamentais |
| **C** | ERPs (SAP, Oracle, JDE, EBS, OCloud) | Registros contábeis/operacionais |

## Comandos Essenciais

```bash
# Desenvolvimento
npm install                    # Instalar dependências
npm run start:dev              # Hot-reload (porta 3001)
npm run start:debug            # Debug mode

# Testes
npm test                       # Unitários
npm run test:watch             # Watch mode
npm run test:e2e               # Integração (requer banco)
npm run test:cov               # Cobertura
npm test -- --testPathPatterns="nome-do-teste"  # Teste específico

# Qualidade
npm run lint                   # ESLint com fix
npm run format                 # Prettier

# Build
npm run build                  # Compilar para dist/
npm run start:prod             # Rodar produção

# Infraestrutura Docker
docker compose up -d           # Subir PostgreSQL + Kafka
docker compose ps              # Status dos containers
docker compose down            # Parar
docker compose down -v         # Parar e apagar dados

# Migrations Multi-Tenant
npm run migration:run          # Executar em todos os tenants
npm run migration:run:tenant dev_tenant   # Tenant específico
npm run migration:revert -- --tenant dev_tenant  # Reverter
npm run migration:generate -- src/infrastructure/database/migrations/tenant/NomeDaMigration
```

## Arquitetura

### Clean Architecture (Camadas)

```
src/
├── domain/                 # Entidades, interfaces de repositórios, exceções
├── application/            # Use cases, consumers Kafka, schedulers
├── infrastructure/         # TypeORM, Kafka, MCP tools, clientes externos
├── presentation/           # Controllers REST, DTOs, SSE gateways
└── modules/                # Módulos NestJS
```

### Multi-Tenant

- Estratégia: **schema por tenant** no PostgreSQL
- Banco compartilhado: `tenant_provisioning` (RDS Aurora)
- Header: `x-tenant-id` identifica o tenant
- Tabela `public.tenant_connections` gerencia configurações
- `TenantConnectionPoolService`: gerencia pool de DataSources por tenant

### MCP Tools (9 ferramentas)

| Tool | Descrição |
|------|-----------|
| `discover_key_mappings` | Descobre mapeamentos semânticos entre 3 origens |
| `confirm_key_mappings` | Persiste mapeamentos confirmados |
| `normalize_documents` | Normaliza docs usando mapeamentos confirmados |
| `execute_matching_iteration` | Matching por par (A_B, A_C, B_C) |
| `consolidate_trilateral` | Consolida matches em visão unificada |
| `interpret_user_prompt` | Interpreta instruções em linguagem natural |
| `analyze_trilateral_divergences` | Analisa padrões e riscos |
| `get_fiscal_knowledge` | Conhecimento tributário BR |
| `analyze_pending_documents` | Analisa documentos pendentes |

Para criar novo MCP Tool:
1. Criar arquivo em `src/infrastructure/mcp/tools/meu-tool.tool.ts`
2. Implementar interface `OnModuleInit` e registrar via `mcpServer.registerTool()`
3. Exportar em `src/infrastructure/mcp/tools/index.ts`
4. Adicionar como provider no módulo `src/modules/conciliation/conciliation.module.ts`

### Fluxo de Matching Trilateral

```
FASE 1: A↔B (6 iterações) → FASE 2: A↔C (6 iterações) → FASE 3: B↔C (6 iterações) → CONSOLIDAÇÃO
```

### Status Trilaterais

| Status | Significado | Severidade |
|--------|-------------|------------|
| `FULL_MATCH` | A=B=C | SUCCESS |
| `FISCAL_MATCH` | A=B, C diverge | WARNING |
| `ERP_MATCH` | A=C, B diverge | ERROR |
| `EXTERNAL_MATCH` | B=C, A diverge | ERROR |
| `ALL_DIFFERENT` | A≠B≠C | CRITICAL |
| `ORPHAN_A/B/C` | Documento isolado | WARN/CRIT/INFO |

## Tópicos Kafka

```
# Lado A - Solução Fiscal
{tenant}.input.taxone.received.v1

# Lado C - ERPs
{tenant}.input.erp_sap.received.v1
{tenant}.input.erp_oracle.received.v1
{tenant}.input.erp_jde.received.v1
{tenant}.input.erp_ebs.received.v1
{tenant}.input.erp_ocloud.received.v1
```

## Variáveis de Ambiente

Principais (ver `.env.example` para lista completa):

```env
PORT=3001
NODE_ENV=development
AUTH_ENABLED=false
DEFAULT_TENANT=dev_tenant

DATABASE_HOST=localhost
DATABASE_PORT=5433
DATABASE_NAME=tenant_provisioning
DATABASE_USERNAME=postgres
DATABASE_PASSWORD=postgres

KAFKA_BROKERS=localhost:9092

LITELLM_BASE_URL=https://litellm.int.thomsonreuters.com
LITELLM_API_KEY=
LITELLM_MODEL=claude-sonnet-4-6
```

## Projetos Relacionados

| Projeto | Descrição |
|---------|-----------|
| `rt-frontend-conciliation` | Frontend Angular 19 (trilateral) |
| `rt-input-service-api` | Worker Kafka para ingestão (compartilha banco) |

## URLs Locais

- **API**: http://localhost:3001
- **Swagger**: http://localhost:3001/api
- **Health**: http://localhost:3001/health
- **Frontend** (se rodando): http://localhost:4200

## Makefile

Use `make help` para ver todos os comandos disponíveis. Atalhos úteis:

```bash
make up         # Sobe infraestrutura + inicia dev
make down       # Para tudo
make test       # Roda testes
make lint       # ESLint
make db-shell   # Shell PostgreSQL
```
