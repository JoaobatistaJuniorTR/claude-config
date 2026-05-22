---
name: create-release
description: Creates a release branch from main via GitHub Actions create-release.yml workflow. Dispatches the Action, monitors the run, and reports the result. Use when starting a new release cycle.
---

# /create-release

Creates a release branch from main end-to-end: dispatches the GitHub Action to create the empty branch, then locally bumps `package.json` with a signed commit and pushes.

## When to Use

- User types `/create-release` or `/create-release X.Y.Z`
- Starting a new release cycle
- Referenced by `git-workflow` skill

## Prerequisites

- `gh` CLI authenticated (`gh auth status`)
- Repository must have `.github/workflows/create-release.yml`
- Local git configured to sign commits (the repo enforces verified signatures)
- Working tree clean (skill will not run if there are uncommitted changes)

## Why the skill bumps locally + opens a PR

Two org-level rules constrain this flow on `tr/*` repos:

1. **Verified signatures required** — `github-actions[bot]` cannot sign via git CLI, so the bump cannot happen in the Action.
2. **Secret Detection status check required** — the TR ProdSec app (`a208370-prodsec-secret-detection`) only triggers on `pull_request` events, so direct push to a protected branch is rejected: the head commit has no Secret Detection check yet.

Combined: the bump must (a) be signed by the dev locally AND (b) reach `release/X.Y.Z` via a PR. The skill orchestrates both — branch off `release/X.Y.Z`, bump, push the chore branch, open PR back to `release/X.Y.Z`.

This split mirrors the precedent set in commit `34a9f51` for `on-release-merged.yml`, where the bump was made manual for the signed-commits reason.

## Flow

```
Verify prerequisites
        ↓
Determine version (arg or ask)
        ↓
Pre-flight checks
        ↓
Dispatch Action (creates empty release/X.Y.Z from main)
        ↓
Monitor Action to completion
        ↓
Fetch + checkout release/X.Y.Z locally
        ↓
Create chore/bump-X.Y.Z off release/X.Y.Z
        ↓
npm version + signed commit + push chore branch
        ↓
Open PR chore/bump-X.Y.Z → release/X.Y.Z (narrative template)
        ↓
Tell user to review/merge
```

## Steps

### 1. Verify workflow exists

```bash
gh workflow list --json name,path --jq '.[] | select(.path | contains("create-release"))'
```

If not found: stop and tell the user "Este repositório não tem o workflow `create-release.yml`. Crie manualmente ou copie do rt-frontend."

### 2. Verify working tree is clean

```bash
git status --porcelain
```

If any output: stop and tell the user "Working tree não está limpo. Commit ou stash antes de criar a release."

### 3. Determine version

- If the user provided a version argument (e.g., `/create-release 1.1.0`), use it
- If no argument, ask: "Qual versão usar para a nova release? (sugerir próximo patch/minor com base no package.json atual)"
- Validate semver format (X.Y.Z)

### 4. Pre-flight checks

```bash
git fetch origin main
git show origin/main:package.json | node -p "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).version"
git ls-remote --heads origin "refs/heads/release/*" "refs/heads/hotfix/*"
```

Show the user:
- Current version in main's package.json
- Existing release/hotfix branches
- The target version

Confirm before dispatching.

### 5. Dispatch the Action

```bash
gh workflow run create-release.yml -f type=release -f version=X.Y.Z
```

For hotfix: `-f type=hotfix`.

### 6. Monitor the run

```bash
sleep 4
RUN_ID=$(gh run list --workflow=create-release.yml --limit 1 --json databaseId --jq '.[0].databaseId')
gh run watch $RUN_ID --exit-status
```

If it fails, show `gh run view $RUN_ID --log-failed` and stop. Do NOT proceed to local bump.

### 7. Fetch + checkout release branch

```bash
RELEASE_BRANCH="release/X.Y.Z"  # or hotfix/X.Y.Z
git fetch origin "$RELEASE_BRANCH"
git checkout "$RELEASE_BRANCH"
git pull origin "$RELEASE_BRANCH"
```

