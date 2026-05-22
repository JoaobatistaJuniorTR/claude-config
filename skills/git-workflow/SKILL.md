---
name: git-workflow
description: Manages git branches (gitflow), conventional commits in English, and PR creation via gh CLI. Use when starting work that needs a branch, or when delivering work via commit + PR. Reusable by any skill.
---

# Git Workflow

Branch management, conventional commits, and PR automation for the rt-frontend project.

## When to Use

- Starting any development work (branch setup)
- Delivering completed work (commit + push + PR)
- Referenced by `front-engineer` skill at start and end of every pipeline

---

## The Iron Law

```
NEVER COMMIT TO develop OR main DIRECTLY
```

## Moment 1 — Start of Work

### Step 1: Check current branch

Run `git branch --show-current` and evaluate:

| Current branch | Action |
|---|---|
| `develop` or `main` | Ask user: "Create new branch? Base: develop or main?" (default: develop) |
| `feature/*`, `bugfix/*`, `hotfix/*` | Ask user: "Continue on this branch or create new?" |
| Other branch | Ask user: "This branch doesn't follow gitflow naming. Continue here or create a proper branch?" |

### Step 2: Create branch (if needed)

```bash
git checkout <base>
git pull origin <base>
git checkout -b <type>/short-description-in-english
```

**Branch naming rules:**
- Gitflow prefixes: `feature/`, `bugfix/`, `hotfix/`
- Description in English, kebab-case
- Derived from the user's request
- Examples: `bugfix/pagination-portuguese-labels`, `feature/add-export-button`, `hotfix/date-format-crash`

**Type selection and base branch:**
| Request type | Branch prefix | Base branch |
|---|---|---|
| New functionality | `feature/` | `develop` |
| Bug fix, correction | `bugfix/` | latest `release/*` |
| Critical production fix | `hotfix/` | `main` |

**Finding the latest release branch (for bugfix):**
```bash
git fetch origin
git branch -r --sort=-committerdate | grep 'origin/release/' | head -1 | sed 's|origin/||' | xargs
```
Use the result as `<base>`. If no `release/*` branch exists, fall back to `develop`.

---

## Moment 2 — Delivery

### Step 1: Verify build passes

```bash
ng build
```

**Zero errors and zero warnings required.** If build fails, fix before proceeding. NEVER skip this step.

### Step 2: Stage and commit

- Stage only relevant files (`git add <specific-files>`)
- NEVER use `git add .` or `git add -A`
- Conventional commits, always in English:

| Type | When to use | Example |
|---|---|---|
| `fix:` | Bug fix | `fix: translate pagination labels to Portuguese` |
| `feat:` | New feature | `feat: add export button to grid toolbar` |
| `refactor:` | Code restructuring | `refactor: extract date formatting to shared utility` |
| `style:` | Visual/CSS only | `style: adjust card spacing with Saffron tokens` |
| `chore:` | Config/tooling | `chore: register SafPagination in main.ts` |
| `test:` | Tests only | `test: add unit tests for date formatting` |

Commit message format:
```
<type>: <short description in English>

<optional body explaining why, not what>

Co-Authored-By: Claude <noreply@anthropic.com>
```

### Step 3: Push to remote

```bash
git push -u origin <branch-name>
```

### Step 4: Create PR

Determine the PR target based on the branch type:

| Branch type | PR target |
|---|---|
| `feature/*` | `develop` |
| `bugfix/*` | the `release/*` branch it was created from |
| `hotfix/*` | `main` |

```bash
gh pr create --base <target-branch> --title "<concise title>" --body "$(cat <<'EOF'
## Summary
- <bullet 1: what changed and why>
- <bullet 2: key technical decisions>
- <bullet 3: components/files affected>

## Test plan
- [ ] `ng build` passes with zero errors/warnings
- [ ] <specific manual verification steps>
- [ ] <edge cases checked>

Generated with [Claude Code](https://claude.com/claude-code)
EOF
)"
```

**PR rules:**
- Title: concise, English, < 70 chars
- Body: creative but informative — explain the WHY, not just the WHAT
- Target branch depends on branch type:
  - `feature/*` → PR to `develop`
  - `bugfix/*` → PR to the `release/*` branch it was based on
  - `hotfix/*` → PR to `main`
- NEVER auto-merge — user reviews first

---

## Moment 3 — Release & Hotfix (via GitHub Actions)

The skill is a **wrapper** for the `create-release.yml` Action. Do NOT create release/hotfix branches locally.

### Create a release

```bash
# With version from package.json (default)
gh workflow run create-release.yml -f type=release

# With explicit version (semver override)
gh workflow run create-release.yml -f type=release -f version=1.1.0
```

### Create a hotfix

```bash
# Hotfix from main, version from package.json
gh workflow run create-release.yml -f type=hotfix

# Hotfix with explicit version
gh workflow run create-release.yml -f type=hotfix -f version=1.0.5
```

### Pre-dispatch validations

Before running `gh workflow run`, verify:
1. `gh auth status` — CLI is authenticated
2. No existing branch with the same name: `git ls-remote --heads origin release/X.Y.Z`
3. If overriding version, confirm with user: "Criar release X.Y.Z? (package.json está em Y.Y.Y)"

### What happens automatically after merge to main

The `on-release-merged.yml` workflow handles:
- Tag `vX.Y.Z` on the merge commit
- GitHub Release with changelog
- PR: source branch → develop (merge-back)
- PR: bump develop → next patch version

**User only needs to approve the 2 auto-generated PRs.**

### Trigger build on release/hotfix branch

```bash
gh workflow run build-and-publish-portal.yml --ref release/X.Y.Z
```

---

## Rules

- Never commit to `develop` or `main` directly
- Never force push
- Never skip hooks (`--no-verify`)
- Never use `git add .` or `git add -A`
- Always pull base before creating branch
- Always verify `ng build` before committing
- Always create PR — never merge directly
- Commit messages always in English
- Branch descriptions always in English, kebab-case

## Prerequisites

- Git authentication: uses the machine's existing git/gh CLI auth
- `gh` CLI must be authenticated (`gh auth status` to verify)
- If `gh` is not authenticated, ask user to run `! gh auth login`
