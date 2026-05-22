---
name: close-release
description: Use when closing/finishing a release or hotfix cycle. Automates PR to main, monitors CI for tag, and edits GitHub Release with narrative template. Invoke via /close-release.
---

# /close-release

Closes a release or hotfix cycle end-to-end: PR → merge → tag → narrative GitHub Release.

## When to Use

- User types `/close-release`
- User says "fechar a release", "close the release", "finalizar release"
- After QA approves a release/hotfix branch

## Prerequisites

- `gh` CLI authenticated
- Active release/* or hotfix/* branch in the repo
- Repo follows Model C (release-centric) branching

## Flow

```
Detect release branch
        ↓
Gather commits + PRs
        ↓
Create PR release → main (narrative style)
        ↓
User merges PR
        ↓
Monitor CI for tag + release
        ↓
Edit GitHub Release (narrative style)
        ↓
Done
```

## Steps

### 1. Detect the release branch

```bash
git fetch --all --prune
git branch -r --list 'origin/release/*' 'origin/hotfix/*'
```

- If exactly one: use it
- If multiple: ask user which one
- If none: stop — "Nenhuma release/hotfix branch ativa."

### 2. Gather release data

```bash
# Version from branch name
BRANCH="release/X.Y.Z"  # or hotfix/X.Y.Z
VERSION=$(echo $BRANCH | sed 's|.*/||')

# Commits ahead of main
git log --oneline origin/main..origin/$BRANCH --no-merges

# PRs merged into the release
gh pr list --base $BRANCH --state merged --json number,title,author

# Diff stats
git diff --stat origin/main..origin/$BRANCH
```

### 3. Create PR to main

Create PR with full narrative description following the global CLAUDE.md PR template:

- **Title:** `release: vX.Y.Z — [Título Dramático]`
- **Body:** must include all sections from CLAUDE.md PR style:
  - `## 🔧 O que rolou aqui` — narrative paragraph
  - `## 🪦 R.I.P.` — only if something was removed
  - `## 🚀 O que entrou no lugar` — before/after table or feature list
  - `## 📋 Commits` — or PRs list
  - `## 🧪 Como testar` — numbered steps
  - `## 💀 Risco` — honest risk assessment
  - Closing quote + `🤖 Generated with [Claude Code](https://claude.com/claude-code)`

```bash
gh pr create --base main --head $BRANCH --title "release: v$VERSION — [Título]" --body "$BODY"
```

### 4. Wait for merge

Tell the user:
- PR number and link
- "Aprove e mergeie o PR. Me avise quando estiver feito."

When user confirms merge, proceed. Or detect:
```bash
gh pr view $PR_NUMBER --json state --jq '.state'
# "MERGED" = proceed
```

### 5. Monitor CI for tag

After merge, the `on-release-merged.yml` workflow creates a tag and a basic GitHub Release.

```bash
# Wait for the workflow to start
sleep 5

# Find the run
RUN_ID=$(gh run list --workflow=on-release-merged.yml --limit 1 --json databaseId,status --jq '.[0].databaseId')

# Watch it complete
gh run watch $RUN_ID
```

Verify tag exists:
```bash
git fetch --tags
git tag -l "v$VERSION"
```

### 6. Edit GitHub Release with narrative template

The CI creates a basic release. Now overwrite it with the full narrative template from CLAUDE.md Release style.

Gather data for the release notes:
```bash
# PRs included
gh pr list --base main --state merged --search "head:release/$VERSION OR head:hotfix/$VERSION" --json number,title

# All PRs that targeted the release branch
gh pr list --base $BRANCH --state merged --json number,title,author
```

Build the release body following CLAUDE.md Release template:
- `## 🚀 [Título curto e dramático]` — narrative paragraph
- `## ✨ Destaques` — grouped by feature, not by commit
- `## 🐛 Correções` — bug fixes with ADO references
- `## 🗑️ Removido` — only if applicable
- `## 🎯 Áreas impactadas` — QA-friendly module list
- `## 💀 Risco` — honest assessment
- `## 📋 PRs incluídos` — list with numbers and titles
- Closing quote + `🤖 Generated with [Claude Code](https://claude.com/claude-code)`

```bash
gh release edit "v$VERSION" --title "v$VERSION — [Título Dramático]" --notes "$RELEASE_BODY"
```

### 7. Cleanup (if needed)

If the old workflow created a merge-back PR to develop, close it:
```bash
MERGEBACK=$(gh pr list --head $BRANCH --base develop --state open --json number --jq '.[0].number')
if [ -n "$MERGEBACK" ]; then
  gh pr close $MERGEBACK --comment "Fechando — Modelo C não usa develop."
fi
```

## Adapting to the Repository

The skill works in any repo with:
- `on-release-merged.yml` workflow (creates tag + basic release)
- Model C branching (releases from main, no develop)

For repos still on gitflow: the merge-back cleanup in step 7 handles the transition.

## Shell Compatibility — CRITICAL

The `gh` CLI `--notes` and `--body` flags need multiline strings. **NEVER use `@-` (stdin redirect) or Bash heredoc (`<<'EOF'`)** — they silently fail or produce literal `@-` as content depending on the shell context.

**Correct approach — always use inline single-quoted strings:**

```bash
gh release edit "v$VERSION" --title "v$VERSION — Título" --notes '## 🚀 Título

Conteúdo multiline aqui...

- Item 1
- Item 2'
```

```bash
gh pr create --base main --head $BRANCH --title "release: v$VERSION" --body '## 🔧 O que rolou

Conteúdo multiline aqui...'
```

Single-quoted strings in Bash support newlines natively. This works in both the Bash tool and PowerShell contexts.

**If the content contains single quotes**, use a temp file approach:
```bash
echo "$BODY" > /tmp/release-notes.md
gh release edit "v$VERSION" --notes-file /tmp/release-notes.md
rm /tmp/release-notes.md
```

## Rules

- ALWAYS gather full context before creating the PR
- ALWAYS use narrative style from CLAUDE.md templates (both PR and Release)
- NEVER auto-merge — user must approve and merge the PR
- ALWAYS monitor the CI run to confirm tag creation
- ALWAYS edit the release — never leave the CI-generated basic release as final
- Group changes by FEATURE in releases, not by individual commit
- NEVER use `@-` or heredoc (`<<'EOF'`) to pass content to `gh` — use inline `--notes '...'` or `--notes-file`
