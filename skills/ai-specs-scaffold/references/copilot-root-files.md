# Contenido mínimo para archivos de configuración de copilots (raíz del proyecto)

Estos archivos van en la **raíz del proyecto**. Todos deben apuntar a la misma fuente de verdad: `ai-specs/specs/base-standards.mdc`. Adaptar el nombre del proyecto si se menciona.

## AGENTS.md

```markdown
# Agent rules

All development rules and guidelines for this project are defined in:

**ai-specs/specs/base-standards.mdc**

Load and follow that document for coding, testing, documentation, and workflow. Specific standards (backend, frontend, documentation) are linked from base-standards.mdc.
```

## CLAUDE.md

```markdown
# Claude / Cursor configuration

Project rules and AI development specs are maintained in:

**ai-specs/specs/base-standards.mdc**

Always apply the principles and standards defined there. For backend, frontend, and documentation details, use the specific standards referenced in base-standards.mdc.
```

## codex.md

```markdown
# GitHub Copilot / Codex configuration

Development rules and guidelines for this project:

**ai-specs/specs/base-standards.mdc**

Follow that document for all code, tests, and documentation. Backend, frontend, and documentation standards are linked from it.
```

## GEMINI.md

```markdown
# Gemini configuration

Project development rules:

**ai-specs/specs/base-standards.mdc**

Apply the rules and standards defined there. See base-standards.mdc for links to backend, frontend, and documentation standards.
```

## .cursor/rules/use-base-rules.mdc (opcional)

```markdown
---
description: Load project base rules from ai-specs.
alwaysApply: true
---

Use the rules and standards defined in the project's ai-specs:

- **Base rules**: ai-specs/specs/base-standards.mdc
- **Backend** (if applicable): ai-specs/specs/backend-standards.mdc
- **Frontend** (if applicable): ai-specs/specs/frontend-standards.mdc
- **Documentation**: ai-specs/specs/documentation-standards.mdc

Do not assume or infer rules not explicitly stated in these documents.
```
