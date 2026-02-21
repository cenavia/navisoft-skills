# Checklists de Verificación — Web Layout Specialist

Usar antes de entregar cualquier maquetación generada con esta skill.

---

## Checklist 1 — Análisis de documentos

- [ ] Se leyó el PRD completo y se identificaron todas las vistas/páginas.
- [ ] Se identificaron todos los componentes mencionados en el PRD.
- [ ] Se extrajeron todos los estados de UI (vacío, carga, error, éxito, sin conexión).
- [ ] Se leyó el style-guidelines completo y se mapearon todos los tokens.
- [ ] Se verificó que no hay información crítica faltante (colores, tipografías, breakpoints).
- [ ] Si falta información crítica → flujo detenido y solicitud enviada al usuario.

---

## Checklist 2 — Estructura de archivos

- [ ] Existe `styles/tokens.css` con todos los design tokens del style-guidelines.
- [ ] Hay un archivo HTML por cada vista/página definida en el PRD.
- [ ] Existe `README.md` en la raíz del proyecto.
- [ ] No se crearon archivos ni carpetas que el PRD no requiera.
- [ ] No se agregaron dependencias (npm, CDN, librerías) que el PRD no mencione.

---

## Checklist 3 — HTML semántico

- [ ] `<!DOCTYPE html>` presente.
- [ ] `<html lang="[idioma-del-PRD]">` con idioma correcto.
- [ ] `<meta charset="UTF-8">` presente.
- [ ] `<meta name="viewport" content="width=device-width, initial-scale=1.0">` presente.
- [ ] `<title>` descriptivo según el PRD.
- [ ] Link "Saltar al contenido" como primer hijo de `<body>`.
- [ ] `<header>` con `role="banner"` para cabecera del sitio.
- [ ] `<nav>` con `aria-label` descriptivo.
- [ ] `<main id="main-content">` como contenedor del contenido principal.
- [ ] `<aside>` usado solo para contenido relacionado pero secundario.
- [ ] `<footer>` con `role="contentinfo"` para el pie del sitio.
- [ ] Jerarquía de headings correcta: un solo `<h1>` por página, `<h2>` para secciones, etc.
- [ ] No hay `<div>` donde debería ir un elemento semántico.

---

## Checklist 4 — Design tokens

- [ ] Todos los colores vienen de `var(--color-[nombre])`, no hardcodeados.
- [ ] Todas las tipografías usan `var(--font-[nombre])`.
- [ ] Todos los espaciados usan `var(--spacing-[n])` o las utilidades del sistema de diseño.
- [ ] Los border-radius usan `var(--radius-[nombre])`.
- [ ] Las sombras usan `var(--shadow-[nombre])`.
- [ ] Ningún valor de color, tamaño o espaciado fue inventado fuera del style-guidelines.

---

## Checklist 5 — Responsive design

- [ ] Los estilos base corresponden a móvil (Mobile First).
- [ ] Cada breakpoint usado está definido en el style-guidelines (o documentado como fallback Tailwind).
- [ ] La navegación es accesible y funcional en móvil (hamburguesa o equivalente del PRD).
- [ ] El sidebar/panel lateral colapsa en mobile si el PRD lo indica.
- [ ] Los grids/columnas se adaptan correctamente en todos los breakpoints del PRD.
- [ ] No hay overflow horizontal en ningún breakpoint.
- [ ] El texto es legible sin zoom en todos los breakpoints (mínimo 16px en body).

---

## Checklist 6 — Performance

- [ ] `loading="lazy"` en todas las imágenes no críticas (above-the-fold sin lazy).
- [ ] `width` y `height` explícitos en todas las etiquetas `<img>` (evita layout shift).
- [ ] `<link rel="preload">` para fuentes críticas definidas en style-guidelines.
- [ ] `font-display: swap` en todos los `@font-face`.
- [ ] SVG de iconos críticos son inline (no peticiones HTTP externas).
- [ ] No hay imágenes de fondo CSS para contenido que el PRD marque como importante (usar `<img>`).
- [ ] CSS crítico (above-the-fold) está inline en `<head>`.
- [ ] No hay scripts bloqueantes en `<head>` sin `defer` o `async`.
- [ ] Si hay `<script>`, usa `defer` o está antes de `</body>`.

---

## Checklist 7 — Accesibilidad (WCAG AA)

- [ ] Contraste texto/fondo ≥ 4.5:1 para texto normal (verificar con tokens del style-guidelines).
- [ ] Contraste texto/fondo ≥ 3:1 para texto grande (≥18px regular o ≥14px bold).
- [ ] Todos los botones con solo icono tienen `aria-label`.
- [ ] Todos los inputs tienen `<label>` asociado o `aria-label`.
- [ ] Todos los SVGs decorativos tienen `aria-hidden="true"`.
- [ ] Los modales tienen `role="dialog"`, `aria-modal="true"` y `aria-labelledby`.
- [ ] Los elementos con `aria-live` están presentes desde el inicio del DOM.
- [ ] Focus visible en todos los elementos interactivos (no se removió `outline` sin alternativa).
- [ ] Orden de foco lógico según el flujo visual del PRD.
- [ ] Mensajes de error usan `role="alert"`.
- [ ] Estados de carga usan `aria-busy="true"` y `aria-label` descriptivo.
- [ ] Imágenes informativas tienen `alt` descriptivo; imágenes decorativas tienen `alt=""`.

---

## Checklist 8 — Estados de UI

- [ ] Estado vacío implementado para cada vista del PRD que lo requiera.
- [ ] Estado de carga (skeleton o spinner) implementado donde el PRD lo indica.
- [ ] Estado con datos implementado para todos los componentes de datos.
- [ ] Estado de error implementado con mensaje claro y acción de reintento.
- [ ] Estado sin conexión / API caída si el PRD lo especifica.
- [ ] Estado de límite (rate limit, sesión expirada) si el PRD lo menciona.

---

## Checklist 9 — README.md del proyecto

- [ ] Tabla de vistas → archivos → sección del PRD.
- [ ] Tabla de design tokens con su origen en style-guidelines.
- [ ] Tabla de breakpoints con comportamiento por vista.
- [ ] Reglas de tokens (cómo y cuándo modificar).
- [ ] Instrucciones para agregar nuevas vistas.
- [ ] Instrucciones para actualizar tokens.
- [ ] Referencia explícita a qué documentos no se debe contradecir.

---

## Checklist 10 — Fidelidad a los documentos fuente

- [ ] Cada sección del HTML traza a una sección del PRD.
- [ ] Cada valor de diseño traza a un token del style-guidelines.
- [ ] No se agregaron secciones, componentes o funcionalidades no mencionados en el PRD.
- [ ] No se usaron colores, tipografías o espaciados no definidos en style-guidelines.
- [ ] No se introdujeron patrones de navegación no especificados en el PRD.
- [ ] Si se tomó alguna decisión de implementación no cubierta por los documentos → está documentada en README.md como decisión explícita y consultada con el usuario.
