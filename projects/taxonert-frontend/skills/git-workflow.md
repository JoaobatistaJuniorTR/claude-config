---
name: git-workflow
description: Manages git branches (gitflow), conventional commits in English, and PR creation via gh CLI. Use when starting work that needs a branch, or when delivering work via commit + PR. Reusable by any skill.
---

# Git Workflow

Branch management, conventional commits, and PR automation for the taxonert_frontend project.

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

**Type selection:**
| Request type | Branch prefix |
|---|---|
| New functionality | `feature/` |
| Bug fix, correction | `bugfix/` |
| Critical production fix | `hotfix/` |

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

### Step 4: Create PR to develop

```bash
gh pr create --base develop --title "<concise title>" --body "$(cat <<'EOF'
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
- Always target `develop` (unless user specifies `main`)
- NEVER auto-merge — user reviews first

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
