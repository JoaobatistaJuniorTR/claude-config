---
name: investigation
description: Investigation-first workflow for frontend fixes and adjustments. Enforces reading code, finding patterns, consulting Saffron/Angular tools, and diagnosing before implementing. Use when fixing bugs, adjusting UI, translating labels, or making any non-design change.
---

# Investigation

Systematic investigation workflow for frontend fixes and adjustments. Diagnose before you fix.

## When to Use

- Bug fixes ("not working", "broken", "error")
- UI adjustments ("adjust", "change", "move", "resize")
- Translations/i18n ("translate", "Portuguese", "locale")
- Refactoring ("refactor", "clean up", "simplify")
- Minor features ("add button", "add column", "add filter")
- Any change that does NOT require a Figma design

---

## The Iron Law

```
DO NOT IMPLEMENT WITHOUT CONFIRMED DIAGNOSIS
```

If you haven't completed Phase 4 (Diagnose), you cannot write implementation code. Period.

---

## Phase 1: Understand the Request

Before touching any file:

1. **Classify the type**:
   - Bug fix — something is broken or behaving incorrectly
   - Visual adjustment — spacing, alignment, colors, layout
   - i18n/translation — labels, messages, locale-specific formatting
   - Refactor — restructuring without behavior change
   - Minor feature — small addition to existing functionality

2. **Identify affected components/screens** — what parts of the app does this touch?

3. **State the expected outcome** — what should the user see after the fix?

---

## Phase 2: Investigate the Code

### Step 1: Locate ALL affected files

Do NOT stop at the obvious component. Find:
- `.component.html`, `.component.ts`, `.component.scss` of the component
- **Services** the component injects
- **Models/interfaces** that define the data types
- **Routes** if navigation is affected
- **Shared components** if any reusable component is involved
- `main.ts` if Saffron component registration is needed
- `styles.scss` if global styles are involved
- **Parent components** that host this component

### Step 2: Read each file completely

NEVER read partially. Skim-reading causes missed context and wrong diagnoses.

### Step 3: Search for existing patterns

Before proposing a solution, check: "how do other components in this project solve this?"

- Grep for similar implementations in the codebase
- Check reference components: `saldo-credor-posterior`, `conciliacao`
- If the fix involves a Saffron component, search for other usages of that same component

### Step 4: Check existing tests

If `.spec.ts` exists for the affected component:
- Read it to understand what's already tested
- Note gaps in coverage relevant to the fix

---

## Phase 3: Consult Tools (mandatory cascade)

Follow this priority order. Do NOT skip levels.

```
1st ── Project skills (.claude/skills/)
│      saffron-patterns, saffron-angular-integration, saffron-tokens,
│      saffron-a11y, saffron-wijmo-grid, saffron-migration
│      Angular patterns: signals, inject(), OnPush, CUSTOM_ELEMENTS_SCHEMA
│
2nd ── Saffron MCP (when saf-* components are involved)
│      get-saffron-code        → component usage snippets
│      get-saffron-tokens      → CSS value → token mapping
│      get-saffron-a11y-attributes → accessibility specs
│      get-saffron-code-equivalent → HTML → Saffron mapping
│
3rd ── Project memories (.claude/memory/)
│      Previous feedback, known issues, saffron-learnings.md
│      CRITICAL: check if there's a known issue or feedback about this
│
4th ── angular-developer skill (global: ~/.claude/skills/angular-developer/)
│      Official Angular reference: signals, routing, DI, forms,
│      testing, components, styling, animations, A11y, CLI
│      35 reference files — consult BEFORE WebSearch
│
5th ── WebSearch
│      Only if angular-developer skill doesn't cover the topic
│      For bleeding-edge Angular features or niche topics
│
6th ── Saffron source code (LAST RESORT)
       C:\repositories\saffron_design_system
       Only if skills + MCP did not resolve
```

---

## Phase 4: Diagnose

### Step 1: Formulate written hypothesis

Write it explicitly:
> "The problem is **X** because **Y**, evidenced by **Z**"

### Step 2: List supporting evidence

- Specific code lines (file:line)
- MCP output that confirms the hypothesis
- Behavior observed vs expected

### Step 3: List what was ruled out

> "It is NOT **W** because we verified that..."

This proves you investigated alternatives, not just jumped to the first guess.

### Step 4: Context-specific checks

| Type | Additional check |
|---|---|
| Bug | Trace the complete data flow that causes the problem |
| Angular adjustment | Verify it uses modern patterns (signals, inject(), OnPush) |
| Saffron-related | Confirm component supports the change NATIVELY before any custom CSS |
| i18n | Check if the component/library has built-in locale support |

### Step 5: Present diagnosis to user

Present the COMPLETE diagnosis:
- The problem (with evidence)
- The proposed solution
- Why this solution (not alternatives)
- What will change (files, components)

### Step 6: Wait for confirmation

**STOP.** Do not implement until the user confirms the diagnosis.

---

## Phase 5: Implement

Only after user confirms the diagnosis:

1. **Fix based on confirmed diagnosis** — do exactly what was approved

2. **Follow the priority ladder:**
   ```
   native component prop > Saffron token > existing Angular pattern > custom CSS (last resort)
   ```
   NEVER create custom CSS for something the component already handles natively.

3. **Follow project patterns:**
   - Angular 19 standalone components
   - Signals (`signal()`, `computed()`, `input()`)
   - `inject()` for DI (never constructor injection)
   - `ChangeDetectionStrategy.OnPush`
   - `CUSTOM_ELEMENTS_SCHEMA` for Saffron components
   - `var(--saf-*)` tokens only, never hardcoded values

4. **Code names in English** — variables, methods, classes, interfaces

5. **Tests (when relevant):**
   - Bug fixes in logic/behavior → create or adjust tests
   - Visual/CSS-only changes → tests optional
   - i18n label changes → tests optional
   - New functionality → create tests

6. **Build verification:**
   ```bash
   ng build
   ```
   Zero errors AND zero warnings. Fix any issues before proceeding.

7. **Test verification (when tests exist):**
   ```bash
   ng test --watch=false
   ```
   All tests must pass.

---

## Red Flags — STOP and return to investigation

If you catch yourself thinking any of these, STOP immediately:

| Thought | Reality |
|---------|---------|
| "I'll just change this value and see if it works" | You don't have a diagnosis. Go back to Phase 2. |
| "I'll create a CSS class to fix this" | Did you check if the component has a native prop? Go to Phase 3. |
| "Couldn't find the native prop, I'll do an override" | Did you check MCP? Did you check the source? Go to Phase 3. |
| "This is simple, no need to investigate" | Simple bugs have root causes too. Follow the process. |
| "I'll skip consulting tools, I know how to fix this" | The cascade exists to prevent exactly this. Follow it. |
| "Let me implement first, then I'll present the diagnosis" | Iron Law violation. Diagnose THEN implement. |

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Issue is simple" | Simple issues have root causes too. Process is fast for simple bugs. |
| "I already know the answer" | Prior knowledge ≠ confirmed diagnosis. Verify. |
| "User is in a hurry" | Systematic investigation is FASTER than guess-and-check. |
| "Just this once" | Habits form. Follow the process every time. |

## Related Skills

- **superpowers:systematic-debugging** — For deep debugging (Phase 4 escalation)
- **superpowers:test-driven-development** — For writing tests (Phase 5)
- **superpowers:verification-before-completion** — Before claiming fix is done
- **angular-developer** (global) — Angular reference documentation
