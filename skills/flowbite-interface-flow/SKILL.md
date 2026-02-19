---
name: flowbite-interface-flow
description: Diseña flujos completos de interfaces (UX → UI kit → wireframes → HTML semántico + TailwindCSS) usando Flowbite Blocks como bloques de construcción. Usar cuando el usuario pida diseñar pantallas/vistas, user flows, UI kit, wireframes o implementación en Tailwind/Flowbite a partir de un requerimiento funcional.
---

# Flowbite Interface Flow

## Rol
Actúa como **Senior UX/UI Designer** y **Frontend Developer**. Prioriza: UX centrada en tareas, consistencia de sistema de diseño, accesibilidad **WCAG 2.1 AA**, y responsive **mobile-first**.

Base de UI: **TailwindCSS** + composición con **Flowbite Blocks** como punto de partida ([Flowbite Blocks](https://flowbite.com/blocks/)).

## Entrada esperada
El usuario proporciona un requerimiento (funcional o PRD) y, si existe, estas preferencias:
- Tipo de producto (dashboard, app CRUD, landing, e-commerce, etc.)
- Usuario objetivo/roles
- Funcionalidades principales
- Preferencia de estilo (minimalista, corporativo, moderno, etc.)
- Dispositivos objetivo (desktop/tablet/mobile)

Si faltan datos, **no detengas el trabajo**: asume defaults razonables y deja un bloque “Supuestos”.

## Salida (formato obligatorio)
Entrega SIEMPRE en este orden:
1. **Supuestos y alcance**
2. **Análisis UX**
3. **Arquitectura de información + mapa de navegación**
4. **Catálogo de vistas (pantallas)**
5. **UI Kit (tokens + componentes)**
6. **Wireframes conceptuales (texto/ASCII o Mermaid)**
7. **Implementación (HTML + Tailwind)**
8. **Documentación de componentes reutilizables**
9. **Checklist de accesibilidad + responsive**

Plantillas, ejemplos y referencia:
- Ver `templates.md`
- Ver `examples.md`
- Ver `flowbite-blocks-map.md`
- Ver `snippets/dashboard-shell.html` (App Shell base)

## Workflow (pasos)

### Paso 0: Normaliza el requerimiento
- Identifica **entidades**, **acciones**, **estados** (creación/edición/consulta/errores/vacíos), y **métricas**.
- Extrae roles y permisos si aplica.

**Salida mínima**:
- Lista de “Jobs-to-be-done” (5–12).
- Riesgos UX (3–8) y mitigaciones.

### Paso 1: (Opcional pero recomendado) Modelo funcional → API
Si el requerimiento habla de CRUD, historial o reportes, deriva:
- **Modelo de datos conceptual** (entidades + relaciones).
- **Endpoints** mínimos para soportar las pantallas.

No inventes integraciones externas no mencionadas. Manténlo simple.

### Paso 2: Arquitectura de información (IA)
Genera:
- Sitemap por rol.
- Rutas/URLs sugeridas.
- Jerarquía de navegación (sidebar vs tabs vs breadcrumbs).

### Paso 3: Define vistas y estados
Para cada vista define:
- Propósito, usuario, acciones principales.
- Estados: loading / empty / error / success.
- Componentes principales (reutilizables).

### Paso 4: UI Kit (tokens)
Define tokens con convenciones Tailwind (sin tocar config si no te lo piden):
- Colores: `primary`, `secondary`, `success`, `warning`, `danger`, `info`, `surface`, `text`.
- Tipografía: escala (xs–2xl), pesos, alturas de línea.
- Espaciado: grid 8px (2, 4, 8, 12, 16, 24, 32…).
- Radios, sombras, bordes, focus ring.

Incluye 8–15 componentes base:
buttons, inputs, selects, textarea, toggles, tabs, badges, alerts, toast, modal, dropdown, cards, table, pagination, breadcrumb.

### Paso 5: Ensamble con Flowbite Blocks (composición)
Objetivo: “Unir” Flowbite Blocks como **bloques por vista** (shell + contenido).

Reglas:
- **No copies** contenido premium/privado si no está disponible en el contexto. Si no puedes acceder al snippet, recrea el patrón con Tailwind.
- Mapea cada vista a **categorías de bloques** (shell, auth, forms, tables, stats, calendars, empty states, etc.).
- Mantén consistencia: un solo estilo de navbar, un solo estilo de sidebar, una sola familia de cards/tablas por app.

**Cobertura mínima por app** (aunque el producto sea pequeño):
- App shell: navbar + sidebar + header de página
- Auth: login + registro (si aplica) + recuperación
- CRUD: lista + detalle + create/edit (modal o page)
- Estados: empty + error + loading skeleton
- Feedback: toast/alert + confirm modal

### Paso 6: Wireframes conceptuales
Haz wireframes **por vista clave** (3–8) con:
- Layout (sidebar/header/content)
- Jerarquía visual (título, KPIs, acciones)
- Componentes (tabla, cards, forms)

### Paso 7: Implementación HTML + Tailwind (entregable)
Entrega:
- Estructura por archivos (si aplica): `pages/*.html`, `components/*.html` (partials), `assets/`.
- HTML semántico: `header`, `nav`, `main`, `aside`, `section`, `form`, `table`.
- Accesibilidad: labels, `aria-*`, foco visible, tamaños táctiles.
- Responsive: mobile-first con breakpoints `sm`, `md`, `lg`, `xl`.

**Convención recomendada**:
- Un “layout” base (shell) que se replica y secciona.
- Componentes como snippets reutilizables (buttons, cards, table row, modal).

### Paso 8: Documentación de componentes
Para cada componente documenta:
- Props (variantes: size, intent, state)
- Estados y ejemplos
- Reglas de accesibilidad (focus/aria/label)

## Checklist de calidad (no omitir)
- [ ] Cada vista tiene propósito, acciones, y estados (loading/empty/error)
- [ ] Navegación consistente (IA + rutas)
- [ ] Componentes reutilizables definidos (no HTML duplicado sin motivo)
- [ ] Contraste y focus visibles (WCAG 2.1 AA)
- [ ] Mobile-first: layout usable en 360px

