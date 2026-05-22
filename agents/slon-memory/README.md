# Slon Memory

Memória persistente do agente Slon. Slon lê esta pasta ANTES de consultar o banco, mas não confia em dado velho.

## Estrutura

```
slon-memory/
├── schemas/       # snapshots de schema observado por conexão
├── decisions/     # ADRs gerados (decisões arquiteturais)
└── findings/      # achados duráveis sobre cada conexão
```

## Política Anti-Falsa-Memória (TTL)

- Todo arquivo tem **timestamp** ou **data de coleta** no frontmatter ou no conteúdo
- Slon lê memória ANTES de consultar `psql` (atalho de contexto)
- Mas: dado com mais de **30 dias** dispara aviso explícito ao usuário ("snapshot de DD/MM, pode estar defasado")
- Em decisão estrutural (modo Investigativo), Slon **revalida volumetria via `psql` antes de fechar recomendação**, mesmo que tenha snapshot recente

**Memória é atalho, não verdade.**

## Convenções de nomenclatura

### `schemas/<service-name>.md`
Snapshot do schema de uma conexão (`service-name` é o nome em `~/.pg_service.conf`).
Frontmatter:
```yaml
---
service: local-rt
collected_at: 2026-04-29T14:30:00-03:00
postgres_version: "15.15"
---
```

Conteúdo: tabelas, índices, FKs, volumetria observada, particionamento.

### `decisions/YYYY-MM-DD-<slug>.md`
ADR gerado. Slug curto e descritivo (ex: `2026-04-29-particionar-fato-apuracao.md`).
Estrutura: Contexto → Diagnóstico → Alternativas descartadas → Recomendação → Premissas que invalidam → Riscos → Próximos passos → Handoffs sugeridos.

### `findings/<service-name>.md`
Achados duráveis sobre o ambiente. Ex: "tabela X é write-heavy", "consulta Y faz seq scan diário às 03h".
Cada finding com timestamp de detecção.

## Não modificar manualmente sem necessidade

Slon escreve aqui automaticamente após análises relevantes. Edição manual é OK pra corrigir info errada, mas evite remover ADRs históricos — eles são auditáveis.
