---
name: wip-check-draft-override-bug
description: Bug do WIP App no workflow taxone-fix — check fica travado eternamente quando PR dev e PR rc draft compartilham o mesmo HEAD; workaround obrigatória é gh pr ready --undo + gh pr ready
metadata: 
  node_type: memory
  type: project
  originSessionId: 0c52b257-f9dc-42d5-9707-3ccd8d1e1528
---

Sempre que [[taxone-fix-workflow]] cria os dois PRs do mesmo branch (`fix/MFS.../...` → dev não-draft + rc draft), o **WIP App** (github-actions/wip) trava o check do PR dev em `status: in_progress`, `title: "draft mode override"`, `summary: "The pull request is in draft mode and will not be merged."`. Isso bloqueia o merge do PR dev até intervenção manual.

**Why:** O WIP App cria UM check run por commit, e esse commit pertence a 2 PRs simultaneamente quando o branch é base de PR dev (não-draft) + PR rc (draft). O WIP App entra em modo "draft override" porque DETECTA o PR draft no commit — e como GitHub mostra o mesmo check run em todos os PRs do commit, o PR não-draft também fica com o WIP travado.

Confirmado empiricamente em MFS1127259 (PR #11 → dev + PR #12 → rc draft, ambos no commit `838205c`). Check run id `77245530960` lista `pull_requests: [11, 12]` com `output.title: "draft mode override"`.

**How to apply:** Imediatamente após criar o PR rc draft (passo 9.3 do skill), executar:

```bash
gh pr ready {numeroPR_dev} --undo
gh pr ready {numeroPR_dev}
```

Isso força o GitHub a disparar um novo evento `ready_for_review` exclusivo do PR dev → o WIP App cria um NOVO check run associado só a esse PR e analisa corretamente (resultado: `success / "Ready for review"`).

A workaround está documentada no skill `~/.claude/skills/taxone-fix/skill.md` (seção 9.4) como passo obrigatório.
