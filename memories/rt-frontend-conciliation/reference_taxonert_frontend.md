---
name: Projeto principal é taxonert_frontend, não rt-frontend-conciliation
description: O projeto de produção da conciliação fiscal fica em C:/repositories/taxonert_frontend/projects/rt/, não no rt-frontend-conciliation que é um protótipo separado.
type: reference
---

O projeto de produção é `C:/repositories/taxonert_frontend` (monorepo Angular com 3 projetos: home, rt, shared).

A feature de conciliação fica em: `projects/rt/src/app/features/conciliacao/`

**Why:** O usuário corrigiu quando implementei no projeto errado (rt-frontend-conciliation). O taxonert_frontend é o projeto real de produção que usa Saffron Design System v3.0.0 com CUSTOM_ELEMENTS_SCHEMA.

**How to apply:** Sempre confirmar ou assumir que implementações de telas do Figma do Tax One devem ser feitas no taxonert_frontend, subprojeto rt.
