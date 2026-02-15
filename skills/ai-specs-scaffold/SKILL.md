---
name: ai-specs-scaffold
description: |
  Genera el scaffold completo de especificaciones para desarrollo con herramientas de IA (AI specs), en la raíz del proyecto indicado por el usuario, estrictamente alineado con un documento de arquitectura.
  Usar cuando el usuario solicite:
  (1) Crear o generar las specs/reglas para desarrollar con IA en un proyecto existente según documento de arquitectura
  (2) Scaffolding de ai-specs (carpeta ai-specs, AGENTS.md, CLAUDE.md, etc.) siguiendo principios y convenciones del documento técnico
  (3) Configurar un proyecto o monorepo para desarrollo asistido por IA con estándares por tecnología (backend, frontend, documentación)
  (4) Detectar subproyectos y tecnologías y generar los archivos specs necesarios por cada uno
  El documento de arquitectura es la única fuente de verdad; no asumir ni inferir decisiones no explícitas.
  Requiere obligatoriamente el nombre del proyecto y la ruta de la carpeta raíz del proyecto principal donde se creará el scaffold; si el usuario no indica la carpeta del proyecto, no crear el scaffold.
---

# AI Specs Scaffold

Genera la estructura completa de especificaciones para desarrollo con IA en la **raíz de la carpeta del proyecto** que el usuario indique, interpretando el documento de arquitectura de forma literal y generando specs por cada tecnología detectada.

## Requisitos obligatorios

Antes de generar cualquier archivo:

1. **Nombre del proyecto** (obligatorio). Solicitar al usuario si no lo ha indicado. Usarlo en README, títulos y referencias.
2. **Carpeta raíz del proyecto principal** (obligatorio). La ruta donde está el proyecto global (monorepo o proyecto único). **Si el usuario no indica esta carpeta, no crear el scaffold.** Todas las configuraciones (ai-specs, AGENTS.md, CLAUDE.md, etc.) se crean en esta raíz.

No asumir rutas por defecto ni crear specs en el directorio actual si no se ha confirmado la carpeta del proyecto.

## Flujo de trabajo

1. **Validar entradas**: Obtener nombre del proyecto y ruta de la carpeta raíz. Si falta la carpeta del proyecto, informar y detener.
2. **Cargar documento de arquitectura**: El usuario debe proporcionar el documento que define principios, decisiones técnicas, restricciones y lineamientos. Interpretarlo de forma literal.
3. **Detectar proyecto y subproyectos**: Inspeccionar la carpeta raíz y subcarpetas para identificar qué tecnologías existen (ver sección Detección de tecnologías).
4. **Generar scaffold por tecnología**: Crear en la raíz del proyecto la estructura `ai-specs/`, archivos de configuración de agentes y README.md según lo detectado y lo definido en el documento de arquitectura.
5. **Generar README.md**: Documentar referencias arquitectónicas, reglas/convenciones y instrucciones para extender y mantener el proyecto.

## Detección de proyectos y tecnologías

**Regla clave:** Catalogar **por tecnología**, no por proyecto ni por rol genérico (backend/frontend). Los archivos de specs y agentes usan el nombre de la **tecnología** como slug: `<tecnologia>-standards.mdc`, `<tecnologia>-developer.md`, y comandos `plan-<tecnologia>-ticket.md`, `develop-<tecnologia>.md`. Ejemplo: `java-developer.md`, `node-developer.md`, `angular-developer.md`.

En la **carpeta raíz del proyecto**, inspeccionar subcarpetas directas (p. ej. `ms-shop/`, `demo-api/`, `workout-app/`, `backend/`, `frontend/`, `apps/*`, `packages/*`). Para cada subcarpeta que sea un proyecto ejecutable o librería relevante:

- **Identificar la tecnología** del proyecto (y opcionalmente el nombre de la carpeta para globs):
  - **java**: Spring Boot, Maven/Gradle, `src/main/java` → slug `java`.
  - **node**: Node/Express/TypeScript, `package.json` con express o servidor → slug `node`.
  - **angular**: `angular.json` o `@angular/core` → slug `angular`.
  - **react**: `package.json` con `react` → slug `react`.
  - **vue**: `package.json` con `vue` → slug `vue`.

Resultado esperado: una **lista de tecnologías** detectadas (sin duplicados) y, por cada una, las carpetas que la usan. Ej. tecnologías `[java, node, angular]` con `java` en `demo-api/`, `node` en `ms-shop/`, `angular` en `workout-app/`. Se genera **un** `<tecnologia>-standards.mdc` y **un** `<tecnologia>-developer.md` por tecnología; los `globs` pueden abarcar todas las carpetas que usen esa tecnología.

