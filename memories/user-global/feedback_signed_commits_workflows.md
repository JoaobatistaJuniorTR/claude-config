---
name: feedback-signed-commits-workflows
description: Repos da empresa exigem verified signatures — workflows que fazem git commit via CLI sempre falham no push. Bump e mutações de arquivo viram passo manual via skill local OU GitHub Contents API.
metadata:
  type: feedback
---

Repos da empresa (tr/*) têm branch protection com **"Commits must have verified signatures"**. Qualquer workflow que use `git commit` via CLI — mesmo configurado como `github-actions[bot]` — produz commit não assinado e é rejeitado no `git push` com `GH013: signed commits required`.

**Why:** Esse problema foi descoberto duas vezes pelo mesmo motivo, em workflows diferentes:
1. `on-release-merged.yml` (rt-frontend) — fix em commit `34a9f51` (2026-05-06): removeu o bump da develop e tornou manual.
2. `create-release.yml` (rt-frontend) — fix em PR #115 (2026-05-21): removeu bump+commit do action; skill `/create-release` faz bump localmente com a chave GPG do dev.

Em ambos os casos a memória existente cobria só o caso já visto, e o problema reapareceu em workflow diferente com a mesma anti-pattern. Hora de generalizar.

**How to apply:**

Ao criar ou revisar **qualquer** workflow do GitHub Actions que precise mutar arquivos do repo, NÃO use:
```yaml
- run: |
    git config user.name "github-actions[bot]"
    git config user.email "..."
    git add <files>
    git commit -m "..."
    git push
```

Use uma das duas alternativas:

**Opção A (preferida) — mover pra skill local:**
- Action só faz infra (criar branch vazia, abrir PR, criar tag, etc.) — coisas que não exigem commit novo.
- Mutação de arquivos é feita por skill no Claude Code rodando localmente, com a chave GPG do dev.
- Padrão usado no rt-frontend: `/create-release` orquestra dispatch + bump local + push.

**Opção B — GitHub Contents API:**
- `gh api PUT /repos/{owner}/{repo}/contents/{path}` cria commit assinado automaticamente quando autenticado com `GITHUB_TOKEN`.
- Mais código (precisa lidar com SHA do arquivo, base64, package-lock.json em arquivos separados), mas mantém tudo automatizado.

Vale pra qualquer arquivo, não só `package.json`. Aplicar regra preventivamente em todos os workflows: build, release, tag, changelog auto-gerado, etc.

Relacionado: [[branching-strategy-model-c]] (no projeto taxonert-frontend), [[feedback-package-lock-standard]] (mesma família, precisa ser atualizada — a linha "Incluir package-lock.json no git add do create-release (bump)" virou obsoleta)