### 8. Create chore branch for the bump

```bash
CHORE_BRANCH="chore/bump-X.Y.Z"
git checkout -b "$CHORE_BRANCH"
```

Why a separate branch: direct push to `release/X.Y.Z` is blocked by the org's Secret Detection rule (only runs on PR events). The chore branch lets us push, trigger the check, and merge via PR.

### 9. Bump package.json and commit

```bash
npm version X.Y.Z --no-git-tag-version
```

Updates both `package.json` and `package-lock.json`.

Verify the diff:
```bash
git diff --stat
git diff package.json | head -20
```

Commit with the user's signed git config:
```bash
git add package.json package-lock.json
git commit -m "chore: bump version to X.Y.Z"
```

**Do NOT pass `--no-gpg-sign` or override signing config.** The dev's git must already be configured to sign — otherwise the push fails with `GH013: Commits must have verified signatures`. If that happens, stop and tell the user to fix git signing config.

### 10. Push chore branch

```bash
git push -u origin "$CHORE_BRANCH"
```

This push succeeds because `chore/*` branches don't have the Secret Detection enforcement (only protected branches like `main`, `release/*`, `hotfix/*` do). The Secret Detection check runs once the PR is opened.

### 11. Open PR back to release branch

Use narrative template from CLAUDE.md (PR style). For a bump-only PR, keep it concise but in the same tone:

```bash
gh pr create \
  --base "$RELEASE_BRANCH" \
  --head "$CHORE_BRANCH" \
  --title "chore: bump version to X.Y.Z" \
  --body '## 🔧 O que rolou aqui

Abertura oficial do ciclo da release vX.Y.Z. Este PR só faz o bump do `package.json` e `package-lock.json` da versão Y.Y.Y (última shipped em main) para X.Y.Z (alvo desta release).

Em Model C, o bump não pode rolar no action (commits do bot não passam na regra de signed commits) nem direto na branch (Secret Detection da ProdSec só roda em eventos de PR). Daí esse PR-ponte de uma linha cada arquivo.

## 🚀 O que entrou

- `package.json`: Y.Y.Y → X.Y.Z
- `package-lock.json`: idem

## 📋 Commits

1. `chore: bump version to X.Y.Z`

## 🧪 Como testar

- Verificar que o check **Secret Detection** rodou e passou
- Conferir o diff: só duas linhas (versão em package.json e package-lock.json)
- Mergear quando aprovado — daí a release está oficialmente aberta para os devs trabalharem

## 💀 Risco

**Nenhum.** Bump puro, sem código de aplicação.

---
> *"Beginnings are usually scary, and endings are usually sad, but it'\''s everything in between that makes it all worth living."* — Bob Marley

🤖 Generated with [Claude Code](https://claude.com/claude-code)'
```

### 12. Report result

Show:
- Release branch: `release/X.Y.Z`
- Bump PR URL (number + link)
- Next step: "Aprove e mergeie o PR — a release fica oficialmente aberta. Devs criam branches `feat/*` ou `fix/* → release/X.Y.Z`."
- Once user confirms PR is merged: "Use `/close-release` quando o ciclo terminar."
- Optional: "Quer que eu monitore o merge do PR e te avise?"

## Rules

- NEVER create the release branch locally — always dispatch the GitHub Action first
- NEVER bypass commit signing (`--no-gpg-sign`, `-c commit.gpgsign=false`) — the repo rule exists for a reason
- NEVER try to push the bump commit directly to `release/X.Y.Z` — Secret Detection rule blocks it
- ALWAYS verify the workflow exists before dispatching
- ALWAYS verify working tree is clean before dispatching
- ALWAYS monitor the Action run to completion before local bump
- ALWAYS bump via PR `chore/bump-X.Y.Z → release/X.Y.Z` — devs later open their own `feat/*` or `fix/* → release/X.Y.Z` PRs
- DO NOT auto-merge the bump PR — user reviews and merges
- If the push of the chore branch fails for signing reasons, stop and explain — do NOT retry blindly
