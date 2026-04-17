---
name: Plano matchTypeLabel pendente
description: Plano aprovado para adicionar matchTypeLabel nas respostas da API — backend retorna label amigável junto com matchType técnico, preparando para profiles customizáveis
type: project
---

Plano para propagar `matchTypeLabel` do backend ao frontend, eliminando mapa hardcoded no front.

**Why:** Os nomes técnicos (EXACT, RELAXED_NO_CODE, etc.) são do profile. Quando usuário customizar profiles, o label definido por ele deve fluir naturalmente. Frontend não deve manter mapa fixo.

**How to apply:** O plano completo está salvo em `.claude/plans/starry-discovering-oasis.md`. Resumo das 9 etapas:

1. Criar utility `src/shared/utils/match-type-labels.ts` com `resolveMatchTypeLabel()` e `buildMatchTypeLabelMap()`
2. Testes da utility
3. Adicionar `label?: string` no `KeyLevel` interface e nos `DEFAULT_KEY_LEVELS`
4. Atualizar DTOs dashboard (`matchTypeLabel`, `byMatchTypeLabels`)
5. Atualizar dashboard controller (3 endpoints)
6. Atualizar interface SSE `MatchingEvent`
7. Atualizar emissão de progresso no use case
8. Atualizar models do frontend (BackendIteration, BackendMatchResult, BackendDashboard)
9. Atualizar service do frontend para usar `matchTypeLabel` do backend com fallback

**Status:** Pendente de implementação (2026-04-08).
