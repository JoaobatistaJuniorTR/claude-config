---
name: Ecossistema Evolução Técnica
description: Contexto sobre os 46 componentes do ecossistema TaxOne — libs, serviços, grupos, dependências
type: project
---

**Ecossistema:** Time Evolução Técnica (NÃO é Reforma Tributária).

**taxone_framework** — mono-repo com 11 libs internas:
multitenancy, multi-tenancy (JPA), multi-tenancy-strategy-schema, queue-configurator-activemq, cacheator, externallogger, krypto, lonestar, metrics-analyzer, redis-configurator, t1framework

**t1shared** — lib compartilhada entre módulos t1 (t1dw, t1icms, t1piscofins, t1jobservidor). Tem chamadas de lista de empresas/estabelecimentos usadas por todos os módulos. Quem depende de t1shared provavelmente depende também do t1framework.

**Grupos:**
- reinf: 5 repos (producer, sender, source, core, utils)
- ecf: 4 repos (core, sender, producer, utils)
- obi: 1 repo (consumer)

**Repos sem pom.xml:**
- taxonecomp_Globalscape-integrator → .NET
- taxone_obi_consumer_poc → Go/Node

**Deprecated:** FileReset (taxonecomp_FileHandler-PermissionReset)

**How to apply:** Ao analisar impacto de mudanças em t1shared, considerar que afeta todos os módulos t1*. Mudanças em libs do framework podem afetar potencialmente todos os 34 serviços.
