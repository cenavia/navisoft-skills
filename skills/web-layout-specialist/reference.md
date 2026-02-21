# Referencia Técnica — Web Layout Specialist

Guía de patrones, templates base y decisiones de maquetación.
Fuentes: `guia-maquetacion-web-tailwind.md`, `guia-layouts-ai-apps.md`, `flowbite-application-ui-blocks.md`.

---

## Patrones de layout

| Patrón | Estructura base | Cuándo usarlo |
|--------|----------------|---------------|
| **Dashboard / Sidebar** | `flex h-screen` → `<aside w-64>` + `<main flex-1 overflow-y-auto>` | Admin panels, apps de gestión |
| **Holy Grail** | `flex flex-col` → header + `flex` (sidebar + main) + footer | Blogs, documentación |
| **Card Grid** | `grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3` | Catálogos, portafolios |
| **Split Pane** | `flex h-screen` → sidebar historial + main chat | Chat / asistente conversacional |
| **Split Editor** | `flex` → `div w-96` (input) + `div flex-1` (output) | Generadores de contenido AI |
| **Hero + Secciones** | `flex flex-col` → nav sticky + sections verticales | Landing pages |
| **AI Overlay Panel** | Main + `<aside>` derecho fijo | Notion-style, Copilot |
| **Wizard / Stepper** | `flex flex-col` → progress bar + step content | Onboarding, flujos guiados |

---

## Estructura semántica obligatoria

```html
<!DOCTYPE html>
<html lang="[idioma-del-PRD]">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="description" content="[descripción del PRD]">
  <title>[Nombre del PRD]</title>
  <!-- Preload de fuentes críticas de style-guidelines -->
  <link rel="preload" href="[fuente-primaria]" as="font" type="font/woff2" crossorigin>
  <!-- CSS de tokens primero -->
  <link rel="stylesheet" href="styles/tokens.css">
  <!-- CSS crítico inline para above-the-fold -->
  <style>/* tokens críticos + layout base */</style>
</head>
<body>
  <a href="#main-content" class="sr-only focus:not-sr-only">Saltar al contenido</a>

  <header role="banner">
    <nav aria-label="Navegación principal">...</nav>
  </header>

  <main id="main-content" tabindex="-1">
    <!-- Secciones del PRD -->
  </main>

  <footer role="contentinfo">...</footer>
</body>
</html>
```

---

## tokens.css — Estructura base

```css
/* styles/tokens.css */
/* TODOS los valores vienen de style-guidelines. No hardcodear aquí. */

:root {
  /* Colores — nombres exactos del style-guidelines */
  --color-[nombre]: [valor];

  /* Tipografía */
  --font-heading: '[familia]', [fallback];
  --font-body:    '[familia]', [fallback];
  --font-mono:    '[familia]', monospace;

  /* Escala de espaciado */
  --spacing-[n]: [valor];

  /* Border radius */
  --radius-[nombre]: [valor];

  /* Sombras */
  --shadow-[nombre]: [valor];

  /* Breakpoints como custom properties (referencia) */
  /* --bp-sm: 640px; -- no usable en media queries, documentar aquí como referencia */
}
```

---

## Breakpoints Tailwind (fallback si style-guidelines no los define)

| Prefijo | Min-width | Dispositivo |
|---------|-----------|-------------|
| base    | 0px       | Móvil pequeño |
| `sm:`   | 640px     | Móvil grande |
| `md:`   | 768px     | Tablet |
| `lg:`   | 1024px    | Laptop |
| `xl:`   | 1280px    | Desktop |
| `2xl:`  | 1536px    | Desktop grande |

Nota: documenta en README.md si usas breakpoints de Tailwind como fallback.

---

## Grillas responsivas comunes

```html
<!-- 1→2→3→4 columnas -->
<div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4 gap-6">

<!-- Contenido + Sidebar (2/3 + 1/3) -->
<div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
  <main class="lg:col-span-2">...</main>
  <aside class="lg:col-span-1">...</aside>
</div>

<!-- Sidebar fijo + contenido flexible -->
<div class="flex h-screen overflow-hidden">
  <aside class="w-64 flex-shrink-0 overflow-y-auto">...</aside>
  <main class="flex-1 overflow-y-auto">...</main>
</div>

<!-- Centrado con max-width -->
<div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
```

---

## Estados de UI — Plantillas

### Estado de carga (skeleton)
```html
<div role="status" aria-busy="true" aria-label="Cargando contenido">
  <div class="skeleton h-4 w-full mb-2"></div>
  <div class="skeleton h-4 w-5/6 mb-2"></div>
  <div class="skeleton h-4 w-4/6"></div>
</div>
```
```css
.skeleton {
  background: linear-gradient(90deg, var(--color-surface) 25%, var(--color-surface-elevated) 50%, var(--color-surface) 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
  border-radius: var(--radius-sm);
}
@keyframes shimmer {
  0%   { background-position: -100% 0; }
  100% { background-position: 100% 0; }
}
```

### Estado vacío
```html
<section aria-label="Sin resultados" class="empty-state">
  <!-- Ícono SVG inline -->
  <p>[Mensaje del PRD o genérico]</p>
  <!-- CTA si el PRD lo especifica -->
</section>
```

### Estado de error
```html
<div role="alert" aria-live="assertive">
  <p>[Mensaje de error]</p>
  <button type="button">Reintentar</button>
</div>
```

### Estado de procesamiento AI (si aplica)
```html
<div aria-live="polite" aria-busy="true" aria-label="Procesando respuesta">
  <!-- Dots animados o spinner definidos en style-guidelines -->
</div>
```

---

## Navegación responsiva — patrón hamburguesa

