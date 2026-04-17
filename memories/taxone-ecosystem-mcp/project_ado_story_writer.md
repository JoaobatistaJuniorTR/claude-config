---
name: Projeto ADO Story Writer
description: MCP server TypeScript que ajuda escrever histórias ADO com análise de impacto automática no ecossistema TaxOne
type: project
---

Projeto taxone-ecosystem-mcp: MCP server + bootstrap CLI + skill /write-story.

**Status:** MVP implementado (2026-04-15). 44 testes passando, 46 componentes na knowledge base.

**Arquitetura:**
- KnowledgeStore → carrega markdowns com frontmatter YAML
- GraphBuilder → grafo de dependências in-memory (libs, filas, serviços)
- 5 MCP tools: search_services, get_service_details, get_impact_analysis, get_dependency_graph, rescan
- Bootstrap CLI: lê seed.yaml + wiki + pom.xml → gera manifests

**Why:** PO não escrevia histórias direito, clonava e alterava título sem contexto. Essa tool identifica repos impactados automaticamente.

**How to apply:** Qualquer evolução no projeto deve manter os 44 testes passando. O bootstrap pode ser re-rodado a qualquer momento com `npm run bootstrap`.
