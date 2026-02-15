# AI Specs Scaffold — Uso y prompts

Skill que genera el scaffold completo de **especificaciones para desarrollo con IA** (ai-specs) en la raíz de tu proyecto, alineado con un documento de arquitectura.

---

## Qué necesitas antes de usar el skill

1. **Nombre del proyecto** (obligatorio). Ej: `mi-api`, `portal-cliente`, `LTI-ATS`.
2. **Ruta de la carpeta raíz del proyecto** (obligatorio). La ruta absoluta o relativa donde está el repo (monorepo o proyecto único). Ej: `./mi-proyecto`, `~/repos/portal`.
3. **Documento de arquitectura** (recomendado). Archivo o contenido que define principios, decisiones técnicas, restricciones y convenciones. Es la fuente de verdad; el skill no inventa reglas no escritas.

Si no indicas la carpeta del proyecto, el skill **no creará** el scaffold.

---

## Cómo usarlo

1. **Abre un chat** donde tengas cargado el skill **ai-specs-scaffold** (o el agente que lo use).
2. **Incluye en el mensaje** (o en un archivo que referencies con `@`):
   - Nombre del proyecto.
   - Ruta de la carpeta raíz del proyecto.
   - Ruta o contenido del documento de arquitectura (opcional pero recomendado).
3. **Pide explícitamente** generar el scaffold de ai-specs / specs para IA.

El agente validará nombre y ruta, detectará tecnologías en esa carpeta y generará `ai-specs/`, `AGENTS.md`, `CLAUDE.md`, etc., más un `README.md` en la raíz del proyecto.

---

## Prompts de ejemplo

Copia y adapta estos prompts (cambia nombre del proyecto, ruta y archivo de arquitectura).

### 1. Uso básico (con nombre, ruta y doc de arquitectura)

```
Genera el scaffold de ai-specs para desarrollo con IA en mi proyecto.

- Nombre del proyecto: mi-app
- Carpeta raíz del proyecto: ./proyectos/mi-app
- Documento de arquitectura: @docs/arquitectura.md

Crea la estructura ai-specs, archivos de agentes (AGENTS.md, CLAUDE.md, etc.) y el README según el documento. Usa el skill ai-specs-scaffold.
```

### 2. Solo nombre y ruta (sin doc de arquitectura)

```
Quiero configurar mi proyecto para desarrollar con IA. Genera las specs (ai-specs) en la raíz del proyecto.

- Nombre del proyecto: portal-backoffice
- Carpeta raíz: /Users/yo/repos/portal-backoffice

Detecta las tecnologías (backend/frontend) y genera los archivos necesarios. Usa las plantillas agnósticas del skill ai-specs-scaffold.
```

### 3. Monorepo con varios subproyectos

```
Necesito el scaffold de ai-specs para el monorepo. La raíz del proyecto es ./monorepo (tiene carpetas backend, frontend y packages). Nombre del proyecto: Acme Platform. El documento de arquitectura está en @monorepo/docs/ARCHITECTURE.md. Genera las specs para cada tecnología detectada usando el skill ai-specs-scaffold.
```

### 4. Solo la base agnóstica (sin backend/frontend específicos)

```
Genera únicamente la parte agnóstica del scaffold de ai-specs: base-standards, documentation-standards, comandos enrich-us, update-docs, meta-prompt, AGENTS.md, CLAUDE.md, codex.md, GEMINI.md y README. Proyecto: api-gateway. Ruta: ./api-gateway. No generes backend-standards ni frontend-standards.
```

### 5. Petición corta (el agente pedirá lo que falte)

```
Configura ai-specs en mi proyecto. Nombre: workout-app. Ruta: ./projects/workout-app.
```

Si falta el documento de arquitectura, el agente puede usar las plantillas agnósticas y detectar el stack para generar backend/frontend-standards según lo que encuentre.

### 6. Con referencia explícita al skill

```
Usa el skill ai-specs-scaffold para generar las especificaciones para desarrollo con IA. Proyecto: {{nombre}}, carpeta raíz: {{ruta}}, documento de arquitectura: @{{archivo}}. No asumas nada que no esté en el documento.
```

---

## Qué se genera

En la **raíz del proyecto** que indiques. **Importante:** se cataloga **por tecnología**, no por proyecto. Se generan specs y un agente **por cada tecnología** detectada (java, node, angular, react, vue), con nombres `<tecnologia>-standards.mdc` y `<tecnologia>-developer.md` (ej. java-developer.md, node-developer.md, angular-developer.md).

| Ubicación | Contenido |
|-----------|-----------|
| `ai-specs/specs/` | base-standards.mdc (enlaces a cada tecnología), documentation-standards.mdc; **por cada tecnología**: `<tecnologia>-standards.mdc` (java, node, angular, react, vue); api-spec.yml, data-model.md, development_guide.md, prompts.md |
| `ai-specs/.commands/` | enrich-us.md, update-docs.md, meta-prompt.md; **por cada tecnología**: plan-&lt;tecnologia&gt;-ticket.md, develop-&lt;tecnologia&gt;.md (ej. plan-java-ticket.md, develop-node.md) |
| `ai-specs/.agents/` | **Un agente por tecnología**: `<tecnologia>-developer.md` (ej. java-developer.md, node-developer.md, angular-developer.md) |
| `ai-specs/changes/` | Carpeta para planes de implementación (vacía) |
| Raíz | AGENTS.md, CLAUDE.md, codex.md, GEMINI.md |
| `.cursor/rules/` | use-base-rules.mdc (enlaces a base, doc y cada &lt;tecnologia&gt;-standards.mdc) |
| Raíz | README.md (referencias arquitectónicas, reglas, instrucciones de mantenimiento) |

Las **plantillas agnósticas** se copian desde `assets/agnostic-templates/`; los specs y agentes **por tecnología** se generan según las tecnologías detectadas (java, node, angular, react, vue).

---

## Resumen rápido

- **Obligatorio**: nombre del proyecto + carpeta raíz del proyecto.  
- **Recomendado**: documento de arquitectura (única fuente de verdad).  
- **Prompt mínimo**: indica nombre, ruta y pide “generar scaffold de ai-specs” o “configurar specs para desarrollo con IA”.  
- Si no indicas la carpeta del proyecto, el skill no crea nada.

Para más detalle sobre flujo, detección de tecnologías y plantillas agnósticas, ver el cuerpo del skill en **SKILL.md**.
