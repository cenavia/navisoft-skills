# Plantillas de comandos (.commands)

Usar como base al generar los archivos en `ai-specs/.commands/`.

**Catalogar por tecnología:** Generar **por cada tecnología** detectada (java, node, angular, react, vue) los comandos `plan-<tecnologia>-ticket.md` y `develop-<tecnologia>.md`, referenciando `ai-specs/specs/<tecnologia>-standards.mdc` y `ai-specs/.agents/<tecnologia>-developer.md`. Ejemplo: plan-java-ticket.md, develop-java.md, plan-node-ticket.md, develop-node.md, plan-angular-ticket.md, develop-angular.md. No usar nombres de proyecto (ms-shop, demo-api).

## enrich-us.md

```markdown
Please analyze and fix the Jira ticket: $ARGUMENTS.

Follow these steps:

1. Use Jira MCP to get the ticket details (if ticket id/number or keywords are provided). If the mention is a local file, skip MCP.
2. Act as a product expert with technical knowledge.
3. Understand the problem described in the ticket.
4. Decide whether the User Story is sufficiently detailed (full description, fields, endpoints, files to modify, completion steps, documentation/tests, non-functional requirements).
5. If the user story lacks detail, provide an improved story that is clearer and more specific. Use technical context from project documentation. Return in markdown format.
6. If using Jira: update the ticket adding the new content, marking sections [original] and [enhanced]. Use proper formatting (lists, code snippets).
7. If ticket status was "To refine", move to "Pending refinement validation" (when using Jira).
```

## plan-<tecnologia>-ticket.md (uno por tecnología)

Generar **un** `plan-<tecnologia>-ticket.md` por cada tecnología detectada (ej. plan-java-ticket.md, plan-node-ticket.md, plan-angular-ticket.md). Slug: java, node, angular, react, vue.

- **Role**: Expert architect para **esa** tecnología (Java/Spring, Node/Express, Angular, etc.).
- **Goal**: Plan paso a paso para el ticket en proyectos de esa tecnología.
- **Process**: (1) Adoptar rol de `ai-specs/.agents/<tecnologia>-developer.md`. (2) Analizar ticket. (3) Proponer plan aplicando `ai-specs/specs/<tecnologia>-standards.mdc` y documentation-standards. (4) Solo plan, sin código.
- **Output**: Markdown en `ai-specs/changes/[ticket_id]_<tecnologia>.md` (ej. SCRUM-10_java.md) con: Header, Overview, Architecture Context, Implementation Steps, Implementation Order, Testing Checklist.
- **References**: Branch y workflow desde `ai-specs/specs/<tecnologia>-standards.mdc`; doc desde documentation-standards.mdc.

## develop-<tecnologia>.md (uno por tecnología)

Generar **un** `develop-<tecnologia>.md` por cada tecnología (ej. develop-java.md, develop-node.md, develop-angular.md).

```markdown
Please analyze and fix the Jira ticket: $ARGUMENTS for the **<tecnologia>** stack (e.g. Java/Spring, Node/Express, Angular).

Follow these steps:

1. Understand the problem described in the ticket.
2. Search the codebase in the folders that use this technology (see ai-specs/specs/<tecnologia>-standards.mdc globs) for relevant files.
3. Start a new branch (e.g. feature/TICKET-123-<tecnologia>).
4. Implement following the plan in ai-specs/changes/[ticket_id]_<tecnologia>.md if it exists, in order: tests, code, documentation.
5. Ensure code passes linting and type checking.
6. Stage only files affected by the ticket. Create a descriptive commit message.
7. Push and create a PR with the ticket ID for traceability.

Use project rules from ai-specs/specs/base-standards.mdc, ai-specs/specs/<tecnologia>-standards.mdc, and ai-specs/specs/documentation-standards.mdc. Adopt the role defined in ai-specs/.agents/<tecnologia>-developer.md when working on this technology.
```

## update-docs.md

```markdown
Use ai-specs/specs/documentation-standards.mdc to update whatever documentation is needed according to the changes made.
```

## meta-prompt.md

```markdown
# Instructions

You are an expert in prompt engineering.
Given the following prompt, prepare it using best practices for structure (role, objective, context) and format to achieve a precise and exhaustive result. Stick only to the requested objective.

# Original prompt:
[The prompt that the user introduces in the command]
```
