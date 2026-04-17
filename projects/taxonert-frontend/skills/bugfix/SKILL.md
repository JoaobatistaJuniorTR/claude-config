---
name: bugfix
description: Reads a Bug work item from Azure DevOps, triages it, and orchestrates the fix through front-engineer (investigation pipeline). Use when the user runs /bugfix <id> or /bugfix <url>. Also supports /bugfix close <id> for post-review closure.
---

# Bugfix — ADO-Integrated Fix Workflow

Reads a Bug from Azure DevOps, triages it, and drives it through the existing investigation pipeline to a PR.

## Invocation

```
/bugfix 12345
/bugfix https://dev.azure.com/tr-ggo/Mastersaf%20Fiscal%20Solutions/_workitems/edit/12345
/bugfix close 12345
```

---

## Input Parsing

Parse the arguments passed to the skill:

1. If first argument is `close` → **Close Mode** (Phase 7 only). Second argument is the work item ID.
2. If argument is a number → **Fix Mode** (Phases 1-5). The number is the work item ID.
3. If argument is a URL containing `/_workitems/edit/` → **Fix Mode**. Extract the ID with: `echo "$URL" | grep -oP '/_workitems/edit/\K\d+'`
4. Otherwise → inform the user: "Usage: `/bugfix <id>` or `/bugfix <url>` or `/bugfix close <id>`" and STOP.

---

## Fix Mode (Phases 1–5)

### Phase 1: Capture

Fetch the Bug from ADO and present a structured summary.

**Step 1 — Validate environment:**

```bash
if [ -z "$ADO_PAT" ] || [ -z "$ADO_ORG" ] || [ -z "$ADO_PROJECT" ]; then
  echo "ERROR: ADO environment variables not configured."
  echo "Set ADO_PAT, ADO_ORG, ADO_PROJECT in ~/.claude/settings.json"
fi
```

If validation fails, STOP and inform the user.

**Step 2 — Fetch work item:**

Follow `ado-client.md` Operation 1 to fetch the work item by ID.

**Step 3 — Extract fields:**

From the JSON response, extract:
- `System.Title` → bug title
- `Microsoft.VSTS.TCM.ReproSteps` → repro steps (HTML → strip tags for readable text)
- `Microsoft.VSTS.TCM.SystemInfo` → system info
- `System.Description` → description (fallback if ReproSteps is empty)
- `Microsoft.VSTS.Common.Priority` → priority (1=Critical, 4=Low)
- `Microsoft.VSTS.Common.Severity` → severity (1=Critical, 4=Low)
- `System.State` → current state

**Step 4 — Fetch attachments:**

Follow `ado-client.md` Operations 2 and 3:
- List attachments from the `relations` array
- Download image attachments (PNG, JPG, GIF) to `/tmp/`
- Use the `Read` tool to view downloaded images (Claude is multimodal)

**Step 5 — Present structured summary:**

```
## Bug #{ID}: {title}
**Estado**: {state} | **Prioridade**: {priority} | **Severidade**: {severity}

### Repro Steps
{clean text from HTML}

### System Info
{system info if available, or "Não informado"}

### Screenshots
{description of each image after viewing them}
```

---

### Phase 2: Comprehension (Triage)

Evaluate whether the bug has enough information to proceed.

**Minimum criteria to proceed (bug is "understood"):**
- **What happens** — clear description of the current (wrong) behavior
- **Where it happens** — screen, module, or path in the system (enough to search the codebase)

Screenshot is NOT required. If present, it helps. If absent, clear text description is sufficient.

**If understood → go to Phase 3.**

**If NOT understood → Phase 2a: Ask the dev in the conversation.**

Present what is missing and ask directly:

