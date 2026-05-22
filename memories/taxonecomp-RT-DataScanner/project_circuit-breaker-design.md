---
name: project_circuit-breaker-design
description: "Design completo do circuit breaker por tenant para RT-DataScanner, aprovado pelo Sincerão em 2 rodadas. PENDENTE de revisão — o gargalo real foi causado por erro de query (tabela inexistente), não por falha de conexão."
metadata: 
  node_type: memory
  type: project
  originSessionId: a58bd510-cd9f-4d26-9039-4869a7a92fb8
---

## Circuit Breaker por Tenant — RT-DataScanner

**US ADO**: #1122196 (Mastersaf Fiscal Solutions / Reforma Tributária)
**Status**: Design aprovado, implementação ADIADA. Precisa revisitar quais exceções ativam o CB.

**Why:** Um tenant com falhas persistentes causava gargalo no banco inteiro em produção, afetando todos os outros tenants. O scanner re-tentava o tenant problemático a cada ciclo (~5min) sem nenhum backoff.

**How to apply:** Quando for implementar, ler este documento completo + revisar a questão das query errors abaixo.

---

### PONTO CRÍTICO A REVISAR

O design atual define que **falhas de query individual por tabela NÃO ativam o circuit breaker** (porque o `QueryService` já faz continue per-table no catch, linha 77). Porém, o usuário relata que **o gargalo real em produção foi causado exatamente por erro de query — tabela não existia**.

Isso significa que:
1. O `QueryService` trata o erro (loga e continua), mas o **custo de tentar a conexão e executar queries em tabelas inexistentes** já é suficiente pra gerar gargalo
2. Ou o erro de tabela inexistente propagava de forma diferente do esperado (talvez não era catch silencioso)
3. Precisa investigar: o gargalo era no tempo de conexão? No tempo de query? Na quantidade de retries?

**Decisão pendente:** talvez query errors em tabelas específicas TAMBÉM devam incrementar o circuit breaker, ou pelo menos um subtipo deles (ex: tabela inexistente = estrutural, não transiente).

---

### Design Aprovado (Sincerão R2)

#### Estados
| Estado | Comportamento |
|--------|---------------|
| CLOSED | Tenant processado normalmente |
| OPEN | Tenant penalizado — não processado até expirar penalidade |
| HALF-OPEN | Uma tentativa cautelosa após expirar penalidade |
| DEAD | Permanentemente excluído — só reset manual |

#### Fluxo
1. Falha circuit-worthy → `INCR` atômico no Redis
2. Contador >= threshold (default 3) → OPEN
3. Backoff exponencial: 5min → 10min → 20min → cap 60min
4. Penalidade expirou → HALF-OPEN (lock atômico via `SET NX`)
5. Sucesso → CLOSED (reset total)
6. Falha → OPEN com penalidade maior
7. Após 5 ciclos open→half-open→open → DEAD

#### Exceções que ativam (PRECISA REVISÃO)
- **Ativam**: `SQLException`, `connection == null`, exceções de I/O
- **NÃO ativam** (QUESTIONAR): query errors per-table, config inválida
- Método: `isCircuitBreakerRelevant(Exception e)`
- IMPORTANTE: `obtainConnection` retorna null sem exceção — registrar falha ANTES do return

#### Operações Redis (atômicas)
- `INCR data_scanner:cb:failures:{tenantCode}`
- `SET data_scanner:cb:open:{tenantCode} {timestamp} EX {ttl}`
- `SET data_scanner:cb:halfopen:{tenantCode} 1 NX EX 60`
- `SET data_scanner:cb:dead:{tenantCode} true` (sem TTL)
- TTL de limpeza: 2x cap máximo (120min)

#### Código
- Criar `CircuitBreakerService` (classe separada)
- Check em `processTenantsForRange`, filtrar `assignedTenants`
- Registro de falha: catch de `processTenantInternal` + null check de `obtainConnection`
- Registro de sucesso: após `processTenantInternal` sem exceção

#### Endpoints REST
- `GET /circuit-breaker` — lista tenants não-closed
- `GET /circuit-breaker/{tenantCode}` — detalhes completos
- `POST /circuit-breaker/{tenantCode}/reset` — reset manual

#### Configurações
```properties
scanner.circuit-breaker.enabled=true
scanner.circuit-breaker.failure-threshold=3
scanner.circuit-breaker.initial-penalty-minutes=5
scanner.circuit-breaker.penalty-multiplier=2
scanner.circuit-breaker.max-penalty-minutes=60
scanner.circuit-breaker.dead-letter-cycles=5
```

#### Logging
- OPEN: WARN — `event=circuit_breaker_opened`
- Skipped: INFO — `event=circuit_breaker_skipped`
- HALF-OPEN: INFO — `event=circuit_breaker_half_open`
- CLOSED: INFO — `event=circuit_breaker_closed`
- DEAD: ERROR — `event=circuit_breaker_dead`