- **API REST**: Si algún proyecto expone API HTTP, considerar `api-spec.yml` y `data-model.md` si hay persistencia.

## Estructura a generar (en la raíz del proyecto)

Todo se crea **en la raíz de la carpeta del proyecto** indicada por el usuario.

**Specs y agentes por tecnología:** Por cada **tecnología detectada** (java, node, angular, react, vue) se genera un archivo de estándares y un agente con el slug de esa tecnología: `<tecnologia>-standards.mdc` y `<tecnologia>-developer.md`. Comandos: `plan-<tecnologia>-ticket.md` y `develop-<tecnologia>.md`. No usar nombres de proyecto (ms-shop, demo-api) ni roles genéricos (backend, frontend).

Ejemplo con tecnologías java, node, angular (proyectos demo-api, ms-shop, workout-app):

```
<raíz del proyecto>/
├── ai-specs/
│   ├── specs/
│   │   ├── base-standards.mdc           # Reglas base + enlaces a cada <tecnologia>-standards.mdc
│   │   ├── documentation-standards.mdc
│   │   ├── java-standards.mdc           # Spring Boot (globs: demo-api/**)
│   │   ├── node-standards.mdc           # Node/Express (globs: ms-shop/**)
│   │   ├── angular-standards.mdc         # Angular (globs: workout-app/**)
│   │   ├── api-spec.yml, data-model.md, development_guide.md, prompts.md
│   ├── changes/
│   ├── .commands/
│   │   ├── enrich-us.md, update-docs.md, meta-prompt.md
│   │   ├── plan-java-ticket.md          # Plan para proyectos Java
│   │   ├── plan-node-ticket.md          # Plan para proyectos Node
│   │   ├── plan-angular-ticket.md       # Plan para proyectos Angular
│   │   ├── develop-java.md              # Referencia .agents/java-developer.md
│   │   ├── develop-node.md
│   │   └── develop-angular.md
│   └── .agents/
│       ├── java-developer.md             # Rol Java/Spring Boot (globs: carpetas Java)
│       ├── node-developer.md             # Rol Node/Express (globs: carpetas Node)
│       └── angular-developer.md         # Rol Angular (globs: carpetas Angular)
├── AGENTS.md, CLAUDE.md, codex.md, GEMINI.md
├── .cursor/rules/use-base-rules.mdc      # Enlaces a base + doc + cada <tecnologia>-standards.mdc
└── README.md
```

