# Building Blocks del Dominio

Plantillas de clases base para implementar DDD en TypeScript.

## Tabla de Contenidos

1. [Identifier (Value Object base para IDs)](#identifier)
2. [ValueObject](#valueobject)
3. [Entity](#entity)
4. [AggregateRoot](#aggregateroot)
5. [DomainEvent](#domainevent)
6. [DomainError](#domainerror)
7. [Either (Result Pattern)](#either)
8. [UseCase Interface](#usecase-interface)

---

## Identifier

```typescript
// src/contexts/shared/domain/identifier.ts
export abstract class Identifier<T> {
  constructor(private readonly _value: T) {
    this.validate(_value);
  }

  protected abstract validate(value: T): void;

  get value(): T {
    return this._value;
  }

  equals(other?: Identifier<T>): boolean {
    if (other === null || other === undefined) {
      return false;
    }
    if (!(other instanceof this.constructor)) {
      return false;
    }
    return this._value === other._value;
  }

  toString(): string {
    return String(this._value);
  }
}
```

### UUID Identifier

```typescript
// src/contexts/shared/domain/uuid.ts
import { randomUUID } from "crypto";
import { Identifier } from "./identifier.js";
import { InvalidUuidError } from "./errors/invalid-uuid.error.js";

const UUID_REGEX = /^[0-9a-f]{8}-[0-9a-f]{4}-[1-5][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i;

export class Uuid extends Identifier<string> {
  protected validate(value: string): void {
    if (!UUID_REGEX.test(value)) {
      throw new InvalidUuidError(value);
    }
  }

  static generate(): Uuid {
    return new Uuid(randomUUID());
  }

  static fromString(value: string): Uuid {
    return new Uuid(value);
  }
}
```

---

## ValueObject

```typescript
// src/contexts/shared/domain/value-object.ts
export abstract class ValueObject<T extends Record<string, unknown>> {
  protected readonly props: T;

  protected constructor(props: T) {
    this.props = Object.freeze(props);
  }

  equals(other?: ValueObject<T>): boolean {
    if (other === null || other === undefined) {
      return false;
    }
    if (other.props === undefined) {
      return false;
    }
    return JSON.stringify(this.props) === JSON.stringify(other.props);
  }
}
```

### Ejemplo: DateRange Value Object

```typescript
// src/contexts/bookings/domain/value-objects/date-range.vo.ts
import { ValueObject } from "@/shared/domain/value-object.js";
import { InvalidDateRangeError } from "../errors/booking.error.js";

interface DateRangeProps {
  startDate: Date;
  endDate: Date;
}

export class DateRange extends ValueObject<DateRangeProps> {
  private constructor(props: DateRangeProps) {
    super(props);
  }

  static create(startDate: Date, endDate: Date): DateRange {
    if (startDate >= endDate) {
      throw new InvalidDateRangeError("Start date must be before end date");
    }
    return new DateRange({ startDate, endDate });
  }

  get startDate(): Date {
    return this.props.startDate;
  }

  get endDate(): Date {
    return this.props.endDate;
  }

  get nights(): number {
    const diff = this.props.endDate.getTime() - this.props.startDate.getTime();
    return Math.ceil(diff / (1000 * 60 * 60 * 24));
  }

  overlaps(other: DateRange): boolean {
    return this.startDate < other.endDate && this.endDate > other.startDate;
  }
}
```

### Ejemplo: Money Value Object

```typescript
// src/contexts/shared/domain/value-objects/money.vo.ts
import { ValueObject } from "../value-object.js";
import { InvalidMoneyError } from "../errors/domain.error.js";

interface MoneyProps {
  amount: number;
  currency: string;
}

export class Money extends ValueObject<MoneyProps> {
  private static readonly VALID_CURRENCIES = ["USD", "EUR", "GBP", "MXN"];

  private constructor(props: MoneyProps) {
    super(props);
  }

  static create(amount: number, currency: string): Money {
    if (amount < 0) {
      throw new InvalidMoneyError("Amount cannot be negative");
    }
    if (!this.VALID_CURRENCIES.includes(currency)) {
      throw new InvalidMoneyError(`Invalid currency: ${currency}`);
    }
    return new Money({ amount, currency });
  }

  get amount(): number {
    return this.props.amount;
  }

  get currency(): string {
    return this.props.currency;
  }

  add(other: Money): Money {
    this.ensureSameCurrency(other);
    return Money.create(this.amount + other.amount, this.currency);
  }

  subtract(other: Money): Money {
    this.ensureSameCurrency(other);
    return Money.create(this.amount - other.amount, this.currency);
  }

  private ensureSameCurrency(other: Money): void {
    if (this.currency !== other.currency) {
      throw new InvalidMoneyError("Cannot operate on different currencies");
    }
  }
}
```

---

## Entity

```typescript
// src/contexts/shared/domain/entity.ts
import { Identifier } from "./identifier.js";

export abstract class Entity<T, IdType extends Identifier<unknown>> {
  protected readonly _id: IdType;
  protected props: T;

  protected constructor(id: IdType, props: T) {
    this._id = id;
    this.props = props;
  }

  get id(): IdType {
    return this._id;
  }

  equals(other?: Entity<T, IdType>): boolean {
    if (other === null || other === undefined) {
      return false;
    }
    if (!(other instanceof Entity)) {
      return false;
    }
    return this._id.equals(other._id);
  }
}
```

### Ejemplo: Room Entity

```typescript
// src/contexts/rooms/domain/entities/room.entity.ts
import { Entity } from "@/shared/domain/entity.js";
import { Uuid } from "@/shared/domain/uuid.js";
import { Money } from "@/shared/domain/value-objects/money.vo.js";
import { RoomType } from "../value-objects/room-type.vo.js";

interface RoomProps {
  number: string;
  type: RoomType;
  pricePerNight: Money;
  isAvailable: boolean;
  createdAt: Date;
  updatedAt: Date;
}

export class Room extends Entity<RoomProps, Uuid> {
  private constructor(id: Uuid, props: RoomProps) {
    super(id, props);
  }

  static create(props: Omit<RoomProps, "createdAt" | "updatedAt">, id?: Uuid): Room {
    const now = new Date();
    return new Room(id ?? Uuid.generate(), {
      ...props,
      createdAt: now,
      updatedAt: now,
    });
  }

  static reconstitute(id: Uuid, props: RoomProps): Room {
    return new Room(id, props);
  }

  get number(): string {
    return this.props.number;
  }

  get type(): RoomType {
    return this.props.type;
  }

  get pricePerNight(): Money {
    return this.props.pricePerNight;
  }

  get isAvailable(): boolean {
    return this.props.isAvailable;
  }

  markAsUnavailable(): void {
    this.props.isAvailable = false;
    this.props.updatedAt = new Date();
  }

  markAsAvailable(): void {
    this.props.isAvailable = true;
    this.props.updatedAt = new Date();
  }

  updatePrice(newPrice: Money): void {
    this.props.pricePerNight = newPrice;
    this.props.updatedAt = new Date();
  }
}
```

---

## AggregateRoot

```typescript
// src/contexts/shared/domain/aggregate-root.ts
import { Entity } from "./entity.js";
import { Identifier } from "./identifier.js";
import { DomainEvent } from "./domain-event.js";

export abstract class AggregateRoot<T, IdType extends Identifier<unknown>> extends Entity<T, IdType> {
  private _domainEvents: DomainEvent[] = [];

  get domainEvents(): ReadonlyArray<DomainEvent> {
    return [...this._domainEvents];
  }

  protected addDomainEvent(event: DomainEvent): void {
    this._domainEvents.push(event);
  }

  clearDomainEvents(): void {
    this._domainEvents = [];
  }
}
```

### Ejemplo: Booking Aggregate

```typescript
// src/contexts/bookings/domain/entities/booking.entity.ts
import { AggregateRoot } from "@/shared/domain/aggregate-root.js";
import { Uuid } from "@/shared/domain/uuid.js";
import { DateRange } from "../value-objects/date-range.vo.js";
import { BookingStatus } from "../value-objects/booking-status.vo.js";
import { BookingCreatedEvent } from "../events/booking-created.event.js";
import { BookingCancelledEvent } from "../events/booking-cancelled.event.js";
import { BookingAlreadyCancelledError } from "../errors/booking.error.js";

interface BookingProps {
  userId: Uuid;
  roomId: Uuid;
  dateRange: DateRange;
  status: BookingStatus;
  totalPrice: number;
  createdAt: Date;
  updatedAt: Date;
}

export class Booking extends AggregateRoot<BookingProps, Uuid> {
  private constructor(id: Uuid, props: BookingProps) {
    super(id, props);
  }

  static create(
    props: Omit<BookingProps, "status" | "createdAt" | "updatedAt">,
    id?: Uuid
  ): Booking {
    const bookingId = id ?? Uuid.generate();
    const now = new Date();

    const booking = new Booking(bookingId, {
      ...props,
      status: BookingStatus.PENDING,
      createdAt: now,
      updatedAt: now,
    });

    booking.addDomainEvent(
      new BookingCreatedEvent({
        bookingId: bookingId.value,
        userId: props.userId.value,
        roomId: props.roomId.value,
        checkIn: props.dateRange.startDate,
        checkOut: props.dateRange.endDate,
        occurredOn: now,
      })
    );

    return booking;
  }

  static reconstitute(id: Uuid, props: BookingProps): Booking {
    return new Booking(id, props);
  }

  get userId(): Uuid {
    return this.props.userId;
  }

  get roomId(): Uuid {
    return this.props.roomId;
  }

  get dateRange(): DateRange {
    return this.props.dateRange;
  }

  get status(): BookingStatus {
    return this.props.status;
  }

  get totalPrice(): number {
    return this.props.totalPrice;
  }

  confirm(): void {
    if (this.props.status === BookingStatus.CANCELLED) {
      throw new BookingAlreadyCancelledError(this._id.value);
    }
    this.props.status = BookingStatus.CONFIRMED;
    this.props.updatedAt = new Date();
  }

  cancel(): void {
    if (this.props.status === BookingStatus.CANCELLED) {
      throw new BookingAlreadyCancelledError(this._id.value);
    }

    this.props.status = BookingStatus.CANCELLED;
    this.props.updatedAt = new Date();

    this.addDomainEvent(
      new BookingCancelledEvent({
        bookingId: this._id.value,
        roomId: this.props.roomId.value,
        occurredOn: new Date(),
      })
    );
  }
}
```

---

## DomainEvent

```typescript
// src/contexts/shared/domain/domain-event.ts
export interface DomainEvent {
  readonly eventName: string;
  readonly occurredOn: Date;
  readonly aggregateId: string;
}

export abstract class BaseDomainEvent implements DomainEvent {
  abstract readonly eventName: string;
  readonly occurredOn: Date;
  readonly aggregateId: string;

  protected constructor(aggregateId: string, occurredOn?: Date) {
    this.aggregateId = aggregateId;
    this.occurredOn = occurredOn ?? new Date();
  }
}
```

### Ejemplo: BookingCreated Event

```typescript
// src/contexts/bookings/domain/events/booking-created.event.ts
import { BaseDomainEvent } from "@/shared/domain/domain-event.js";

interface BookingCreatedPayload {
  bookingId: string;
  userId: string;
  roomId: string;
  checkIn: Date;
  checkOut: Date;
  occurredOn?: Date;
}

export class BookingCreatedEvent extends BaseDomainEvent {
  static readonly EVENT_NAME = "booking.created";
  readonly eventName = BookingCreatedEvent.EVENT_NAME;

  readonly userId: string;
  readonly roomId: string;
  readonly checkIn: Date;
  readonly checkOut: Date;

  constructor(payload: BookingCreatedPayload) {
    super(payload.bookingId, payload.occurredOn);
    this.userId = payload.userId;
    this.roomId = payload.roomId;
    this.checkIn = payload.checkIn;
    this.checkOut = payload.checkOut;
  }
}
```

---

## DomainError

```typescript
// src/contexts/shared/domain/errors/domain.error.ts
export abstract class DomainError extends Error {
  readonly code: string;
  readonly timestamp: Date;

  protected constructor(message: string, code: string) {
    super(message);
    this.name = this.constructor.name;
    this.code = code;
    this.timestamp = new Date();
    Error.captureStackTrace(this, this.constructor);
  }

  toJSON(): Record<string, unknown> {
    return {
      name: this.name,
      message: this.message,
      code: this.code,
      timestamp: this.timestamp.toISOString(),
    };
  }
}

// Errores comunes
export class InvalidUuidError extends DomainError {
  constructor(value: string) {
    super(`Value "${value}" is not a valid UUID`, "INVALID_UUID");
  }
}

export class InvalidMoneyError extends DomainError {
  constructor(message: string) {
    super(message, "INVALID_MONEY");
  }
}

export class EntityNotFoundError extends DomainError {
  constructor(entityName: string, id: string) {
    super(`${entityName} with id "${id}" not found`, "ENTITY_NOT_FOUND");
  }
}
```

### Errores de Contexto Específico

```typescript
// src/contexts/bookings/domain/errors/booking.error.ts
import { DomainError } from "@/shared/domain/errors/domain.error.js";

export class InvalidDateRangeError extends DomainError {
  constructor(message: string) {
    super(message, "INVALID_DATE_RANGE");
  }
}

export class BookingAlreadyCancelledError extends DomainError {
  constructor(bookingId: string) {
    super(`Booking "${bookingId}" is already cancelled`, "BOOKING_ALREADY_CANCELLED");
  }
}

export class RoomNotAvailableError extends DomainError {
  constructor(roomId: string, dateRange: string) {
    super(`Room "${roomId}" is not available for ${dateRange}`, "ROOM_NOT_AVAILABLE");
  }
}

export class BookingNotFoundError extends DomainError {
  constructor(bookingId: string) {
    super(`Booking with id "${bookingId}" not found`, "BOOKING_NOT_FOUND");
  }
}
```

---

## Either

```typescript
// src/contexts/shared/domain/either.ts

export type Either<L, R> = Left<L, R> | Right<L, R>;

export class Left<L, R> {
  readonly value: L;

  constructor(value: L) {
    this.value = value;
  }

  isLeft(): this is Left<L, R> {
    return true;
  }

  isRight(): this is Right<L, R> {
    return false;
  }

  map<U>(_fn: (value: R) => U): Either<L, U> {
    return this as unknown as Left<L, U>;
  }

  flatMap<U>(_fn: (value: R) => Either<L, U>): Either<L, U> {
    return this as unknown as Left<L, U>;
  }

  getOrElse(defaultValue: R): R {
    return defaultValue;
  }

  fold<U>(onLeft: (value: L) => U, _onRight: (value: R) => U): U {
    return onLeft(this.value);
  }
}

export class Right<L, R> {
  readonly value: R;

  constructor(value: R) {
    this.value = value;
  }

  isLeft(): this is Left<L, R> {
    return false;
  }

  isRight(): this is Right<L, R> {
    return true;
  }

  map<U>(fn: (value: R) => U): Either<L, U> {
    return new Right(fn(this.value));
  }

  flatMap<U>(fn: (value: R) => Either<L, U>): Either<L, U> {
    return fn(this.value);
  }

  getOrElse(_defaultValue: R): R {
    return this.value;
  }

  fold<U>(_onLeft: (value: L) => U, onRight: (value: R) => U): U {
    return onRight(this.value);
  }
}

// Helper functions
export const left = <L, R>(value: L): Either<L, R> => new Left(value);
export const right = <L, R>(value: R): Either<L, R> => new Right(value);
```

---

## UseCase Interface

```typescript
// src/contexts/shared/application/use-case.ts
export interface UseCase<Input, Output> {
  execute(input: Input): Promise<Output>;
}

// Variante con Either para manejo de errores
export interface UseCaseWithResult<Input, Error, Success> {
  execute(input: Input): Promise<Either<Error, Success>>;
}
```

---

## Index Exports

```typescript
// src/contexts/shared/domain/index.ts
export * from "./identifier.js";
export * from "./uuid.js";
export * from "./value-object.js";
export * from "./entity.js";
export * from "./aggregate-root.js";
export * from "./domain-event.js";
export * from "./either.js";
export * from "./errors/domain.error.js";

// src/contexts/shared/application/index.ts
export * from "./use-case.js";
```
