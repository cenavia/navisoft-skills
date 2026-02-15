---
name: astro-tailwind-scaffold
description: |
  Genera esqueletos de proyectos frontend con Astro y Tailwind CSS siguiendo un documento de arquitectura.
  Usar cuando el usuario solicite:
  (1) Crear o generar un proyecto Astro + Tailwind desde cero según especificaciones de arquitectura
  (2) Scaffolding de un sitio estático o contenido con Astro siguiendo principios, estructura y convenciones definidas
  (3) Generar la estructura de carpetas, archivos base y configuraciones iniciales para un frontend Astro + Tailwind
  (4) Montar un esqueleto de proyecto alineado con reglas de Astro (routing, componentes, hidratación, Tailwind, SEO, accesibilidad)
  El documento de arquitectura es la única fuente de verdad; el scaffold debe respetarlo de forma literal.
  Requiere que el usuario indique el nombre del proyecto (obligatorio).
---

# Astro + Tailwind Scaffold

Genera un esqueleto de proyecto frontend con Astro, TypeScript y Tailwind CSS, alineado estrictamente con el documento de arquitectura proporcionado por el usuario.

## Obligatorio: nombre del proyecto

Antes de generar cualquier archivo, **solicitar al usuario el nombre del proyecto**. Si no lo ha indicado, preguntar explícitamente. No asumir un nombre por defecto. Usar ese nombre para:

- Carpeta raíz del proyecto
- `name` en `package.json`
- Título por defecto en layouts/SEO cuando corresponda

## Flujo de trabajo

