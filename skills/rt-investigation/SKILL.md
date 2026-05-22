---
name: rt-investigation
description: Investigation-first workflow for fixes in RT backend projects (NestJS/Express). Enforces reading code, finding patterns, consulting ecosystem tools, and diagnosing before implementing. Used by /rt-fix when the project is backend.
---

# RT Investigation

Systematic investigation workflow for backend fixes in RT (Reforma Tributaria) projects. Diagnose before you fix.

## When to Use

- Bug fixes in NestJS/Express services
- API adjustments (endpoints, DTOs, validation)
- Data flow fixes (Kafka consumers, TypeORM queries, migrations)
- Refactoring (restructuring without behavior change)
- Minor features (new endpoint, new field, new consumer)
- Any change that does NOT require a new project/service from scratch

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
   - API adjustment — endpoint behavior, response format, validation rules
   - Data flow fix — Kafka consumer, TypeORM query, migration issue
   - Refactor — restructuring without behavior change
   - Minor feature — small addition to existing functionality

2. **Identify affected layers** — which Clean Architecture layers does this touch?
   - `domain/` — entities, interfaces, exceptions
   - `application/` — use cases, consumers, schedulers
   - `infrastructure/` — TypeORM, Kafka, external clients
   - `presentation/` — controllers, DTOs, gateways

3. **State the expected outcome** — what should change after the fix?

---

## Phase 2: Investigate the Code

### Step 1: Locate ALL affected files

Do NOT stop at the obvious file. Trace the full dependency chain:
- **Controller/Gateway** that receives the request
- **Use case** that processes the logic
- **Repository interface** (domain) and **implementation** (infrastructure)
- **Entity/Model** that defines the data structure
- **DTOs** at the presentation layer
- **Kafka consumers** if the flow involves messaging
- **Module file** (`.module.ts`) for provider registration
- **Migration files** if schema changes are needed
- **Config/env** if environment variables are involved

### Step 2: Read each file completely

NEVER read partially. Skim-reading causes missed context and wrong diagnoses.

### Step 3: Search for existing patterns

Before proposing a solution, check: "how do other parts of this project solve this?"

- Grep for similar implementations in the codebase
- Check how other use cases handle the same type of operation
- If multi-tenant: check how tenant context propagates through the affected flow
- If Kafka: check how other consumers handle errors, retries, and acknowledgments

### Step 4: Check existing tests

If `.spec.ts` exists for the affected module:
- Read it to understand what's already tested
- Note gaps in coverage relevant to the fix

---

## Phase 3: Consult Tools (mandatory cascade)

Follow this priority order. Do NOT skip levels.

```
1st ── Project CLAUDE.md
│      Read the project's CLAUDE.md for architecture overview, commands,
│      conventions, and environment setup.
│      This is your primary reference for the project.
│
2nd ── Codebase patterns
│      Search the project itself for how similar problems are solved.
│      Other use cases, other consumers, other controllers.
│      The best reference is the code that already works.
│
3rd ── TaxOne Ecosystem MCP (when cross-service context is needed)
│      search_services      → find related services/libs
│      get_service_details  → understand a service's purpose and API
│      get_dependency_graph → trace dependencies between components
│      get_impact_analysis  → understand blast radius of changes
│
4th ── Project memories (.claude/memory/)
│      Previous feedback, known issues, architectural decisions.
│      CRITICAL: check if there's a known issue or feedback about this.
│
5th ── WebSearch
│      For NestJS/TypeORM/Kafka-specific questions not answered by code.
│      Official docs: docs.nestjs.com, typeorm.io, kafka.js.org
│
6th ── SonarQube (code quality check)
│      Use SONAR_MCP tools to check if the affected files have
│      existing issues that should be addressed alongside the fix.
```

### Multi-Tenant Checks (when applicable)

If the project uses multi-tenant (schema per tenant):
- Verify tenant context propagation in the affected flow
- Check if the fix needs to work across all tenant schemas
- Verify migration applies to all tenants (`npm run migration:run`)
- Check `TenantConnectionPoolService` if DataSource management is affected

### Kafka Checks (when applicable)

If the fix involves Kafka consumers/producers:
- Check consumer group configuration
- Verify error handling and retry strategy
- Check if offset management is affected
- Verify topic naming follows the pattern: `{tenant}.{domain}.{event}.v{version}`

---

## Phase 4: Diagnose

### Step 1: Formulate written hypothesis

Write it explicitly:
> "The problem is **X** because **Y**, evidenced by **Z**"

### Step 2: List supporting evidence

- Specific code lines (file:line)
- Log output or error messages
- Data flow trace showing where it breaks
- Behavior observed vs expected

### Step 3: List what was ruled out

> "It is NOT **W** because we verified that..."

This proves you investigated alternatives, not just jumped to the first guess.

### Step 4: Context-specific checks

| Type | Additional check |
|---|---|
| Bug | Trace the complete data flow from entry point to failure |
| API issue | Verify DTO validation, guards, interceptors in the chain |
| Data flow | Check TypeORM query, entity mapping, migration state |
| Kafka | Check consumer offset, deserialization, retry config |
| Multi-tenant | Verify tenant isolation is maintained |

### Step 5: Present diagnosis to user

Present the COMPLETE diagnosis:
- The problem (with evidence)
- The proposed solution
- Why this solution (not alternatives)
- What will change (files, layers)
- Risk assessment (low/medium/high)

### Step 6: Wait for confirmation

**STOP.** Do not implement until the user confirms the diagnosis.

---

## Phase 5: Implement

Only after user confirms the diagnosis:

1. **Fix based on confirmed diagnosis** — do exactly what was approved

2. **Follow Clean Architecture layers:**
   ```
   domain/ → pure business logic, no framework imports
   application/ → use cases, orchestration
   infrastructure/ → framework-specific implementations
   presentation/ → HTTP/WebSocket interface
   ```
   NEVER put business logic in controllers. NEVER import infrastructure in domain.

3. **Follow project patterns:**
   - NestJS modules, providers, dependency injection
   - TypeORM entities with decorators
   - DTOs with class-validator decorators
   - Exception filters for error handling
   - Guards for authentication/authorization

4. **Code names in English** — variables, methods, classes, interfaces

5. **Tests:**
   - Bug fixes → create or adjust unit tests covering the scenario
   - New functionality → create tests
   - Data flow changes → verify with integration tests if available
   - Use `jest` mocking patterns consistent with the project

6. **Build verification:**
   ```bash
   npm run build
   ```
   Zero errors required. Fix any issues before proceeding.

7. **Test verification:**
   ```bash
   npm test
   ```
   All tests must pass.

8. **Lint (if available):**
   ```bash
   npm run lint
   ```

---

## Red Flags — STOP and return to investigation

| Thought | Reality |
|---------|---------|
| "I'll just change this value and see if it works" | You don't have a diagnosis. Go back to Phase 2. |
| "This is simple, no need to investigate" | Simple bugs have root causes too. Follow the process. |
| "I'll skip consulting tools, I know how to fix this" | The cascade exists to prevent exactly this. Follow it. |
| "Let me implement first, then I'll present the diagnosis" | Iron Law violation. Diagnose THEN implement. |
| "I'll fix the entity and hope the migration works" | Check migration state. Trace the full data flow. |
| "I'll just add a try-catch around it" | That's hiding the bug, not fixing it. Find the root cause. |

---

## Related Skills

- `/rt-fix` — Orchestrates the full fix workflow (ADO + git + investigation)
- `git-workflow` (global) — Branch management, commits, PRs
- `ado-client` (global) — Azure DevOps API patterns
- `taxone-ecosystem MCP` — Cross-service dependency analysis
