---
name: NUNCA apagar documents_inbox
description: A tabela documents_inbox contém dados reais ingeridos via Kafka/rt-input-service que NÃO podem ser recriados. Jamais incluir em TRUNCATE, DELETE ou qualquer operação destrutiva.
type: feedback
---

**NUNCA apagar dados da tabela `documents_inbox`.**

**Why:** Os dados do `documents_inbox` são ingeridos externamente via Kafka pelo `rt-input-service-api` e NÃO têm seed, backup local, nem forma fácil de recriar. Uma vez apagados, o usuário precisa re-ingerir manualmente — processo demorado e trabalhoso. O endpoint de simulação (`/simulation/generate`) gera dados sintéticos com formato diferente e NÃO substitui os dados reais.

**How to apply:** Quando o usuário pedir para "apagar dados" ou "limpar o banco":
1. NUNCA incluir `documents_inbox` em TRUNCATE/DELETE
2. Perguntar explicitamente quais tabelas podem ser apagadas
3. Tabelas seguras para limpar: `sessions`, `documents_conciliation`, `match_results_trilateral`, `iteration_logs`, `key_mappings`, `session_prompts`, `session_chats`, `session_chat_messages`
4. Tabelas que NUNCA devem ser apagadas sem confirmação explícita: `documents_inbox`, `profiles`
5. Na dúvida, SEMPRE perguntar antes de qualquer operação destrutiva no banco
