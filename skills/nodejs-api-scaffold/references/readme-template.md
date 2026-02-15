# Plantilla README.md - Guía de Desarrollo

Esta plantilla genera un README.md completo con todas las convenciones y guías de desarrollo del proyecto.

## Instrucciones de Uso

1. Reemplazar `{{VARIABLE}}` con valores específicos del proyecto
2. Eliminar secciones no aplicables
3. Agregar bounded contexts específicos del dominio
4. Personalizar ejemplos con entidades reales del proyecto

---

## Plantilla README.md

````markdown
# {{PROJECT_NAME}}

{{PROJECT_DESCRIPTION}}

## Tabla de Contenidos

- [Requisitos](#requisitos)
- [Instalación](#instalación)
- [Scripts Disponibles](#scripts-disponibles)
- [Arquitectura](#arquitectura)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Guía de Desarrollo](#guía-de-desarrollo)
- [Convenciones de Código](#convenciones-de-código)
- [Testing](#testing)
- [Path Aliases](#path-aliases)

---

## Requisitos

- **Node.js**: {{NODE_VERSION}} o superior
- **npm**: 10.x o superior
- **Base de datos**: {{DATABASE}} ({{DATABASE_VERSION}})

## Instalación

```bash
# Clonar repositorio
git clone {{REPO_URL}}
cd {{PROJECT_NAME}}

# Instalar dependencias
npm install

# Configurar variables de entorno
cp .env.example .env

# Ejecutar en desarrollo
npm run dev
```

## Scripts Disponibles

| Script | Descripción |
|--------|-------------|
| `npm run dev` | Inicia el servidor en modo desarrollo con hot-reload |
| `npm run build` | Compila TypeScript a JavaScript |
| `npm start` | Ejecuta la versión compilada |
| `npm test` | Ejecuta todos los tests |
| `npm run test:watch` | Ejecuta tests en modo watch |
| `npm run test:coverage` | Genera reporte de cobertura |
| `npm run lint` | Ejecuta el linter (Biome) |
| `npm run format` | Formatea el código (Biome) |
| `npm run check` | Ejecuta lint + format check |

---

## Arquitectura

Este proyecto sigue **Clean Architecture** con **Domain-Driven Design (DDD)**.

### Principios Fundamentales

1. **Independencia de frameworks**: El dominio no depende de librerías externas
2. **Testeable**: La lógica de negocio se puede probar sin UI, DB o servicios externos
3. **Independencia de UI**: La interfaz puede cambiar sin afectar el dominio
4. **Independencia de BD**: Se puede cambiar MongoDB por PostgreSQL sin tocar el dominio
5. **Independencia de agentes externos**: Las reglas de negocio no conocen el mundo exterior

### Capas de la Arquitectura

```
┌─────────────────────────────────────────────────────┐
│                   INFRASTRUCTURE                     │
│  (Controllers, Repositories Impl, External APIs)     │
├─────────────────────────────────────────────────────┤
│                    APPLICATION                       │
│         (Use Cases, DTOs, Orchestration)            │
├─────────────────────────────────────────────────────┤
│                      DOMAIN                          │
│  (Entities, Value Objects, Repository Interfaces)    │
└─────────────────────────────────────────────────────┘
```

### Regla de Dependencias

```
infrastructure → application → domain
     ↓               ↓            ↓
   puede          puede        NO puede
  importar       importar      importar
     de             de         de nadie
  application     domain        más
```

**CRÍTICO**: El dominio es puro y no importa de capas superiores.

---

## Estructura del Proyecto

```
{{PROJECT_NAME}}/
├── src/
│   └── contexts/
│       ├── shared/                    # Building blocks compartidos
│       │   ├── domain/
│       │   │   ├── aggregate-root.ts
│       │   │   ├── entity.ts
│       │   │   ├── value-object.ts
│       │   │   ├── domain-event.ts
│       │   │   ├── either.ts
│       │   │   └── errors/
│       │   │       └── domain.error.ts
│       │   └── application/
│       │       └── use-case.ts
│       │
{{#each BOUNDED_CONTEXTS}}
│       ├── {{this.name}}/
│       │   ├── domain/
│       │   │   ├── entities/
│       │   │   ├── value-objects/
│       │   │   ├── events/
│       │   │   ├── errors/
│       │   │   └── repositories/      # Interfaces (Ports)
│       │   ├── application/
│       │   │   ├── use-cases/
│       │   │   └── dtos/
│       │   └── infrastructure/        # Implementations (Adapters)
│       │       └── persistence/
{{/each}}
│
├── tests/
│   └── contexts/                      # Estructura espejo de src/
│
├── package.json
├── tsconfig.json
├── biome.json
├── vitest.config.ts
└── README.md
```

### Bounded Contexts

| Contexto | Responsabilidad |
|----------|-----------------|
{{#each BOUNDED_CONTEXTS}}
| `{{this.name}}` | {{this.description}} |
{{/each}}

---

## Guía de Desarrollo

### Crear una Nueva Entidad

1. **Crear el archivo** en `src/contexts/{context}/domain/entities/{name}.entity.ts`
2. **Extender** de `Entity` o `AggregateRoot` según corresponda
3. **Definir propiedades** como interfaz privada
4. **Implementar** método factory `create()` y `reconstitute()`

```typescript
// src/contexts/{{CONTEXT}}/domain/entities/{{entity}}.entity.ts
import { Entity } from "@/shared/domain/entity.js";
import { Uuid } from "@/shared/domain/uuid.js";

interface {{Entity}}Props {
  name: string;
  // ... más propiedades
  createdAt: Date;
  updatedAt: Date;
}

export class {{Entity}} extends Entity<{{Entity}}Props, Uuid> {
  private constructor(id: Uuid, props: {{Entity}}Props) {
    super(id, props);
  }

  // Factory method para crear nuevas instancias
  static create(props: Omit<{{Entity}}Props, "createdAt" | "updatedAt">, id?: Uuid): {{Entity}} {
    const now = new Date();
    return new {{Entity}}(id ?? Uuid.generate(), {
      ...props,
      createdAt: now,
      updatedAt: now,
    });
  }

  // Reconstituir desde persistencia
  static reconstitute(id: Uuid, props: {{Entity}}Props): {{Entity}} {
    return new {{Entity}}(id, props);
  }

  // Getters
  get name(): string {
    return this.props.name;
  }

  // Métodos de dominio
  // ...
}
```

### Crear un Value Object

1. **Crear el archivo** en `src/contexts/{context}/domain/value-objects/{name}.vo.ts`
2. **Extender** de `ValueObject<Props>`
3. **Validar** en el método factory `create()`

```typescript
// src/contexts/{{CONTEXT}}/domain/value-objects/{{value-object}}.vo.ts
import { ValueObject } from "@/shared/domain/value-object.js";
import { Invalid{{ValueObject}}Error } from "../errors/{{context}}.error.js";

interface {{ValueObject}}Props {
  value: string;
}

export class {{ValueObject}} extends ValueObject<{{ValueObject}}Props> {
  private constructor(props: {{ValueObject}}Props) {
    super(props);
  }

  static create(value: string): {{ValueObject}} {
    // Validaciones
    if (!value || value.trim().length === 0) {
      throw new Invalid{{ValueObject}}Error("Value cannot be empty");
    }
    return new {{ValueObject}}({ value });
  }

  get value(): string {
    return this.props.value;
  }
}
```

### Crear un Caso de Uso

1. **Crear el archivo** en `src/contexts/{context}/application/use-cases/{action}-{entity}.usecase.ts`
2. **Implementar** interfaz `UseCase<Input, Output>`
3. **Inyectar** repositorios y servicios por constructor
4. **Crear DTOs** de entrada y salida

```typescript
// src/contexts/{{CONTEXT}}/application/use-cases/create-{{entity}}.usecase.ts
import { UseCase } from "@/shared/application/use-case.js";
import { {{Entity}} } from "../../domain/entities/{{entity}}.entity.js";
import { {{Entity}}Repository } from "../../domain/repositories/{{entity}}.repository.js";
import { Create{{Entity}}Dto } from "../dtos/create-{{entity}}.dto.js";
import { {{Entity}}ResponseDto, {{Entity}}Mapper } from "../dtos/{{entity}}.response.dto.js";

export class Create{{Entity}}UseCase implements UseCase<Create{{Entity}}Dto, {{Entity}}ResponseDto> {
  constructor(private readonly repository: {{Entity}}Repository) {}

  async execute(input: Create{{Entity}}Dto): Promise<{{Entity}}ResponseDto> {
    // 1. Validar/transformar input
    // 2. Crear entidad
    const entity = {{Entity}}.create({
      // ... map from DTO
    });

    // 3. Persistir
    await this.repository.save(entity);

    // 4. Retornar DTO de respuesta
    return {{Entity}}Mapper.toResponse(entity);
  }
}
```

### Crear un Repositorio (Port)

```typescript
// src/contexts/{{CONTEXT}}/domain/repositories/{{entity}}.repository.ts
import { Uuid } from "@/shared/domain/uuid.js";
import { {{Entity}} } from "../entities/{{entity}}.entity.js";

export interface {{Entity}}Repository {
  findById(id: Uuid): Promise<{{Entity}} | null>;
  findAll(): Promise<{{Entity}}[]>;
  save(entity: {{Entity}}): Promise<void>;
  delete(id: Uuid): Promise<void>;
}
```

### Crear un Error de Dominio

```typescript
// src/contexts/{{CONTEXT}}/domain/errors/{{context}}.error.ts
import { DomainError } from "@/shared/domain/errors/domain.error.js";

export class {{Entity}}NotFoundError extends DomainError {
  constructor(id: string) {
    super(`{{Entity}} with id "${id}" not found`, "{{ENTITY}}_NOT_FOUND");
  }
}

export class Invalid{{ValueObject}}Error extends DomainError {
  constructor(message: string) {
    super(message, "INVALID_{{VALUE_OBJECT}}");
  }
}
```

---

## Convenciones de Código

### Nomenclatura de Archivos

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
| Test | `{name}.test.ts` | `booking.entity.test.ts` |
| Fixture | `{entity}.fixture.ts` | `booking.fixture.ts` |

### Nomenclatura de Código

```typescript
// Archivos y carpetas: kebab-case
create-booking.usecase.ts

// Clases y tipos: PascalCase
class CreateBookingUseCase {}
interface BookingRepository {}
type BookingStatus = "PENDING" | "CONFIRMED";

// Funciones y variables: camelCase
function createBooking() {}
const bookingRepository = {};

// Constantes: UPPER_SNAKE_CASE
const MAX_BOOKING_DAYS = 30;
const BOOKING_STATUS = { PENDING: "PENDING" } as const;

// Enums: PascalCase con valores UPPER_SNAKE_CASE
enum BookingStatus {
  PENDING = "PENDING",
  CONFIRMED = "CONFIRMED",
}
```

### TypeScript

- **Strict mode**: Siempre habilitado (`strict: true`)
- **No `any`**: Usar `unknown` + type guards
- **Readonly**: Preferir propiedades inmutables
- **Interfaces vs Types**: Interfaces para contratos públicos, types para uniones/utilidades

```typescript
// ❌ Evitar
const data: any = fetchData();

// ✅ Preferir
const data: unknown = fetchData();
if (isValidData(data)) {
  // TypeScript ahora conoce el tipo
}
```

### Imports

- Usar **path aliases** sobre rutas relativas profundas
- Organizar imports (automático con Biome)
- Incluir extensión `.js` en imports (requerido por ESM)

```typescript
// ❌ Evitar
import { Booking } from "../../../domain/entities/booking.entity";

// ✅ Preferir
import { Booking } from "@/bookings/domain/entities/booking.entity.js";
```

---

## Testing

### Enfoque: TDD/BDD con Given-When-Then

```typescript
describe("Booking Entity", () => {
  describe("create", () => {
    it("should create a booking with PENDING status", () => {
      // Given
      const props = BookingFixture.validCreateProps();

      // When
      const booking = Booking.create(props);

      // Then
      expect(booking.status).toBe(BookingStatus.PENDING);
    });
  });
});
```

### Estructura de Tests

```
tests/
└── contexts/
    └── {bounded-context}/
        ├── domain/
        │   ├── entities/
        │   │   └── {entity}.entity.test.ts
        │   └── value-objects/
        │       └── {vo}.vo.test.ts
        ├── application/
        │   └── use-cases/
        │       └── {action}-{entity}.usecase.test.ts
        ├── fixtures/
        │   └── {entity}.fixture.ts
        └── mocks/
            └── {entity}.repository.mock.ts
```

### Fixtures (Object Mothers)

```typescript
// tests/contexts/bookings/fixtures/booking.fixture.ts
export class BookingFixture {
  static pending(): Booking {
    return Booking.create(this.validCreateProps());
  }

  static confirmed(): Booking {
    const booking = this.pending();
    booking.confirm();
    return booking;
  }

  static validCreateProps() {
    return {
      userId: Uuid.fromString("550e8400-e29b-41d4-a716-446655440001"),
      roomId: Uuid.fromString("550e8400-e29b-41d4-a716-446655440002"),
      dateRange: DateRangeFixture.nextWeek(),
      totalPrice: 400,
    };
  }
}
```

### Mocks de Repositorios

```typescript
// tests/contexts/bookings/mocks/booking.repository.mock.ts
export class MockBookingRepository implements BookingRepository {
  private result: Booking | null = null;
  saveCalled = false;

  setFindByIdResult(booking: Booking | null): void {
    this.result = booking;
  }

  async findById(_id: Uuid): Promise<Booking | null> {
    return this.result;
  }

  async save(booking: Booking): Promise<void> {
    this.saveCalled = true;
  }
  
  // ... resto de métodos
}
```

### Cobertura

Mantener cobertura **> 80%** en código de dominio y aplicación.

```bash
npm run test:coverage
```

---

## Path Aliases

### Configuración

Los aliases están configurados en `tsconfig.json` y `vitest.config.ts`:

| Alias | Ruta |
|-------|------|
{{#each PATH_ALIASES}}
| `{{this.alias}}` | `{{this.path}}` |
{{/each}}

### Uso

```typescript
// Importar desde shared
import { Entity } from "@/shared/domain/entity.js";
import { UseCase } from "@/shared/application/use-case.js";

// Importar desde contexto específico
import { Booking } from "@/bookings/domain/entities/booking.entity.js";
import { CreateBookingUseCase } from "@/bookings/application/use-cases/create-booking.usecase.js";

// Importar fixtures en tests
import { BookingFixture } from "@/test/contexts/bookings/fixtures/booking.fixture.js";
```

---

## Git Workflow

### Branches

- `main` - Producción
- `feat/{description}` - Nueva funcionalidad
- `fix/{description}` - Corrección de bugs
- `refactor/{description}` - Refactorización
- `test/{description}` - Agregar/modificar tests
- `docs/{description}` - Documentación

### Commits (Conventional Commits)

```
type(scope): description

feat(bookings): add create booking use case
fix(rooms): handle null price in room entity
refactor(shared): extract base entity class
test(bookings): add unit tests for date range VO
docs(readme): update development guide
```

---

## Licencia

{{LICENSE}}
````

---

## Ejemplo Completo Generado

Para un proyecto de reservas hoteleras, el README generado se vería así (fragmento):

```markdown
# Hotel Booking Platform

Plataforma de reservas hoteleras con Clean Architecture y DDD.

## Bounded Contexts

| Contexto | Responsabilidad |
|----------|-----------------|
| `shared` | Building blocks compartidos (Entity, VO, Either) |
| `auth` | Autenticación y tokens |
| `users` | Perfiles y roles de usuario |
| `rooms` | Inventario y estado de habitaciones |
| `bookings` | Core del negocio: creación y gestión de reservas |
| `payments` | Pagos y transacciones |

## Path Aliases

| Alias | Ruta |
|-------|------|
| `@/shared/*` | `src/contexts/shared/*` |
| `@/auth/*` | `src/contexts/auth/*` |
| `@/users/*` | `src/contexts/users/*` |
| `@/rooms/*` | `src/contexts/rooms/*` |
| `@/bookings/*` | `src/contexts/bookings/*` |
| `@/payments/*` | `src/contexts/payments/*` |
| `@/test/*` | `tests/*` |
```
