## Mapa práctico de Flowbite Blocks → vistas de app

Usa Flowbite Blocks como “secciones” por página. Objetivo: consistencia (mismo sistema) más que mezclar 10 estilos distintos.

### 1) App Shell (siempre)
- **Topbar / Navbar**: marca, buscador, notificaciones, perfil.
- **Sidebar**: navegación primaria, agrupada por dominios.
- **Page header**: título + breadcrumb + acciones.

### 2) Autenticación
- Login
- Registro (si aplica)
- Recuperación de contraseña (si aplica)

### 3) CRUD (dominio principal)
- Lista: table o cards + filtros + orden + paginación
- Detalle: resumen + secciones + historial
- Crear/Editar: forms + validación + ayuda contextual

### 4) Planificación / Agenda (si aplica)
- Calendario mensual / semanal
- Lista diaria
- Reprogramar (modal o drawer)

### 5) Reportes / Analítica (si aplica)
- KPIs (stats)
- Gráficos (si no hay snippet accesible, usa contenedores + placeholders y documenta la futura librería)
- Tablas de histórico

### 6) Estados y feedback (siempre)
- Empty state (por lista y por reportes)
- Skeleton loaders (por páginas y cards)
- Error state (con retry)
- Toast / Alert
- Confirm modal (acciones destructivas)

### Notas importantes
- **Accesibilidad**: navegación por teclado, foco visible, labels asociados, `aria-live` para toasts si aplica.
- **Premium**: si no tienes acceso al snippet exacto, recrea el patrón visual con Tailwind sin copiar contenido no disponible.