1. **Obtener nombre del proyecto** (obligatorio).
2. **Cargar documento de arquitectura**: si el usuario adjuntó o referenció un documento (p. ej. `.mdc`, `.md`), usarlo como única fuente de verdad. Si no hay documento, usar [references/architecture-default.md](references/architecture-default.md) como referencia base Astro + Tailwind.
3. **Analizar el documento**: extraer estructura de carpetas, convenciones, stack (Astro, Tailwind, TypeScript), reglas de componentes, routing, estilos, SEO, testing y accesibilidad.
4. **Planificar**: lista de directorios y archivos a crear sin inventar decisiones no explícitas.
5. **Generar scaffold**: crear todos los archivos y carpetas.
6. **Generar README.md** del proyecto según la sección [README del proyecto](#readme-del-proyecto).

## Estructura de proyecto a generar

Respetar la estructura definida en el documento. Si el documento indica la estructura recomendada de Astro, usar:

```
<nombre-proyecto>/
├── src/
│   ├── components/
│   ├── layouts/
│   ├── pages/
│   ├── styles/
│   └── content/
│       └── config.ts          # content collections si aplica
├── public/
├── astro.config.mjs
├── tailwind.config.mjs
├── tsconfig.json
├── package.json
├── .gitignore
└── README.md
```

- **src/components/**: componentes reutilizables `.astro` (y opcionalmente React/Vue/Svelte si el doc lo permite).
- **src/layouts/**: layouts base (p. ej. `BaseLayout.astro`) con `<head>`, meta, y slot para contenido.
- **src/pages/**: rutas basadas en archivo; incluir `index.astro` y `404.astro`.
- **src/styles/**: estilos globales; importarlos desde el layout.
- **public/**: assets estáticos.

## Archivos de configuración

- **astro.config.mjs**: integración `@astrojs/tailwind`, TypeScript, y otras integraciones mencionadas en el doc (p. ej. `@astrojs/image`).
- **tailwind.config.mjs**: tema y extensiones si el doc lo indica; no añadir `@apply` si el documento prohíbe su uso.
- **tsconfig.json**: `"strict"` y rutas/aliases si el documento los especifica.
- **package.json**: scripts `dev`, `build`, `preview`, `astro`; dependencias `astro`, `@astrojs/tailwind`, `tailwindcss`, `typescript`.

## Componentes y páginas base

- **Layout**: `src/layouts/BaseLayout.astro` con `<html>`, `<head>` (charset, viewport), slot `<slot />`, y opcionalmente un componente SEO reutilizable si el doc menciona patrón SEO.
- **Página principal**: `src/pages/index.astro` que use el layout y muestre contenido mínimo.
- **404**: `src/pages/404.astro` para manejo de rutas no encontradas.
- **Componente SEO** (si el doc lo indica): p. ej. `src/components/SEO.astro` con props para title, description, canonical.

## Estilos

- Usar clases de utilidad Tailwind en componentes; no usar `@apply` si el documento lo prohíbe.
- Estilos globales en `src/styles/` e importarlos en el layout.
- Estilos con alcance con `<style>` en `.astro` cuando el doc lo indique.

## Contenido y routing

- Si el doc menciona content collections: `src/content/config.ts` y estructura bajo `src/content/` (p. ej. `blog/`).
- Rutas dinámicas con `getStaticPaths()` y convención `[...slug].astro` solo si el documento lo requiere; no añadir por defecto si no se menciona.

## README del proyecto

Generar un **README.md** en la raíz del proyecto generado que documente:

1. **Referencias arquitectónicas clave**
   - Enlace o cita al documento de arquitectura usado.
   - Stack (Astro, Tailwind, TypeScript, integraciones).
   - Estructura de carpetas y criterio (components, layouts, pages, styles, public).

2. **Reglas, convenciones y buenas prácticas**
   - Principios del doc: respuestas técnicas concisas, hidratación parcial, prioridad a estático y poco JS.
   - Convenciones de nombres (archivos, componentes, rutas).
   - Uso de Tailwind (utilidades, responsive, sin `@apply` si aplica).
   - Directivas `client:*` (load, idle, visible) y cuándo usarlas.
   - SEO (meta, canonical, componente SEO).
   - Accesibilidad (HTML semántico, ARIA, teclado).
   - Testing (unit, E2E, visual) si el doc lo menciona.

3. **Instrucciones para extender y mantener**
   - Cómo añadir páginas y rutas dinámicas.
   - Cómo añadir componentes y layouts.
   - Cómo gestionar contenido (Markdown/MDX, content collections).
   - Variables de entorno y despliegue (build, preview, plataformas estáticas).
   - Referencia a la documentación oficial de Astro para más detalle.

El README debe ser Markdown válido y autocontenido para que cualquier desarrollador pueda seguir la arquitectura sin el documento original a mano.

## Validación

- No introducir decisiones no explícitas en el documento (p. ej. frameworks de componentes adicionales, convenciones de nombres no definidas).
- Asegurar que cada archivo generado sea coherente con el doc (imports, sintaxis Astro/TS, uso de Tailwind).
- Comprobar que el proyecto pueda ejecutarse con `npm install` y `npm run dev` (incluir instrucciones en el README).

## Resumen de decisiones desde el documento de referencia

Cuando se use el documento tipo “Astro + Tailwind + Cursor rules”:

| Área | Regla |
|------|--------|
| Estructura | `src/components`, `layouts`, `pages`, `styles`; `public/`; `astro.config.mjs` |
| Componentes | `.astro`; props para datos; composición; `<Markdown />` cuando aplique |
| Routing | file-based en `src/pages`; dinámicos con `[...slug].astro` y `getStaticPaths()`; `404.astro` |
| Contenido | Markdown/MDX, frontmatter, content collections |
| Estilos | Scoped con `<style>` en .astro; globales en layouts; Tailwind con utilidades; **nunca @apply** |
| Hidratación | `client:load`, `client:idle`, `client:visible` de forma juiciosa |
| Datos | `Astro.props`, `getStaticPaths()`, `Astro.glob()` |
| SEO | `<head>`, canonical, patrón componente SEO |
| Integraciones | Configurar en `astro.config.mjs` (p. ej. `@astrojs/tailwind`, `@astrojs/image`) |
| Build | Comando build de Astro; variables de entorno; hosting estático |
| Testing | Unit para utilidades; E2E (p. ej. Cypress); regresión visual si aplica |
| Accesibilidad | HTML semántico, ARIA, navegación por teclado |
| TypeScript | Uso explícito para tipo y DX |

Si el usuario aporta un documento distinto, reemplazar esta tabla por las decisiones extraídas de ese documento y generar el scaffold en consecuencia.