```html
<nav aria-label="Navegación principal">
  <!-- Desktop: links visibles -->
  <ul class="hidden md:flex gap-6" role="list">
    <li><a href="#seccion">Sección</a></li>
  </ul>

  <!-- Mobile: botón hamburguesa -->
  <button
    aria-expanded="false"
    aria-controls="mobile-menu"
    aria-label="Abrir menú de navegación"
    class="md:hidden"
    type="button">
    <!-- SVG hamburguesa inline -->
  </button>

  <!-- Mobile: menú desplegable -->
  <ul id="mobile-menu" class="hidden md:hidden" role="list">
    <li><a href="#seccion">Sección</a></li>
  </ul>
</nav>
```

---

## Sidebar de aplicación — patrón Flowbite

```html
<!-- Botón hamburguesa móvil -->
<button
  data-drawer-target="sidebar"
  data-drawer-toggle="sidebar"
  aria-controls="sidebar"
  type="button"
  class="sm:hidden">
  <span class="sr-only">Abrir sidebar</span>
  <!-- SVG -->
</button>

<!-- Sidebar -->
<aside
  id="sidebar"
  aria-label="Navegación lateral"
  class="fixed top-0 left-0 z-40 w-64 h-full -translate-x-full sm:translate-x-0 transition-transform">
  <nav>
    <ul role="list">
      <li>
        <a href="#" aria-current="page">
          <!-- SVG icono aria-hidden="true" -->
          <span>Sección activa</span>
        </a>
      </li>
    </ul>
  </nav>
</aside>
```

---

## Performance — técnicas permitidas (sin dependencias)

```html
<!-- 1. Preload de fuentes críticas -->
<link rel="preload" href="font.woff2" as="font" type="font/woff2" crossorigin>

<!-- 2. Imágenes lazy -->
<img src="imagen.webp" alt="[descripción]" loading="lazy" width="800" height="600">

<!-- 3. Imagen responsiva -->
<picture>
  <source srcset="img-480.webp" media="(max-width: 480px)" type="image/webp">
  <source srcset="img-1200.webp" media="(min-width: 481px)" type="image/webp">
  <img src="img-1200.jpg" alt="[descripción]" loading="lazy" width="1200" height="800">
</picture>

<!-- 4. CSS diferido (no crítico) -->
<link rel="preload" href="styles/main.css" as="style" onload="this.onload=null;this.rel='stylesheet'">
<noscript><link rel="stylesheet" href="styles/main.css"></noscript>
```

```css
/* 5. font-display para fuentes del proyecto */
@font-face {
  font-family: '[nombre-de-style-guidelines]';
  src: url('[fuente].woff2') format('woff2');
  font-display: swap;
}
```

---

## Accesibilidad — reglas rápidas

| Elemento | Atributo requerido | Notas |
|---|---|---|
| `<img>` | `alt="[descripción]"` | `alt=""` si decorativa |
| `<button>` sin texto | `aria-label="[acción]"` | Siempre descriptivo |
| `<input>` | `<label>` asociado o `aria-label` | No usar solo `placeholder` |
| Ícono SVG | `aria-hidden="true"` | Si el texto ya describe la acción |
| Respuesta dinámica | `aria-live="polite"` | `assertive` solo para errores críticos |
| Modal | `role="dialog"` + `aria-modal="true"` + `aria-labelledby` | Focus trap obligatorio |
| Link skip | `<a href="#main-content" class="sr-only focus:not-sr-only">` | Primer elemento del `<body>` |
| Tab order | `tabindex` no negativo en elementos interactivos | Seguir orden visual del PRD |

---

## Patrones AI UX (si el PRD es una app de IA)

Aplicar solo si el PRD describe funcionalidades de IA:

- `aria-live="polite"` en el contenedor de respuestas.
- `aria-busy="true"` mientras la API procesa.
- Badge "Generado por IA" visible si el PRD lo requiere.
- Opciones feedback post-respuesta (útil / mejorar) si el PRD las menciona.
- Botón regenerar si el PRD lo especifica.
- Disclaimer "puede cometer errores" si el PRD lo indica.
- Estados de streaming: cursor parpadeante o dots animados según style-guidelines.

---

## Template de README.md del proyecto

```markdown
# [Nombre del Proyecto — del PRD]

## Referencias técnicas

### Secciones del PRD implementadas
| Vista | Archivo | Sección PRD |
|-------|---------|-------------|
| [nombre] | [archivo].html | [ref. del PRD] |

### Design tokens aplicados
| Token | Valor | Origen (style-guidelines) |
|-------|-------|--------------------------|
| `--color-primary` | `#xxxxxx` | Style-guidelines, sección X |

## Breakpoints activos
| Prefijo | Min-width | Comportamiento |
|---------|-----------|----------------|
| base | 0px | [descripción del PRD] |
| md: | 768px | [descripción del PRD] |
| lg: | 1024px | [descripción del PRD] |

## Reglas de tokens
- Nunca usar valores hardcodeados en los HTML; usar siempre `var(--token)`.
- Para añadir un nuevo color, primero agregarlo a `styles/tokens.css`.
- No modificar nombres de tokens existentes sin actualizar todos los archivos.

## Instrucciones de extensión
1. Para añadir una nueva vista: crear `[nombre].html`, consultar el PRD sección X.
2. Para actualizar colores: modificar `styles/tokens.css` → afecta toda la app.
3. Para nuevos breakpoints: agregarlos a `tokens.css` como referencia documentada.
4. Cualquier decisión de layout debe trazarse a una sección del PRD o style-guidelines.

## Checklist antes de hacer deploy
Ver `checklist.md` en la skill para la lista completa.
```
