---
name: nodejs-api-scaffold
description: |
  Genera esqueletos de proyectos API REST con Node.js/TypeScript siguiendo Clean Architecture y DDD.
  Usar cuando el usuario solicite:
  (1) Crear/generar una API REST desde cero basada en documento de arquitectura
  (2) Scaffolding de bounded contexts, casos de uso, entidades y repositorios
  (3) Estructura inicial de proyecto con Clean Architecture (domain/application/infrastructure)
  (4) Generar archivos base (entities, value objects, use cases, DTOs) respetando convenciones DDD
  (5) Configurar proyecto Node.js con TypeScript estricto, ESM, path aliases y testing
  Soporta Node.js 20+, TypeScript strict, Vitest, Biome, y patrones enterprise (Ports & Adapters, Repository, Domain Events).
---

# Node.js API REST Scaffold Generator

Skill para generar esqueletos de proyectos API REST con Node.js y TypeScript siguiendo Clean Architecture y Domain-Driven Design.

## Tabla de Contenidos

1. [Flujo de Trabajo Principal](#flujo-de-trabajo-principal)
2. [Análisis del Documento de Arquitectura](#análisis-del-documento-de-arquitectura)
3. [Generación de Estructura](#generación-de-estructura)
4. [Patrones de Código](#patrones-de-código)
5. [Building Blocks del Dominio](#building-blocks-del-dominio)
6. [Validación Final](#validación-final)

---

## Prerrequisito Obligatorio

> **⚠️ IMPORTANTE**: Antes de iniciar cualquier generación de scaffold, **SIEMPRE preguntar al usuario el nombre del proyecto**.
>
> El nombre del proyecto es obligatorio y se usará para:
> - Nombre en `package.json`
> - Carpeta raíz del proyecto
> - Referencias en `README.md`
> - Configuraciones generales
>
> **Pregunta sugerida**: "¿Cuál es el nombre del proyecto? (usar kebab-case, ej: `hotel-booking-api`)"

---

## Flujo de Trabajo Principal

```
0. PREGUNTAR nombre del proyecto al usuario (OBLIGATORIO)
1. ANALIZAR documento de arquitectura → extraer bounded contexts y requisitos
2. PLANIFICAR estructura de carpetas por contexto (vertical slicing)
3. GENERAR configuración base (package.json, tsconfig.json, biome.json)
4. CREAR bounded contexts con capas domain/application/infrastructure
5. IMPLEMENTAR building blocks (entities, value objects, repositories)
6. GENERAR casos de uso con DTOs de entrada/salida
7. CONFIGURAR path aliases y estructura de tests
8. GENERAR README.md con guía completa de desarrollo
9. VALIDAR coherencia del scaffold (compila, lint pasa)
```

---

## Análisis del Documento de Arquitectura

### Extraer del Documento

| Categoría | Qué buscar | Impacto en scaffold |
|-----------|------------|---------------------|
| **Tech Stack** | Versión Node, TypeScript, DB | `package.json`, `tsconfig.json` |
| **Bounded Contexts** | Dominios identificados | `src/contexts/` estructura |
| **Entidades** | Agregados, entidades | `domain/entities/` |
| **Value Objects** | Objetos de valor | `domain/value-objects/` |
| **Casos de Uso** | Operaciones de negocio | `application/use-cases/` |
| **Repositorios** | Contratos de persistencia | `domain/repositories/` (ports) |
| **Eventos de Dominio** | Eventos entre contextos | `domain/events/` |
| **Integraciones** | APIs externas, servicios | `infrastructure/adapters/` |
| **Errores** | Errores de dominio tipados | `domain/errors/` |

### Checklist de Requisitos

```markdown
[ ] Versión Node.js especificada (20+)
[ ] TypeScript strict mode requerido
[ ] Bounded contexts identificados
[ ] Entidades y agregados definidos
[ ] Value objects necesarios
[ ] Casos de uso listados por contexto
[ ] Repositorios/ports requeridos
[ ] Eventos de dominio definidos
[ ] Errores de dominio tipados
[ ] Path aliases configurados
[ ] Framework de testing (Vitest)
[ ] Linter/formatter (Biome)
```

---

## Generación de Estructura

### Estructura Base Clean Architecture + DDD

```
project-root/
├── src/
│   └── contexts/
│       ├── shared/                          # Building blocks compartidos
│       │   ├── domain/
│       │   │   ├── aggregate-root.ts        # Base class
│       │   │   ├── entity.ts                # Base class
│       │   │   ├── value-object.ts          # Base class
│       │   │   ├── domain-event.ts          # Base interface
│       │   │   ├── either.ts                # Result pattern
│       │   │   └── errors/
│       │   │       └── domain.error.ts      # Base error class
│       │   └── application/
│       │       └── use-case.ts              # Base interface
│       │
│       └── {bounded-context}/               # Por cada contexto
│           ├── domain/
│           │   ├── entities/
│           │   │   └── {entity}.entity.ts
│           │   ├── value-objects/
│           │   │   └── {vo}.vo.ts
│           │   ├── events/
│           │   │   └── {event}.event.ts
│           │   ├── errors/
│           │   │   └── {context}.error.ts
│           │   └── repositories/
│           │       └── {entity}.repository.ts   # Port/interface
│           │
│           ├── application/
│           │   ├── use-cases/
│           │   │   └── {action}-{entity}.usecase.ts
│           │   └── dtos/
│           │       ├── {action}-{entity}.dto.ts
│           │       └── {entity}.response.dto.ts
│           │
│           └── infrastructure/              # Cuando aplique
│               ├── persistence/
│               │   └── mongo-{entity}.repository.ts
│               └── adapters/
│                   └── {external}.adapter.ts
│
├── tests/
│   └── contexts/
│       └── {bounded-context}/
│           ├── domain/
│           ├── application/
│           └── fixtures/
│               └── {entity}.fixture.ts
│
├── package.json
├── tsconfig.json
├── biome.json
├── vitest.config.ts
└── README.md                            # Guía de desarrollo del proyecto
```

### Reglas de Nomenclatura

| Tipo | Patrón | Ejemplo |
|------|--------|---------|
| Entidad | `{name}.entity.ts` | `booking.entity.ts` |
| Value Object | `{name}.vo.ts` | `date-range.vo.ts` |
| Caso de Uso | `{action}-{entity}.usecase.ts` | `create-booking.usecase.ts` |
| DTO entrada | `{action}-{entity}.dto.ts` | `create-booking.dto.ts` |
| DTO salida | `{entity}.response.dto.ts` | `booking.response.dto.ts` |
| Repositorio (port) | `{entity}.repository.ts` | `booking.repository.ts` |
| Repositorio (impl) | `mongo-{entity}.repository.ts` | `mongo-booking.repository.ts` |
| Error | `{context}.error.ts` | `booking.error.ts` |
| Evento | `{entity}-{action}.event.ts` | `booking-created.event.ts` |
| Fixture | `{entity}.fixture.ts` | `booking.fixture.ts` |

### Convenciones de Código

```typescript
// Archivos y carpetas: kebab-case
create-booking.usecase.ts

// Clases y tipos: PascalCase
class CreateBookingUseCase {}
interface BookingRepository {}

// Funciones y variables: camelCase
function createBooking() {}
const bookingRepository = {};

// Constantes: UPPER_SNAKE_CASE
const MAX_BOOKING_DAYS = 30;
```

---

## Patrones de Código

### tsconfig.json con Path Aliases

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "baseUrl": ".",
    "paths": {
      "@/shared/*": ["src/contexts/shared/*"],
      "@/{{context}}/*": ["src/contexts/{{context}}/*"],
      "@/test/*": ["tests/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### package.json Base

```json
{
  "name": "{{project-name}}",
  "version": "0.1.0",
  "type": "module",
  "scripts": {
    "build": "tsc && tsc-alias",
    "dev": "tsx watch src/main.ts",
    "start": "node dist/main.js",
    "test": "vitest",
    "test:watch": "vitest --watch",
    "test:coverage": "vitest --coverage",
    "lint": "biome lint ./src ./tests",
    "format": "biome format --write ./src ./tests",
    "check": "biome check ./src ./tests"
  },
  "devDependencies": {
    "@biomejs/biome": "^1.9.0",
    "@types/node": "^22.0.0",
    "tsc-alias": "^1.8.0",
    "tsx": "^4.19.0",
    "typescript": "^5.6.0",
    "vitest": "^2.1.0"
  }
}
```

### biome.json

```json
{
  "$schema": "https://biomejs.dev/schemas/1.9.0/schema.json",
  "organizeImports": { "enabled": true },
  "formatter": {
    "enabled": true,
    "indentStyle": "space",
    "indentWidth": 2,
    "lineWidth": 120
  },
  "linter": {
    "enabled": true,
    "rules": { "recommended": true }
  },
  "javascript": {
    "formatter": { "quoteStyle": "double" }
  }
}
```

### vitest.config.ts

```typescript
import { defineConfig } from "vitest/config";
import { resolve } from "path";

export default defineConfig({
  test: {
    globals: true,
    environment: "node",
    include: ["tests/**/*.{test,spec}.ts"],
    coverage: {
      provider: "v8",
      reporter: ["text", "lcov"],
      include: ["src/**/*.ts"],
      exclude: ["src/**/*.dto.ts", "src/**/index.ts"]
    }
  },
  resolve: {
    alias: {
      "@/shared": resolve(__dirname, "src/contexts/shared"),
      "@/test": resolve(__dirname, "tests")
      // Agregar alias por cada bounded context
    }
  }
});
```

---

## Building Blocks del Dominio

Para plantillas completas de building blocks (Entity, ValueObject, AggregateRoot, etc.), ver:
- [references/building-blocks.md](references/building-blocks.md) - Clases base del dominio

Para patrones de casos de uso, repositorios y DTOs, ver:
- [references/application-patterns.md](references/application-patterns.md) - Patrones de capa aplicación

Para plantillas de tests con Vitest, ver:
- [references/testing-patterns.md](references/testing-patterns.md) - Patrones de testing TDD/BDD

Para generar el README.md del proyecto con guía de desarrollo, ver:
- [references/readme-template.md](references/readme-template.md) - Plantilla README con convenciones y guías

---

## Validación Final

### Checklist de Scaffold Completo

```markdown
## Configuración
[ ] package.json con type: "module" y scripts correctos
[ ] tsconfig.json con strict: true, ESM y path aliases
[ ] biome.json configurado
[ ] vitest.config.ts con aliases resueltos
[ ] README.md con guía completa de desarrollo

## Estructura por Bounded Context
[ ] Carpeta domain/ con entities, value-objects, repositories, errors, events
[ ] Carpeta application/ con use-cases y dtos
[ ] Carpeta infrastructure/ cuando hay implementaciones

## Building Blocks Shared
[ ] Entity base class
[ ] ValueObject base class  
[ ] AggregateRoot base class
[ ] DomainEvent interface
[ ] Either/Result pattern
[ ] DomainError base class
[ ] UseCase interface

## Código
[ ] Imports usan path aliases (@/context/*)
[ ] Sin uso de 'any' - usar unknown + type guards
[ ] Entidades extienden Entity o AggregateRoot
[ ] Value Objects extienden ValueObject
[ ] Repositorios son interfaces (ports) en domain/
[ ] Casos de uso implementan UseCase interface
[ ] DTOs son tipos/interfaces planos
[ ] Errores extienden DomainError

## Tests
[ ] Estructura espejo en tests/contexts/
[ ] Fixtures por entidad
[ ] Naming: *.test.ts o *.spec.ts

## Regla de Dependencias
[ ] domain/ NO importa de application/ ni infrastructure/
[ ] application/ puede importar de domain/
[ ] infrastructure/ puede importar de domain/ y application/
```

### Comandos de Verificación

```bash
# Verificar que compila
npm run build

# Verificar linting
npm run lint

# Ejecutar tests
npm test

# Verificar tipos
npx tsc --noEmit
```

---

## Instrucciones de Ejecución

1. **Leer completamente** el documento de arquitectura provisto
2. **Identificar** bounded contexts y sus elementos (entidades, VOs, casos de uso)
3. **Crear estructura** base con shared/ y contextos identificados
4. **Generar archivos** de configuración (package.json, tsconfig, biome, vitest)
5. **Implementar** building blocks en shared/domain/
6. **Crear** estructura de cada bounded context con sus capas
7. **Generar** entidades y value objects del dominio
8. **Definir** interfaces de repositorios (ports)
9. **Crear** casos de uso con DTOs de entrada/salida (stubs)
10. **Generar** estructura de tests con fixtures
11. **Generar README.md** usando la plantilla de [references/readme-template.md](references/readme-template.md)
12. **Validar** que compila y lint pasa sin errores

### Notas Importantes

- **NO generar lógica de negocio completa** - solo stubs con `// TODO: implement`
- **SÍ generar imports** correctos con path aliases
- **SÍ incluir** comentarios TODO donde se requiera implementación
- **Respetar** regla de dependencias (dominio puro, sin imports de infra)
- **Mantener** consistencia con convenciones del proyecto existente
- **Exportar** tipos públicos desde archivos `index.ts` por carpeta
