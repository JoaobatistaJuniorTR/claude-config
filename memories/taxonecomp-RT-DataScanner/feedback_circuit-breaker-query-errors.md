---
name: feedback_circuit-breaker-query-errors
description: "O gargalo real em produção do DataScanner foi causado por erro de query (tabela inexistente), não por falha de conexão. O design do circuit breaker precisa considerar isso."
metadata: 
  node_type: memory
  type: feedback
  originSessionId: a58bd510-cd9f-4d26-9039-4869a7a92fb8
---

O incidente de produção que motivou o circuit breaker foi causado por **erro de query em tabela inexistente** em um tenant, não por falha de conexão/SQL genérica.

**Why:** O design inicial do CB assumia que query errors per-table são "seguros" porque o `QueryService` faz catch+continue (linha 77). Mas o custo acumulado de tentar queries em tabelas inexistentes repetidamente já era suficiente para gerar gargalo no banco inteiro.

**How to apply:** Ao implementar o circuit breaker ([[project_circuit-breaker-design]]), investigar se query errors (especialmente estruturais como tabela inexistente) devem ativar o CB. Possível distinção: erros estruturais (tabela não existe, permissão negada) vs erros transientes (timeout, deadlock). Estruturais podem merecer CB; transientes já são tratados.
