---
name: front-engineer
description: Unified entry point for all frontend development tasks. Triages the request (design vs investigation vs migration), sets up git workflow, and orchestrates the correct pipeline. Use when the user asks to create, adjust, fix, translate, or modify any screen or component.
---

# Front Engineer

Unified workflow for all frontend development in taxonert_frontend. Every screen task — creation, adjustment, fix, migration — enters through here.

## When to Use

**Always.** Any request that touches screens or components:
- Creating new screens/components
- Adjusting existing screens (layout, spacing, labels, behavior)
- Fixing bugs in screens
- Translating labels or formatting
- Migrating screens to Saffron
- Any visual or behavioral change in the UI

---

## Phase 0: Triage

Classify the user's request into one of three scenarios:

| Signal in the request | Scenario | Pipeline |
|---|---|---|
| Figma URL (`figma.com/design/...`), "criar tela", "redesenhar", "nova tela", "implementar design" | **design** | Git Workflow → Saffron Orchestrator → Git Workflow |
| "corrigir", "ajustar", "traduzir", "bug", "não funciona", "alterar", "mover", "trocar", "adicionar X ao Y existente" | **investigation** | Git Workflow → Investigation → Git Workflow |
| "migrar para Saffron", "converter para Saffron" | **migration** | Git Workflow → Saffron Orchestrator (migration mode) → Git Workflow |
| Ambiguous or unclear | **ask** | Ask the user which scenario applies |

**Detection rules:**
- If the request mentions a Figma URL → always `design`
- If the request mentions an EXISTING component/screen + a change → `investigation`
- If the request describes a NEW screen from scratch (no Figma) → `design` (prompt-to-design mode)
- When in doubt → ask

---

## Investigation Pipeline

For fixes, adjustments, translations, and minor changes to existing code.

```
┌─────────────────────────────────────────────┐
│ 1. GIT WORKFLOW — Moment 1                  │
│    Follow @git-workflow skill               │
│    → Check branch, ask user, create if needed│
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│ 2. INVESTIGATION — Phases 1-4               │
│    Follow @investigation skill              │
│    → Understand → Investigate → Consult     │
│    → Diagnose → PRESENT TO USER             │
└──────────────────┬──────────────────────────┘
                   │
            ┌──────▼──────┐
            │ User approves│
            │ diagnosis?   │
            └──┬───────┬──┘
              YES      NO → revise diagnosis
               │
┌──────────────▼──────────────────────────────┐
│ 3. INVESTIGATION — Phase 5                  │
│    → Implement fix                          │
│    → Tests (if relevant)                    │
│    → ng build (mandatory)                   │
│    → ng test (if tests exist)               │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│ 4. GIT WORKFLOW — Moment 2                  │
│    Follow @git-workflow skill               │
│    → Commit (conventional, English)         │
│    → Push                                   │
│    → PR to develop                          │
└─────────────────────────────────────────────┘
```

---

## Design Pipeline

For new screens, Figma implementations, and prompt-to-design.

```
┌─────────────────────────────────────────────┐
│ 1. GIT WORKFLOW — Moment 1                  │
│    Follow @git-workflow skill               │
│    → Check branch, ask user, create if needed│
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│ 2. SAFFRON ORCHESTRATOR                     │
│    Follow .claude/agents/saffron-orchestrator│
│    The orchestrator runs its own internal    │
│    phases:                                  │
│    → Designer → Specialist → Developer      │
│    → Auditor (Saffron A1-A10 + Angular B1-B6)│
│                                             │
│    front-engineer does NOT interfere with    │
│    the orchestrator's internal pipeline.     │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│ 3. GIT WORKFLOW — Moment 2                  │
│    Follow @git-workflow skill               │
│    → Commit (conventional, English)         │
│    → Push                                   │
│    → PR to develop                          │
└─────────────────────────────────────────────┘
```

---

## Migration Pipeline

For migrating existing screens to Saffron.

```
┌─────────────────────────────────────────────┐
│ 1. GIT WORKFLOW — Moment 1                  │
│    → Branch setup                           │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│ 2. SAFFRON ORCHESTRATOR (migration mode)    │
│    Follow .claude/agents/saffron-orchestrator│
│    Scenario: migration                      │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│ 3. GIT WORKFLOW — Moment 2                  │
│    → Commit + Push + PR                     │
└─────────────────────────────────────────────┘
```

---

## Rules

- **Never skip investigation** — even if the fix seems obvious
- **Never commit without build passing** — `ng build` must show zero errors/warnings
- **Never merge directly** — always create PR to develop (or main if user specifies)
- **Always consult Saffron** when `saf-*` components are involved
- **Conventional commits in English** — always, no exceptions
- **PR with creative description** — explain the WHY, not just the WHAT
- **Code names in English** — variables, methods, classes, interfaces
- **Follow project patterns** — signals, inject(), OnPush, CUSTOM_ELEMENTS_SCHEMA
- **Tokens only** — `var(--saf-*)`, never hardcoded hex/px values

## Error Handling

| Situation | Action |
|---|---|
| User request is ambiguous | Ask which scenario applies (design/investigation/migration) |
| Build fails after fix | Fix build errors before committing. Do NOT skip. |
| `gh` CLI not authenticated | Ask user to run `! gh auth login` |
| Investigation reveals deeper issue | Present findings, suggest scope change to user |
| Fix requires changes beyond original scope | Present expanded scope to user, get approval before proceeding |
