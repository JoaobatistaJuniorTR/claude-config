---
name: Sempre usar agentes do orquestrador quando referenciado
description: Quando o usuario menciona @saffron-orchestrator, DEVE seguir o pipeline de agentes (Designer → Specialist → Developer → Auditor), nunca implementar direto
type: feedback
---

Quando o usuario referencia `@saffron-orchestrator`, seguir o pipeline completo de agentes especializados definido nele.

**Why:** O usuario estruturou um sistema multi-agente com papeis especializados (designer, specialist, developer, auditor). Implementar direto bypassa toda essa arquitetura e perde os beneficios de cada fase (blueprint, mapeamento Saffron, auditoria A1-A10).

**How to apply:** Ao ver `@saffron-orchestrator` ou `.claude/agents/saffron-orchestrator.md`:
1. Detectar o cenario (figma, image, prompt, migration)
2. Criar o time com TeamCreate
3. Despachar agentes na ordem correta via Agent tool
4. Pausar para validacao do usuario entre fases
5. NUNCA pular a fase de auditoria
