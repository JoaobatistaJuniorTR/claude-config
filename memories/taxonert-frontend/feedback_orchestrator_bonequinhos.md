---
name: Orchestrator como roteiro, nao agente
description: saffron-orchestrator deve ser lido e executado pela sessao principal, nao spawnado como Agent, para que cada fase gere bonequinho separado
type: feedback
---

O saffron-orchestrator NAO deve ser spawnado como Agent. Deve ser lido e seguido pela sessao principal diretamente.

**Why:** Quando o orchestrator e spawnado como Agent, ele cria UM bonequinho. Os sub-agentes que ele spawna internamente (designer, specialist, developer, auditor) rodam como sub-sub-processos dentro dele e NAO criam bonequinhos proprios. O usuario quer ver cada fase como um bonequinho separado no UI.

**How to apply:** Quando o usuario referencia `@saffron-orchestrator`, a sessao principal le o arquivo e segue as instrucoes diretamente, spawnando cada fase como um Agent separado via tool call. Cada Agent call direta da sessao principal cria seu proprio bonequinho visivel. O orchestrator e um roteiro/script, nao um agente.
