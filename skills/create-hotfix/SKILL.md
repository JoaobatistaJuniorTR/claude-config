---
name: create-hotfix
description: Creates a hotfix branch from main via GitHub Actions create-release.yml workflow. Dispatches the Action, monitors the run, and reports the result. Use for urgent production fixes.
---

# /create-hotfix

Creates a hotfix branch from main via the `create-release.yml` GitHub Action.

## When to Use

- User types `/create-hotfix` or `/create-hotfix X.Y.Z`
- Urgent production fix needed
- Referenced by `git-workflow` skill

## Prerequisites

- `gh` CLI authenticated (`gh auth status`)
- Repository must have `.github/workflows/create-release.yml`

## Steps

### 1. Verify workflow exists

```bash
gh workflow list --json name,path --jq '.[] | select(.path | contains("create-release"))'
```

If not found:
- Stop and tell the user: "Este repositório não tem o workflow `create-release.yml`. Crie manualmente ou copie do rt-frontend."

### 2. Determine version

- If the user provided a version argument (e.g., `/create-hotfix 1.0.5`), use it
- If no argument, ask: "Usar versão do package.json (main) ou informar uma específica?"
  - If package.json: leave version empty in the workflow dispatch
  - If specific: validate semver format (X.Y.Z)

### 3. Pre-flight checks

```bash
# Check current version in package.json on main
git fetch origin main
git show origin/main:package.json | node -p "JSON.parse(require('fs').readFileSync('/dev/stdin','utf8')).version"

# Check if hotfix branch already exists
git ls-remote --heads origin | grep "refs/heads/hotfix/" || echo "Nenhuma hotfix branch ativa"
```

Show the user:
- Current version in main's package.json
- Any existing hotfix branches
- The version that will be used

### 4. Confirm and dispatch

Ask user to confirm, then:

```bash
# Without version (reads from package.json)
gh workflow run create-release.yml -f type=hotfix

# With explicit version
gh workflow run create-release.yml -f type=hotfix -f version=X.Y.Z
```

### 5. Monitor the run

```bash
# Wait for the run to appear
sleep 3

# Get the latest run ID
RUN_ID=$(gh run list --workflow=create-release.yml --limit 1 --json databaseId --jq '.[0].databaseId')

# Watch it
gh run watch $RUN_ID
```

### 6. Report result

On success, show:
- Branch name created
- Version set
- Next step: "Corrija o bug na branch hotfix, depois dispare o build"
- Ask: "Quer que eu crie uma branch de trabalho a partir da hotfix?"

On failure, show the error from the workflow logs:
```bash
gh run view $RUN_ID --log-failed
```

## Rules

- NEVER create the hotfix branch locally — always dispatch the GitHub Action
- ALWAYS verify the workflow exists before dispatching
- ALWAYS show the user what will happen before dispatching
- ALWAYS monitor the run to completion
- Hotfix is URGENT — minimize questions, maximize speed
