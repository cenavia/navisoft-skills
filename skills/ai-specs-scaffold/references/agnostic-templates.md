# Plantillas agnósticas (independientes del stack)

Estos artefactos del contexto ai-specs **no dependen de una tecnología concreta** (React, Node, Express, Spring, etc.) y pueden generarse o copiarse en cualquier proyecto, parametrizando solo nombre del proyecto y rutas (`ai-specs/specs`, etc.).

Usar estas plantillas cuando el skill genere el scaffold: permiten crear una base consistente sin reescribir contenido que es común a todos los proyectos. El documento de arquitectura puede sobrescribir o extender secciones concretas (p. ej. principios en base-standards, idioma en documentation-standards).

## Lista de plantillas agnósticas

| Artefacto | Ubicación en scaffold | Parametrización |
|-----------|------------------------|-----------------|
| **base-standards.mdc** | `ai-specs/specs/base-standards.mdc` | Principios del doc; **enlaces a cada** `ai-specs/specs/<tecnologia>-standards.mdc` (uno por tecnología: java, node, angular, react, vue). |
| **documentation-standards.mdc** | `ai-specs/specs/documentation-standards.mdc` | Ruta de specs (`ai-specs/specs`); idioma si el doc exige otro. |
| **enrich-us.md** | `ai-specs/.commands/enrich-us.md` | Ruta `@documentation` o equivalente; nombre del proyecto si se cita. |
| **update-docs.md** | `ai-specs/.commands/update-docs.md` | Solo la ruta al doc (p. ej. `ai-specs/specs/documentation-standards.mdc`). |
| **meta-prompt.md** | `ai-specs/.commands/meta-prompt.md` | Ninguna. |
| **AGENTS.md / CLAUDE.md / codex.md / GEMINI.md** | Raíz del proyecto | Ruta `ai-specs/specs/base-standards.mdc` (fija). |
| **use-base-rules.mdc** | `.cursor/rules/use-base-rules.mdc` | Enlaces a base-standards, documentation-standards y **a cada** `<tecnologia>-standards.mdc` generado (java, node, angular, etc.). |
| **Esqueleto OpenAPI** | `ai-specs/specs/api-spec.yml` | `info.title`, `info.description`, `servers`; `paths` vacío o con `/health` genérico. |
| **Esqueleto data-model.md** | `ai-specs/specs/data-model.md` | Título y párrafo intro; entidades las define el proyecto o el doc. |
| **Esqueleto development_guide.md** | `ai-specs/specs/development_guide.md` | Nombre del proyecto; prerequisitos y comandos según stack detectado (ese tramo ya no es agnóstico). |
| **Esqueleto prompts.md** | `ai-specs/specs/prompts.md` | Secciones genéricas (Enrich US, Plan ticket, Update docs). |

## Qué no es agnóstico (catalogar por tecnología)

Catalogar **por tecnología**, no por proyecto. Slugs: `java`, `node`, `angular`, `react`, `vue`. Generar **por cada tecnología detectada**:

- **&lt;tecnologia&gt;-standards.mdc**: Un archivo por tecnología (ej. `java-standards.mdc`, `node-standards.mdc`, `angular-standards.mdc`), con convenciones y globs para las carpetas que usen esa tecnología.
- **plan-&lt;tecnologia&gt;-ticket.md** y **develop-&lt;tecnologia&gt;.md**: Un comando de plan y uno de desarrollo por tecnología, referenciando `ai-specs/specs/<tecnologia>-standards.mdc` y `ai-specs/.agents/<tecnologia>-developer.md`.
- **.agents/&lt;tecnologia&gt;-developer.md**: Un agente por tecnología (ej. `java-developer.md`, `node-developer.md`, `angular-developer.md`), con rol y globs para esa tecnología.
- **api-spec.yml** contenido de `paths**: Es específico del dominio; solo el esqueleto YAML es agnóstico.
- **data-model.md** entidades y relaciones: Específicas; solo la cabecera es plantilla agnóstica.

## Uso en el skill

1. **Generar siempre (agnóstico)**: base-standards.mdc, documentation-standards.mdc, enrich-us, update-docs, meta-prompt, AGENTS/CLAUDE/codex/GEMINI, .cursor/rules/use-base-rules.mdc, esqueletos de api-spec.yml, data-model.md, development_guide.md, prompts.md.
2. **Parametrizar**: Rutas `ai-specs/specs`, nombre del proyecto, idioma si el doc lo exige.
3. **Sobrescribir con documento de arquitectura**: Principios en base-standards, reglas en documentation-standards, estructura de development_guide según stack.
4. **Generar por tecnología (no agnóstico)**: Por cada tecnología detectada (java, node, angular, react, vue), un `<tecnologia>-standards.mdc`, un `<tecnologia>-developer.md`, y comandos `plan-<tecnologia>-ticket.md` y `develop-<tecnologia>.md`. No usar nombres de proyecto ni backend/frontend genéricos.

Las plantillas literales (contenido completo de los archivos agnósticos del contexto) están en [../assets/agnostic-templates/](../assets/agnostic-templates/) para copiar y solo sustituir placeholders.
