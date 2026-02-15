# Arquitectura por defecto: Astro + Tailwind + TypeScript

Referencia condensada para el scaffold cuando el usuario no proporciona un documento de arquitectura. Interpretar de forma literal.

## Principios

- Respuestas técnicas concisas con ejemplos Astro correctos.
- Aprovechar hidratación parcial y soporte multi-framework de Astro.
- Priorizar generación estática y mínimo JavaScript.
- Nombres descriptivos y convenciones de nombres de Astro.
- Organizar con file-based routing.

## Estructura de proyecto

```
src/
  components/
  layouts/
  pages/
  styles/
public/
astro.config.mjs
```

## Componentes

- Crear componentes en archivos `.astro`.
- Usar componentes de otros frameworks (React, Vue, Svelte) cuando sea necesario.
- Composición y reutilización; pasar datos con props de Astro.
- Usar `<Markdown />` cuando sea apropiado.

## Routing y páginas

- Routing basado en archivos en `src/pages/`.
- Rutas dinámicas con `[...slug].astro` y `getStaticPaths()`.
- Página `404.astro` para rutas no encontradas.

## Contenido

- Markdown (`.md`) o MDX (`.mdx`) para páginas con mucho contenido.
- Frontmatter en Markdown; content collections para organizar contenido.

## Estilos

- Estilos con alcance con `<style>` en `.astro`.
- Estilos globales cuando haga falta, importados en layouts.
- Sass/Less solo si se requiere.
- Diseño responsive con custom properties y media queries.
- **Tailwind**: integrar con `@astrojs/tailwind`.
- Usar clases de utilidad Tailwind en componentes.
- Responsive con `sm:`, `md:`, `lg:`, etc.
- Paleta y escala de espaciado de Tailwind para consistencia.
- Extensiones de tema en `tailwind.config.cjs` cuando haga falta.
- **Nunca usar la directiva @apply.**

## Rendimiento e hidratación

- Minimizar JavaScript en cliente; priorizar generación estática.
- Directivas `client:*` con criterio:
  - `client:load` para interactividad inmediata
  - `client:idle` para interactividad no crítica
  - `client:visible` para hidratar al ser visibles
- Lazy loading de imágenes y assets.
- Optimización de assets de Astro.

## Datos

- `Astro.props` para pasar datos a componentes.
- `getStaticPaths()` para datos en build.
- `Astro.glob()` para archivos locales.
- Manejo de errores en operaciones de datos.

## SEO y meta

- Usar `<head>` para meta.
- URLs canónicas.
- Patrón de componente `<SEO>` reutilizable.

## Integraciones

- Integraciones oficiales de Astro (p. ej. `@astrojs/image`) en `astro.config.mjs`.
- Preferir integraciones oficiales.

## Build y despliegue

- Optimizar con el comando build de Astro.
- Variables de entorno por entorno.
- Hosting estático compatible (Netlify, Vercel, etc.).
- CI/CD para builds y despliegues.

## Testing

- Tests unitarios para utilidades y helpers.
- E2E (p. ej. Cypress) para el sitio construido.
- Regresión visual si aplica.

## Accesibilidad

- HTML semántico en componentes Astro.
- Atributos ARIA cuando haga falta.
- Navegación por teclado en elementos interactivos.

## Convenciones

1. Guía de estilo de Astro para formato.
2. TypeScript para tipos y DX.
3. Manejo de errores y logging.
4. RSS de Astro para sitios con mucho contenido.
5. Componente Image de Astro para imágenes optimizadas.

## Métricas de rendimiento

- Priorizar Core Web Vitals (LCP, FID, CLS).
- Lighthouse y WebPageTest para auditorías.
- Presupuestos y monitoreo de rendimiento.

## Documentación

Consultar la documentación oficial de Astro para componentes, routing e integraciones.
