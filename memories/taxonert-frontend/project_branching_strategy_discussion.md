---
name: branching-strategy-model-c
description: Modelo C (release-centric) APROVADO pelo time em 2026-05-12. Migração concluída em todos os repos backend (2026-05-13). Frontend aguarda merge da release/1.0.4.
metadata: 
  node_type: memory
  type: project
  originSessionId: e6e32652-7776-4815-bd6b-227db97b4983
---

## Status: APROVADO — Modelo C (release-centric) adotado (2026-05-12)

## Contexto

Após a release v1.0.3 (primeira release real do gitflow), vários problemas operacionais surgiram:
- Workflow `on-release-merged.yml` falhava no bump por regra de signed commits da empresa
- Merge-back deu problema 2x (check falhado contaminava PRs)
- Bump na develop adicionava cerimônia sem valor claro

Isso levou a uma reavaliação completa da estratégia de branching.

## Fluxo de QA real do time

1. Dev cria feature branch → desenvolve → mergeia na develop via PR
2. Deploy da develop pro QA
3. QA testa, acha bugs → abre bug no ADO → dev corrige na develop
4. Quando QA valida → corta release branch da develop
5. Fixes de QA vão na release branch
6. Deploy da release pro QA (reteste)
7. QA aprova → pacote da release vai pra produção
8. Release mergeia na main → tag

## Três modelos avaliados

### Modelo A — Gitflow atual (com develop)
- main = produção, develop = integração
- Custo: merge-back, bump pós-release, complexidade
- Vantagem: branch contínua (develop) independente de versão

### Modelo B — Sem develop, main como integração (DESCARTADO)
- Descartado porque QA testa de branch específica, não da main
- Main teria código não validado pelo QA

### Modelo C — Release-centric (RECOMENDADO pelo Sincerão, 2x APROVADO)
```
1. Main = produção (protegida por tags)
2. Criar release/X.Y.Z da main (bump aqui)
3. Devs criam feature branches da release → PR → release
4. Deploy da release pro QA
5. QA testa, bugs corrigidos na release
6. QA aprova → release mergeia na main → tag → produção
7. Próximo ciclo: nova release da main
```

Benefícios:
- Sem merge-back, sem bump pós-release
- Main limpa (só código que passou por QA)
- Hotfix simples (corta da tag, merge na main)
- Workflow pós-release: só tag + GitHub Release
- Formaliza o que já acontece (features já iam direto na release/1.0.3)

## Trade-off central (motivo da decisão pendente)

**Develop (Modelo A):** branch contínua, independente de versão. Se release N+1 depende de algo em QA na N, já está na develop. Não trava.

**Modelo C:** release É o ambiente de trabalho. Se N+1 depende de código em QA na N, precisa esperar N mergear na main ou fazer merge manual de release/N na release/N+1.

**Pergunta pro time:** com que frequência uma feature nova depende de algo que ainda está em QA?
- Quase nunca → Modelo C funciona
- Frequentemente → develop tem valor

## Cenários de borda analisados

### Hotfix com release ativa
- Corta da tag, fix, merge na main, merge de main na release ativa
- Funciona melhor que no gitflow (um merge a menos)

### Feature reprovada pelo QA
- `git revert` na release (igual em qualquer modelo)
- Feature flags pra features arriscadas

### Releases sobrepostas (N e N+1 simultâneas)
- Cria release/N+1 da main
- Quando N mergeia, faz merge de main na N+1
- Precisa documentar como protocolo

## Migração concluída — repos backend (2026-05-13)

| Repo | Tags criadas | Develop | Status |
|------|-------------|---------|--------|
| rt-debito-api | v1.0.0–v1.0.3 | Deletada | Completo |
| rt-tenant-provisioning | v1.0.0 | Deletada | Completo |
| rt-input-service-api | v1.0.0–v1.0.2 | Deletada | Completo |
| rt-agents-conciliation | — | — | Já estava no Modelo C |
| rt-credito-api | — | — | Já estava no Modelo C |

### 4 workflows padrão Model C (em todos os repos)
- `validate-build.yml` — validação de PR (npm ci + cache)
- `build-and-publish.yml` — build + publish JFrog (npm ci + cache + lock no zip)
- `create-release.yml` — criação de release/hotfix branch via workflow_dispatch
- `on-release-merged.yml` — tag + GitHub Release automáticos pós-merge

## Plano de migração — frontend (pendente)

A release 1.0.4 do rt-frontend é a **última do gitflow** — serve como ponte para o Modelo C.

| Passo | Ação | Quando | Status |
|-------|------|--------|--------|
| 1 | QA testa release/1.0.4 normalmente | Ciclo normal | PENDENTE |
| 2 | Merge release/1.0.4 → main + tag v1.0.4 | Após QA aprovar | PENDENTE |
| 3 | Atualizar `create-release.yml` (base: develop → main) | Logo após merge | PENDENTE |
| 4 | Atualizar `on-release-merged.yml` (remover merge-back) | Logo após merge | PENDENTE |
| 5 | Atualizar skill `git-workflow.md` | Junto com 3-4 | PENDENTE |
| 6 | Deletar branch develop | Após confirmar main ok | PENDENTE |
| 7 | Documentar protocolo de releases sobrepostas | Antes da 1.0.5 | PENDENTE |
| 8 | Branch protection rules na main | Antes da 1.0.5 | PENDENTE |

**Why:** develop acumulou todo o trabalho da 1.0.4. Cortar a release dela (como sempre) e mergear na main sincroniza tudo. A partir da 1.0.5, releases cortam da main — Modelo C ativo.

**How to apply:** Não mudar NADA no fluxo da 1.0.4. Mudanças nos workflows e skill só após o merge da 1.0.4 na main.

## Relacionados
- [[feedback-package-lock-standard]] — padrão adotado junto com a migração
