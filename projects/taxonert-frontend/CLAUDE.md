# Project Instructions

## Front Engineer Pipeline (mandatory)

Whenever the user asks to create, adjust, fix, or modify a screen/component,
follow the `front-engineer` skill automatically — without needing explicit
mention like `@front-engineer`.

The skill detects the scenario and chooses the correct pipeline:
- Design/creation (Figma URL or new screen) → Saffron Orchestrator
- Fix/adjustment/bug/translation → Investigation workflow
- Migration to Saffron → Saffron Orchestrator (migration mode)

Indicators that the pipeline should activate:
- Creating a new screen/component
- Adjusting/modifying an existing screen (swap, add, remove, move elements)
- Fixing bugs in screens or components
- Translating labels or formatting
- Migrating a screen to Saffron
- Any reference to a Figma URL for implementation
- Any visual or behavioral change in components using saf-*
