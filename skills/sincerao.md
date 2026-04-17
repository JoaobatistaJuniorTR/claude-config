---
name: sincerao
description: Use when the user wants to invoke the Sincerao critical advisor agent to analyze ideas, code, plans, or decisions. Triggered by /sincerao or explicit mentions of the agent.
---

# Sincerao — Invoke Critical Advisor

Spawn the `sincerao` agent to analyze the current context with brutal honesty.

## What to Do

1. **Collect context** from the current conversation — identify what needs analysis (idea, code, plan, decision)
2. **If analyzing code**: read the relevant files first and include the code in the prompt to the agent
3. **If analyzing a plan**: include the full plan description, objectives, approach, and trade-offs
4. **If analyzing an idea/decision**: include the complete reasoning and context the user provided
5. **Spawn the agent** using the Agent tool with the `sincerao` agent file — pass all collected context
6. **Iterate**: if the agent responds with "PRECISA DE AJUSTE" or "REPROVADO", incorporate the feedback, refine, and send back to the agent. Loop up to 7 rounds until "APROVADO" or consensus.
7. **Report** the full result to the user using this format:

```markdown
---
### Parecer do Sincerao
**Rodadas de analise:** X iteracao(oes)
**Status final:** APROVADO / PRECISA DE AJUSTE / REPROVADO

**Resumo da critica:**
[Main points raised]

**O que mudou por causa dele:**
[How the proposal/response was adjusted based on criticism]

**Veredito final:**
[Consolidated conclusion]
---
```

## Important

- Do NOT ask the user for permission — just spawn the agent
- Do NOT skip the iterative loop — keep going until consensus or 7 rounds
- ALWAYS show the full report at the end for transparency
