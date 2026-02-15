# Patrones de Capa de Aplicación

Plantillas para casos de uso, DTOs y repositorios.

## Tabla de Contenidos

1. [Caso de Uso Básico](#caso-de-uso-básico)
2. [Caso de Uso con Either](#caso-de-uso-con-either)
3. [DTOs de Entrada](#dtos-de-entrada)
4. [DTOs de Respuesta](#dtos-de-respuesta)
5. [Repository Interface (Port)](#repository-interface-port)
6. [Repository Implementation (Adapter)](#repository-implementation-adapter)
7. [Event Publisher](#event-publisher)

---

## Caso de Uso Básico

```typescript
// src/contexts/bookings/application/use-cases/create-booking.usecase.ts
import { UseCase } from "@/shared/application/use-case.js";
import { Uuid } from "@/shared/domain/uuid.js";
import { Booking } from "../../domain/entities/booking.entity.js";
import { BookingRepository } from "../../domain/repositories/booking.repository.js";
import { RoomRepository } from "@/rooms/domain/repositories/room.repository.js";
import { DateRange } from "../../domain/value-objects/date-range.vo.js";
import { CreateBookingDto } from "../dtos/create-booking.dto.js";
import { BookingResponseDto, BookingMapper } from "../dtos/booking.response.dto.js";
import { RoomNotAvailableError } from "../../domain/errors/booking.error.js";

export class CreateBookingUseCase implements UseCase<CreateBookingDto, BookingResponseDto> {
  constructor(
    private readonly bookingRepository: BookingRepository,
    private readonly roomRepository: RoomRepository
  ) {}

  async execute(input: CreateBookingDto): Promise<BookingResponseDto> {
    const userId = Uuid.fromString(input.userId);
    const roomId = Uuid.fromString(input.roomId);
    const dateRange = DateRange.create(
      new Date(input.checkIn),
      new Date(input.checkOut)
    );

    // Verificar disponibilidad
    const room = await this.roomRepository.findById(roomId);
    if (!room || !room.isAvailable) {
      throw new RoomNotAvailableError(roomId.value, `${input.checkIn} - ${input.checkOut}`);
    }

    // Verificar conflictos de reserva
    const hasConflict = await this.bookingRepository.existsOverlapping(roomId, dateRange);
    if (hasConflict) {
      throw new RoomNotAvailableError(roomId.value, `${input.checkIn} - ${input.checkOut}`);
    }

    // Calcular precio total
    const totalPrice = room.pricePerNight.amount * dateRange.nights;

    // Crear reserva
    const booking = Booking.create({
      userId,
      roomId,
      dateRange,
      totalPrice,
    });

    await this.bookingRepository.save(booking);

    // TODO: Publicar eventos de dominio
    // await this.eventPublisher.publish(booking.domainEvents);
    // booking.clearDomainEvents();

    return BookingMapper.toResponse(booking);
  }
}
```

---

## Caso de Uso con Either

```typescript
// src/contexts/bookings/application/use-cases/cancel-booking.usecase.ts
import { UseCaseWithResult } from "@/shared/application/use-case.js";
import { Either, left, right } from "@/shared/domain/either.js";
import { Uuid } from "@/shared/domain/uuid.js";
import { BookingRepository } from "../../domain/repositories/booking.repository.js";
import { CancelBookingDto } from "../dtos/cancel-booking.dto.js";
import { BookingResponseDto, BookingMapper } from "../dtos/booking.response.dto.js";
import { BookingNotFoundError, BookingAlreadyCancelledError } from "../../domain/errors/booking.error.js";

type CancelBookingError = BookingNotFoundError | BookingAlreadyCancelledError;

export class CancelBookingUseCase
  implements UseCaseWithResult<CancelBookingDto, CancelBookingError, BookingResponseDto>
{
  constructor(private readonly bookingRepository: BookingRepository) {}

  async execute(input: CancelBookingDto): Promise<Either<CancelBookingError, BookingResponseDto>> {
    const bookingId = Uuid.fromString(input.bookingId);

    const booking = await this.bookingRepository.findById(bookingId);
    if (!booking) {
      return left(new BookingNotFoundError(input.bookingId));
    }

    try {
      booking.cancel();
    } catch (error) {
      if (error instanceof BookingAlreadyCancelledError) {
        return left(error);
      }
      throw error;
    }

    await this.bookingRepository.save(booking);

    return right(BookingMapper.toResponse(booking));
  }
}
```

### Uso del Either en controlador

```typescript
// Ejemplo de uso en capa de presentación
const result = await cancelBookingUseCase.execute({ bookingId });

result.fold(
  (error) => {
    // Manejar error según tipo
    if (error instanceof BookingNotFoundError) {
      return res.status(404).json(error.toJSON());
    }
    return res.status(400).json(error.toJSON());
  },
  (booking) => {
    return res.status(200).json(booking);
  }
);
```

---

## DTOs de Entrada

```typescript
// src/contexts/bookings/application/dtos/create-booking.dto.ts
export interface CreateBookingDto {
  userId: string;
  roomId: string;
  checkIn: string;  // ISO date string
  checkOut: string; // ISO date string
}

// src/contexts/bookings/application/dtos/cancel-booking.dto.ts
export interface CancelBookingDto {
  bookingId: string;
}

// src/contexts/bookings/application/dtos/get-bookings-by-user.dto.ts
export interface GetBookingsByUserDto {
  userId: string;
  status?: string;
  page?: number;
  limit?: number;
}

// src/contexts/bookings/application/dtos/update-booking.dto.ts
export interface UpdateBookingDto {
  bookingId: string;
  checkIn?: string;
  checkOut?: string;
}
```

---

## DTOs de Respuesta

```typescript
// src/contexts/bookings/application/dtos/booking.response.dto.ts
import { Booking } from "../../domain/entities/booking.entity.js";

export interface BookingResponseDto {
  id: string;
  userId: string;
  roomId: string;
  checkIn: string;
  checkOut: string;
  nights: number;
  totalPrice: number;
  status: string;
  createdAt: string;
  updatedAt: string;
}

export class BookingMapper {
  static toResponse(booking: Booking): BookingResponseDto {
    return {
      id: booking.id.value,
      userId: booking.userId.value,
      roomId: booking.roomId.value,
      checkIn: booking.dateRange.startDate.toISOString(),
      checkOut: booking.dateRange.endDate.toISOString(),
      nights: booking.dateRange.nights,
      totalPrice: booking.totalPrice,
      status: booking.status,
      createdAt: booking.createdAt.toISOString(),
      updatedAt: booking.updatedAt.toISOString(),
    };
  }

  static toResponseList(bookings: Booking[]): BookingResponseDto[] {
    return bookings.map(this.toResponse);
  }
}
```

### DTO con paginación

```typescript
// src/contexts/shared/application/dtos/paginated.dto.ts
export interface PaginatedDto<T> {
  data: T[];
  meta: {
    total: number;
    page: number;
    limit: number;
    totalPages: number;
    hasNextPage: boolean;
    hasPrevPage: boolean;
  };
}

// Uso
export type PaginatedBookingsDto = PaginatedDto<BookingResponseDto>;
```

---

## Repository Interface (Port)

```typescript
// src/contexts/bookings/domain/repositories/booking.repository.ts
import { Uuid } from "@/shared/domain/uuid.js";
import { Booking } from "../entities/booking.entity.js";
import { DateRange } from "../value-objects/date-range.vo.js";
import { BookingStatus } from "../value-objects/booking-status.vo.js";

export interface FindBookingsFilters {
  userId?: Uuid;
  roomId?: Uuid;
  status?: BookingStatus;
  dateRange?: DateRange;
}

export interface PaginationOptions {
  page: number;
  limit: number;
}

export interface PaginatedResult<T> {
  data: T[];
  total: number;
}

export interface BookingRepository {
  findById(id: Uuid): Promise<Booking | null>;
  findAll(filters?: FindBookingsFilters): Promise<Booking[]>;
  findPaginated(
    filters: FindBookingsFilters,
    pagination: PaginationOptions
  ): Promise<PaginatedResult<Booking>>;
  findByUserId(userId: Uuid): Promise<Booking[]>;
  findByRoomId(roomId: Uuid): Promise<Booking[]>;
  existsOverlapping(roomId: Uuid, dateRange: DateRange): Promise<boolean>;
  save(booking: Booking): Promise<void>;
  delete(id: Uuid): Promise<void>;
}
```

### Repository genérico

```typescript
// src/contexts/shared/domain/repositories/repository.ts
import { Identifier } from "../identifier.js";
import { Entity } from "../entity.js";

export interface Repository<T extends Entity<unknown, IdType>, IdType extends Identifier<unknown>> {
  findById(id: IdType): Promise<T | null>;
  findAll(): Promise<T[]>;
  save(entity: T): Promise<void>;
  delete(id: IdType): Promise<void>;
}
```

---

## Repository Implementation (Adapter)

```typescript
// src/contexts/bookings/infrastructure/persistence/mongo-booking.repository.ts
import { Collection, Db, Filter } from "mongodb";
import { Uuid } from "@/shared/domain/uuid.js";
import { Booking } from "../../domain/entities/booking.entity.js";
import {
  BookingRepository,
  FindBookingsFilters,
  PaginationOptions,
  PaginatedResult,
} from "../../domain/repositories/booking.repository.js";
import { DateRange } from "../../domain/value-objects/date-range.vo.js";
import { BookingStatus } from "../../domain/value-objects/booking-status.vo.js";

interface BookingDocument {
  _id: string;
  userId: string;
  roomId: string;
  checkIn: Date;
  checkOut: Date;
  status: string;
  totalPrice: number;
  createdAt: Date;
  updatedAt: Date;
}

export class MongoBookingRepository implements BookingRepository {
  private collection: Collection<BookingDocument>;

  constructor(db: Db) {
    this.collection = db.collection<BookingDocument>("bookings");
  }

  async findById(id: Uuid): Promise<Booking | null> {
    const doc = await this.collection.findOne({ _id: id.value });
    return doc ? this.toDomain(doc) : null;
  }

  async findAll(filters?: FindBookingsFilters): Promise<Booking[]> {
    const query = this.buildQuery(filters);
    const docs = await this.collection.find(query).toArray();
    return docs.map((doc) => this.toDomain(doc));
  }

  async findPaginated(
    filters: FindBookingsFilters,
    pagination: PaginationOptions
  ): Promise<PaginatedResult<Booking>> {
    const query = this.buildQuery(filters);
    const skip = (pagination.page - 1) * pagination.limit;

    const [docs, total] = await Promise.all([
      this.collection.find(query).skip(skip).limit(pagination.limit).toArray(),
      this.collection.countDocuments(query),
    ]);

    return {
      data: docs.map((doc) => this.toDomain(doc)),
      total,
    };
  }

  async findByUserId(userId: Uuid): Promise<Booking[]> {
    const docs = await this.collection.find({ userId: userId.value }).toArray();
    return docs.map((doc) => this.toDomain(doc));
  }

  async findByRoomId(roomId: Uuid): Promise<Booking[]> {
    const docs = await this.collection.find({ roomId: roomId.value }).toArray();
    return docs.map((doc) => this.toDomain(doc));
  }

  async existsOverlapping(roomId: Uuid, dateRange: DateRange): Promise<boolean> {
    const count = await this.collection.countDocuments({
      roomId: roomId.value,
      status: { $ne: BookingStatus.CANCELLED },
      $or: [
        {
          checkIn: { $lt: dateRange.endDate },
          checkOut: { $gt: dateRange.startDate },
        },
      ],
    });
    return count > 0;
  }

  async save(booking: Booking): Promise<void> {
    const doc = this.toPersistence(booking);
    await this.collection.updateOne(
      { _id: doc._id },
      { $set: doc },
      { upsert: true }
    );
  }

  async delete(id: Uuid): Promise<void> {
    await this.collection.deleteOne({ _id: id.value });
  }

  private buildQuery(filters?: FindBookingsFilters): Filter<BookingDocument> {
    const query: Filter<BookingDocument> = {};

    if (filters?.userId) {
      query.userId = filters.userId.value;
    }
    if (filters?.roomId) {
      query.roomId = filters.roomId.value;
    }
    if (filters?.status) {
      query.status = filters.status;
    }

    return query;
  }

  private toDomain(doc: BookingDocument): Booking {
    return Booking.reconstitute(
      Uuid.fromString(doc._id),
      {
        userId: Uuid.fromString(doc.userId),
        roomId: Uuid.fromString(doc.roomId),
        dateRange: DateRange.create(doc.checkIn, doc.checkOut),
        status: doc.status as BookingStatus,
        totalPrice: doc.totalPrice,
        createdAt: doc.createdAt,
        updatedAt: doc.updatedAt,
      }
    );
  }

  private toPersistence(booking: Booking): BookingDocument {
    return {
      _id: booking.id.value,
      userId: booking.userId.value,
      roomId: booking.roomId.value,
      checkIn: booking.dateRange.startDate,
      checkOut: booking.dateRange.endDate,
      status: booking.status,
      totalPrice: booking.totalPrice,
      createdAt: booking.createdAt,
      updatedAt: booking.updatedAt,
    };
  }
}
```

---

## Event Publisher

```typescript
// src/contexts/shared/domain/events/event-publisher.ts
import { DomainEvent } from "../domain-event.js";

export interface EventPublisher {
  publish(events: DomainEvent[]): Promise<void>;
  publishSingle(event: DomainEvent): Promise<void>;
}

// src/contexts/shared/infrastructure/events/in-memory-event-publisher.ts
import { DomainEvent } from "@/shared/domain/domain-event.js";
import { EventPublisher } from "@/shared/domain/events/event-publisher.js";

type EventHandler = (event: DomainEvent) => Promise<void>;

export class InMemoryEventPublisher implements EventPublisher {
  private handlers: Map<string, EventHandler[]> = new Map();

  subscribe(eventName: string, handler: EventHandler): void {
    const handlers = this.handlers.get(eventName) || [];
    handlers.push(handler);
    this.handlers.set(eventName, handlers);
  }

  async publish(events: DomainEvent[]): Promise<void> {
    for (const event of events) {
      await this.publishSingle(event);
    }
  }

  async publishSingle(event: DomainEvent): Promise<void> {
    const handlers = this.handlers.get(event.eventName) || [];
    await Promise.all(handlers.map((handler) => handler(event)));
  }
}
```

---

## Index Exports por Contexto

```typescript
// src/contexts/bookings/application/index.ts
export * from "./use-cases/create-booking.usecase.js";
export * from "./use-cases/cancel-booking.usecase.js";
export * from "./dtos/create-booking.dto.js";
export * from "./dtos/cancel-booking.dto.js";
export * from "./dtos/booking.response.dto.js";

// src/contexts/bookings/domain/index.ts
export * from "./entities/booking.entity.js";
export * from "./value-objects/date-range.vo.js";
export * from "./value-objects/booking-status.vo.js";
export * from "./repositories/booking.repository.js";
export * from "./events/booking-created.event.js";
export * from "./events/booking-cancelled.event.js";
export * from "./errors/booking.error.js";
```
