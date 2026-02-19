## Ejemplo resumido: Workout Tracker (flujo de pantallas)

### Supuestos
- **Usuarios**: persona individual (sin equipo) con login.
- **Dispositivos**: mobile-first, optimizado también para desktop.

### IA + rutas (propuesta)
- `/login`
- `/app/dashboard`
- `/app/workouts` (lista)
- `/app/workouts/new` (crear plan / workout)
- `/app/workouts/:id` (detalle)
- `/app/schedule` (calendario / agenda)
- `/app/reports` (progreso + históricos)
- `/app/settings`

### Vistas mínimas (con estados)
- **Dashboard**: KPIs (volumen semanal, PRs, adherencia), “próximo entrenamiento”, acceso rápido.
- **Workouts (lista)**: filtros (activo/pendiente/pasado), orden por fecha/hora, empty state.
- **Workout (detalle)**: lista de ejercicios (sets/reps/weight), notas/comentarios, CTA “Marcar completado”.
- **Crear/Editar workout**: formulario dinámico para ejercicios + validación.
- **Schedule**: vista calendario + lista diaria, reprogramar.
- **Reports**: gráficos/tabla de progreso, export (si aplica).

### Ensamble con Flowbite Blocks (categorías)
- App shell: sidebar + topbar + header
- Dashboard: stats/cards + listas
- CRUD: tables + forms + modals
- Schedule: calendar + list
- Reports: charts (si no hay snippet, recrear con Tailwind y placeholders)
- Feedback: toast/alerts + confirm modal
- Estados: empty state + skeleton loader

### Componentes reutilizables sugeridos
- `AppShell`, `PageHeader`, `KpiCard`, `WorkoutCard`, `WorkoutTable`, `ExerciseRow`, `WorkoutForm`, `ConfirmModal`, `Toast`