> "Li o Bug #{ID} mas preciso de mais informações:
> - {specific point that's missing}
> - {another point}
> Você sabe responder isso?"

Wait for the dev's response.

- If the dev answers → re-evaluate triage with the new info.
- If the dev says "não sei" or cannot answer → **Phase 2b: Escalate to ADO.**

**Phase 2b — Post comment on ADO:**

Follow `ado-client.md` Operation 4 to post a comment in PT-BR:

```html
<b>Preciso de mais informações para corrigir este bug:</b><br>
<ul>
<li>{specific point 1}</li>
<li>{specific point 2}</li>
</ul>
<br><em>Comentário gerado via Claude Code</em>
```

Inform the user:
> "Comentei no Bug #{ID} no ADO pedindo mais informações. Quando tiver resposta, rode `/bugfix {ID}` novamente."

**STOP execution.**

---

### Phase 3: Create Task in ADO

After positive triage, register the work in ADO.

**Step 1 — Create child Task:**

Follow `ado-client.md` Operation 6 to create a Task:
- Title: `Fix: {short description in English}`
- Original Estimate: estimate hours based on bug complexity (simple=1h, medium=2h, complex=4h)
- Parent link: the Bug work item ID

**Step 2 — Inform the dev:**

> "Task #{NEW_TASK_ID} criada no ADO, linkada ao Bug #{BUG_ID}. Estimativa: {hours}h."

---

### Phase 4: Implement

Hand off to the existing front-engineer pipeline.

**Step 1 — Git setup:**

Follow `git-workflow.md` Moment 1:
- `git checkout develop`
- `git pull origin develop`
- Create branch: `bugfix/{descriptive-name-in-english}`

**Step 2 — Delegate to front-engineer (investigation pipeline):**

The bug context from Phase 1 (title, repro steps, screenshots, analysis) becomes the input for the investigation pipeline.

Follow `front-engineer` → Investigation Pipeline:
- The request type is a bug fix → `investigation` pipeline
- **Phase 1 (Understand)**: already done in our Phase 2 — pass the context
- **Phase 2 (Investigate)**: follow investigation.md — read affected files, search patterns
- **Phase 3 (Consult)**: follow the mandatory cascade — skills → MCP → memories → angular-developer → WebSearch
- **Phase 4 (Diagnose)**: formulate hypothesis, present to user, wait for confirmation
- **Phase 5 (Implement)**: implement the fix, `ng build` (zero errors/warnings), `ng test --watch=false`

**CRITICAL:** Follow the investigation skill's Iron Law: DO NOT IMPLEMENT WITHOUT CONFIRMED DIAGNOSIS.

---

### Phase 5: PR

After implementation is complete and build passes:

**Step 1 — Commit and push:**

Follow `git-workflow.md` Moment 2:
- Stage specific files (never `git add .`)
- Commit: `fix: {description in English}`
- `git push -u origin bugfix/{branch-name}`

**Step 2 — Create PR:**

```bash
gh pr create --base develop --title "fix: {concise title}" --body "$(cat <<'EOF'
{PR body following the CLAUDE.md global PR style — PT-BR, narrative, ironic}
EOF
)"
```

The PR description MUST follow the global CLAUDE.md style (PT-BR, narrative tone, comparison table, ironic quote).

**Step 3 — Comment on ADO:**

Follow `ado-client.md` Operation 4 to comment on BOTH the Bug and the Task:

```html
PR aberto para revisão: <a href="{PR_URL}">{PR_URL}</a><br>
Branch: <code>bugfix/{branch-name}</code><br>
<br><em>Comentário gerado via Claude Code</em>
```

**Step 4 — Inform the user:**

> "PR criado: {PR_URL}
> Comentei no Bug #{ID} e na Task #{TASK_ID} no ADO com o link do PR.
> Após sua revisão e aprovação, rode `/bugfix close {ID}` para fechar no ADO."

**STOP — Phase 6 (review) is asynchronous.**

---

## Close Mode (Phase 7)

Invoked via `/bugfix close {ID}`. Only runs after the dev has reviewed and approved the PR.

**Step 1 — Ask the dev for hours:**

> "Quantas horas gastar nesse fix? (minha estimativa foi {X}h)"

Wait for the dev's response.

**Step 2 — Update Task in ADO:**

Follow `ado-client.md` Operation 5:
- Set `Completed Work` = hours confirmed by dev
- Set `Remaining Work` = 0
- Set `System.State` = "Closed"

**Step 3 — Update Bug in ADO:**

Follow `ado-client.md` Operation 5:
- Set `System.State` = "Resolved"

**Step 4 — Post closing comment on Bug:**

Follow `ado-client.md` Operation 4:

```html
Bug corrigido e PR aprovado.<br>
<b>Horas:</b> {hours}h<br>
<b>PR:</b> <a href="{PR_URL}">{PR_URL}</a><br>
<br><em>Comentário gerado via Claude Code</em>
```

**Step 5 — Inform the user:**

> "Bug #{ID} resolvido no ADO. Task #{TASK_ID} fechada. {hours}h lançadas."

---

## Rules

1. **Never close bug/task automatically** — only via explicit `/bugfix close` from the dev
2. **Never skip investigation** — even if the bug seems obvious, follow all 5 phases
3. **Always branch from develop** — no direct production fixes (for now)
4. **Conventional commits in English** — `fix:`, `feat:`, etc.
5. **PR to develop** — never merge directly
6. **Comments on ADO in PT-BR** — team language
7. **Never fabricate data** — if something is not in the bug, ask
8. **Hours are estimates** — Claude estimates, dev confirms/adjusts
9. **Follow front-engineer rules** — all existing rules from front-engineer and investigation skills apply

## Error Handling

| Situation | Action |
|---|---|
| ADO API returns 401 | PAT expired. Tell user: "Atualize o ADO_PAT em ~/.claude/settings.json" |
| ADO API returns 404 | Wrong ID or project. Tell user: "Work item não encontrado. Verifique o ID." |
| Work item is not a Bug | Inform: "Work item #{ID} é tipo {type}, não Bug. O /bugfix é para Bugs." STOP. |
| `gh` CLI not authenticated | Tell user to run `! gh auth login` |
| Build fails | Fix build errors before committing (standard investigation rule) |
| Dev doesn't confirm diagnosis | Revise diagnosis — do NOT proceed to implementation |

## Related Skills

- `ado-client.md` (global) — ADO API patterns referenced throughout
- `front-engineer` — Orchestrates the investigation pipeline (Phase 4)
- `investigation.md` — 5-phase investigation workflow (used inside Phase 4)
- `git-workflow.md` — Branch management, commits, PRs (Phases 4-5)
