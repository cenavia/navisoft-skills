---
name: web-layout-specialist
description: Genera el esqueleto visual completo de una página web o aplicación a partir de un PRD y style-guidelines (design tokens). Crea la maquetación con enfoque en performance, accesibilidad, responsive design y estructura semántica, respetando estrictamente los documentos fuente. Usar cuando el usuario pida maquetar, crear el layout, hacer el esqueleto visual, crear la estructura HTML, o diseñar la interfaz de una app o página web a partir de documentos de requisitos.
---

# Web Layout Specialist

Genera esqueletos visuales completos desde PRD + style-guidelines, con foco en performance, accesibilidad y responsive.

## Entradas obligatorias

Antes de generar cualquier archivo, solicita ambos documentos si no fueron proporcionados:

1. **PRD** — Product Requirements Document: secciones, páginas, funcionalidades, flujos de usuario.
2. **Style-guidelines** — Design tokens: colores, tipografías, espaciados, breakpoints, componentes, border-radius, sombras.

Si falta alguno, **detén el flujo** y pide lo que falta con un mensaje claro:
> "Para continuar necesito [el PRD / las style-guidelines con design tokens]. Sin este documento no puedo generar la maquetación correctamente."

---

## Workflow de maquetación

Sigue estos pasos en orden:

### Paso 1 — Análisis de documentos

Lee ambos documentos y extrae literalmente:
- **Del PRD**: tipo de app, páginas/vistas, secciones de cada página, componentes requeridos, flujos de navegación, estados de UI (vacío, carga, error, éxito).
- **De style-guidelines**: tokens de color, familias tipográficas, escala de espaciado, breakpoints, radio de bordes, sombras, componentes base.

Si detectas información crítica faltante (ej. breakpoints no definidos, colores no especificados), detén y solicita.

### Paso 2 — Selección de patrón de layout

Basado estrictamente en el PRD, elige el patrón (ver `reference.md`):

| Tipo de app (según PRD) | Patrón |
|---|---|
| Dashboard / Admin panel | Sidebar fijo + Main scrollable |
| Chat / Asistente conversacional | Split Pane (sidebar + área de chat) |
| Generador de contenido AI | Split Editor (input \| output) |
| Landing page / Marketing | Hero + Secciones verticales |
| Documentación / Blog | Holy Grail (header + content + sidebar + footer) |
| Catálogo / E-commerce | Card Grid + Filtros |
| App móvil / PWA | Bottom Nav + Pantallas apiladas |
| Análisis de documentos | Document Viewer + AI Panel |

### Paso 3 — Estructura de carpetas y archivos

Genera la estructura de carpetas usando **únicamente** lo especificado en el PRD y style-guidelines. Si no se indica tecnología/framework, usa HTML + CSS nativo con los tokens como variables CSS.

Estructura base mínima:
```
proyecto/
├── index.html           # (o la página principal del PRD)
├── styles/
│   └── tokens.css       # Variables CSS de los design tokens
├── [pagina-2].html      # Una por cada vista del PRD
└── README.md            # Documentación generada
```

Si el PRD especifica framework (React, Vue, etc.), adapta la estructura sin inventar convenciones no explícitas.

### Paso 4 — Generación del esqueleto HTML

Para cada página/vista del PRD:

1. Usa **HTML semántico** estrictamente:
   - `<header>` para cabeceras
   - `<nav>` para navegación
   - `<main>` para contenido principal
   - `<aside>` para sidebars
   - `<section>` para secciones con significado propio
   - `<article>` para contenido independiente
   - `<footer>` para pies de página

2. Aplica **design tokens** como variables CSS (`var(--color-primary)`, etc.) — nunca valores hardcodeados que no vengan de style-guidelines.

3. Incluye **todos los estados** que aparezcan en el PRD: vacío, cargando (skeleton), con datos, error, sin conexión.

4. Añade placeholders comentados donde irá el contenido dinámico:
   ```html
   <!-- [COMPONENTE: Tabla de usuarios — ver PRD sección 3.2] -->
   ```

### Paso 5 — Performance (solo técnicas sin dependencias externas)

Aplica estas técnicas sin necesitar librerías adicionales:
- `loading="lazy"` en todas las imágenes no críticas.
- `<link rel="preload">` para fuentes críticas definidas en style-guidelines.
- CSS crítico inline en `<head>` para above-the-fold; resto en `<link>` diferido.
- `font-display: swap` en `@font-face` para fuentes del proyecto.
- `will-change: transform` solo en elementos animados especificados en style-guidelines.
- SVG inline para iconos pequeños y críticos (evita peticiones HTTP extra).
- `<picture>` + `srcset` para imágenes responsivas si el PRD las menciona.

### Paso 6 — Accesibilidad

Aplica en todo el esqueleto:
- `lang="[idioma del PRD]"` en `<html>`.
- `aria-label` descriptivos en todos los botones sin texto visible.
- `aria-live="polite"` en contenedores de respuestas dinámicas (chat, resultados AI, notificaciones).
- `aria-busy="true"` durante estados de carga.
- `role="alert"` en mensajes de error.
- `alt` descriptivo en todas las imágenes (o `alt=""` si son decorativas).
- Orden lógico de `tabindex` para navegación por teclado.
- Contraste mínimo WCAG AA (4.5:1 para texto normal, 3:1 para texto grande) — verifica con los colores del style-guidelines.
- Focus visible en todos los elementos interactivos (`outline` no removido sin alternativa).

### Paso 7 — Responsive design

Usa **únicamente** los breakpoints definidos en style-guidelines. Si no están definidos, usa los de Tailwind CSS como fallback y documéntalo.

Patrón Mobile First obligatorio:
- Estilos base → móvil.
- Luego sobreescribir progresivamente hacia tablet y desktop.

Para cada sección del PRD, define el comportamiento en cada breakpoint:
- Mobile: columna única, navegación colapsada.
- Tablet: sidebar colapsable, grid 2 columnas.
- Desktop: layout completo según patrón elegido.

### Paso 8 — README.md del proyecto

Genera un `README.md` en el proyecto con:
- **Referencias técnicas**: cita textual de qué parte del PRD y style-guidelines corresponde a cada decisión.
- **Reglas de tokens**: cómo usar variables CSS, qué no cambiar.
- **Breakpoints activos** con tabla de comportamiento por vista.
- **Checklist de extensión**: qué verificar antes de añadir nuevas vistas.
- **Instrucciones de mantenimiento**: cómo evolucionar sin desviarse de los documentos fuente.

---

## Reglas operativas

- **Nunca inventar**: colores, tipografías, breakpoints, espaciados, componentes ni secciones que no estén en los documentos.
- **Solo tecnologías explícitas**: no agregar frameworks, librerías o dependencias que el PRD/style-guidelines no mencionen.
- **Placeholders visibles**: los elementos aún sin contenido real llevan comentario HTML descriptivo.
- **Un archivo por vista**: cada página del PRD es un archivo separado.
- **Tokens centralizados**: todos los valores de diseño viven en `styles/tokens.css`; las páginas los consumen vía `var()`.
- **Sin comentarios que narren**: los comentarios en código solo explican decisiones de maquetación no obvias, nunca "aquí va el botón".

## Recursos de referencia

- Para patrones de layout y templates HTML+CSS: ver [reference.md](reference.md)
- Para checklists de verificación: ver [checklist.md](checklist.md)