- **base-standards.mdc**: Principios nucleares y **lista de enlaces a cada** `ai-specs/specs/<tecnologia>-standards.mdc` (java, node, angular, etc.). Incluir siempre `documentation-standards.mdc`.
- **&lt;tecnologia&gt;-standards.mdc** (uno por tecnología detectada): Convenciones para esa tecnología (stack, estructura, testing). Frontmatter `globs` con las rutas de todas las carpetas que usen esa tecnología (ej. `["demo-api/**"]` para java, `["ms-shop/**"]` para node). Slug de tecnología: `java`, `node`, `angular`, `react`, `vue`.
- **documentation-standards.mdc**: Proceso de actualización de documentación, idioma. Agnóstico.
- **api-spec.yml**, **data-model.md**, **development_guide.md**, **prompts.md**: Como antes.
- **.commands/**: Por **cada tecnología** generar **plan-&lt;tecnologia&gt;-ticket.md** y **develop-&lt;tecnologia&gt;.md** que referencien `ai-specs/specs/<tecnologia>-standards.mdc` y `ai-specs/.agents/<tecnologia>-developer.md`. Comandos agnósticos: enrich-us, update-docs, meta-prompt.
- **.agents/**: **Un agente por tecnología**: `<tecnologia>-developer.md` (ej. `java-developer.md`, `node-developer.md`, `angular-developer.md`). Rol y globs para esa tecnología; referencia a `ai-specs/specs/<tecnologia>-standards.mdc`.
- **use-base-rules.mdc**: Enlaces a base-standards, documentation-standards y **a cada** `<tecnologia>-standards.mdc` generado.
- **AGENTS.md, CLAUDE.md, codex.md, GEMINI.md**: Referencia a `ai-specs/specs/base-standards.mdc`.

## Contenido del README.md

Generar un **README.md** en la raíz del proyecto que documente:

1. **Referencias arquitectónicas clave**
   - Enlace o mención al documento de arquitectura usado como fuente de verdad.
   - Estructura de `ai-specs/` y propósito de cada carpeta/archivo principal (specs, changes, .commands, .agents).
   - Cómo los distintos copilots (Claude/Cursor, Codex, Gemini) cargan las reglas vía AGENTS.md, CLAUDE.md, codex.md, GEMINI.md.

2. **Reglas, convenciones y buenas prácticas**
   - Resumen de `base-standards.mdc` (principios, idioma, tipado, TDD).
   - Lista de specs por tecnología: `ai-specs/specs/<tecnologia>-standards.mdc` (java, node, angular, etc.) y cuándo usar cada uno. Referencia a documentation-standards.
   - Convenciones que el documento de arquitectura explicite.

3. **Instrucciones para extender, mantener y evolucionar**
   - Editar `base-standards.mdc` como núcleo; cada tecnología tiene su `<tecnologia>-standards.mdc` y su agente `<tecnologia>-developer.md`.
   - Cuándo actualizar api-spec.yml, data-model.md, development_guide.md.
   - Uso de `.commands`: plan-&lt;tecnologia&gt;-ticket y develop-&lt;tecnologia&gt; por cada tecnología (ej. plan-java-ticket, develop-node); enrich-us, update-docs, meta-prompt son agnósticos. Cada comando referencia el agente y standards de esa tecnología.
   - Recordar que el documento de arquitectura es la única fuente de verdad; cambios de criterio deben reflejarse primero allí y luego en los specs.

El README debe ser conciso pero suficiente para que un desarrollador o un agente de IA entienda cómo usar y mantener las AI specs del proyecto.

## Reglas de interpretación

- **Documento de arquitectura**: Es la única fuente de verdad. No añadir principios, convenciones o archivos que no estén explícitos o no se deduzcan literalmente del documento.
- **Detección**: Identificar las **tecnologías** presentes (java, node, angular, react, vue) y qué carpetas usan cada una. Generar **un** `<tecnologia>-standards.mdc` y **un** `<tecnologia>-developer.md` por tecnología (ej. java-developer.md, node-developer.md); no usar nombres de proyecto ni roles genéricos backend/frontend.
- **Ruta única**: Todo el scaffold vive en la raíz del proyecto indicada por el usuario. No crear `ai-specs` en subproyectos a menos que el documento de arquitectura lo exija explícitamente (p. ej. monorepo con specs por paquete).
- **Idioma**: Mantener el idioma que el documento de arquitectura indique para código y documentación (p. ej. inglés en specs y README si así se establece).

## Plantillas agnósticas

Existen **plantillas que pueden generarse de forma agnóstica** (independientes del stack). Para esos artefactos, **copiar desde `assets/agnostic-templates/`** y solo parametrizar:

- **Nombre del proyecto**: sustituir `{{PROJECT_NAME}}` en api-spec.yml, development_guide.md y README.
- **Rutas**: ya usan `ai-specs/specs/`; no cambiar salvo que el documento de arquitectura exija otra ruta.
- **base-standards.mdc** y **use-base-rules.mdc**: incluir un enlace por cada **tecnología** detectada a `ai-specs/specs/<tecnologia>-standards.mdc` (java, node, angular, etc.). Generar esa lista dinámicamente según las tecnologías.

Archivos agnósticos disponibles en assets:
- `base-standards.mdc`, `documentation-standards.mdc`
- `enrich-us.md`, `update-docs.md`, `meta-prompt.md`
- `AGENTS.md`, `CLAUDE.md`, `codex.md`, `GEMINI.md`
- `use-base-rules.mdc` (para .cursor/rules/)
- Esqueletos: `api-spec.yml`, `data-model.md`, `development_guide.md`, `prompts.md`

Detalle de qué es agnóstico y qué no: [references/agnostic-templates.md](references/agnostic-templates.md).

## Referencias del skill

- [references/agnostic-templates.md](references/agnostic-templates.md): Lista de plantillas agnósticas y uso; copiar desde `assets/agnostic-templates/`.
- [references/commands-templates.md](references/commands-templates.md): Plantillas de comandos dependientes del stack (plan-backend, plan-frontend, develop-backend, develop-frontend).
- [references/specs-snippets.md](references/specs-snippets.md): Fragmentos de frontmatter y secciones para backend-standards, frontend-standards cuando se generen por tecnología.
- [references/copilot-root-files.md](references/copilot-root-files.md): Contenido mínimo para AGENTS.md, CLAUDE.md, codex.md, GEMINI.md (también en assets/agnostic-templates/).

Usar plantillas agnósticas para la base; adaptar o generar el resto según documento de arquitectura y tecnologías detectadas.
