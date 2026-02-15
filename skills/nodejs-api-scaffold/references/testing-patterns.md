# Patrones de Testing TDD/BDD

Plantillas para tests unitarios y de integración con Vitest.

## Tabla de Contenidos

1. [Estructura de Tests](#estructura-de-tests)
2. [Tests de Entidades](#tests-de-entidades)
3. [Tests de Value Objects](#tests-de-value-objects)
4. [Tests de Casos de Uso](#tests-de-casos-de-uso)
5. [Fixtures y Object Mothers](#fixtures-y-object-mothers)
6. [Mocks de Repositorios](#mocks-de-repositorios)
7. [Tests de Integración](#tests-de-integración)

---

## Estructura de Tests

```
tests/
├── contexts/
│   ├── shared/
│   │   └── domain/
│   │       ├── uuid.test.ts
│   │       └── either.test.ts
│   │
│   └── bookings/
│       ├── domain/
│       │   ├── entities/
│       │   │   └── booking.entity.test.ts
│       │   └── value-objects/
│       │       └── date-range.vo.test.ts
│       ├── application/
│       │   └── use-cases/
│       │       ├── create-booking.usecase.test.ts
│       │       └── cancel-booking.usecase.test.ts
│       ├── fixtures/
│       │   ├── booking.fixture.ts
│       │   └── date-range.fixture.ts
│       └── mocks/
│           └── booking.repository.mock.ts
│
└── setup.ts
```

---

## Tests de Entidades

```typescript
// tests/contexts/bookings/domain/entities/booking.entity.test.ts
import { describe, it, expect, beforeEach } from "vitest";
import { Booking } from "@/bookings/domain/entities/booking.entity.js";
import { BookingStatus } from "@/bookings/domain/value-objects/booking-status.vo.js";
import { BookingAlreadyCancelledError } from "@/bookings/domain/errors/booking.error.js";
import { BookingCreatedEvent } from "@/bookings/domain/events/booking-created.event.js";
import { BookingFixture } from "../../fixtures/booking.fixture.js";

describe("Booking Entity", () => {
  describe("create", () => {
    it("should create a booking with PENDING status", () => {
      // Given
      const props = BookingFixture.validCreateProps();

      // When
      const booking = Booking.create(props);

      // Then
      expect(booking.status).toBe(BookingStatus.PENDING);
      expect(booking.userId.value).toBe(props.userId.value);
      expect(booking.roomId.value).toBe(props.roomId.value);
    });

    it("should emit BookingCreatedEvent when created", () => {
      // Given
      const props = BookingFixture.validCreateProps();

      // When
      const booking = Booking.create(props);

      // Then
      expect(booking.domainEvents).toHaveLength(1);
      expect(booking.domainEvents[0]).toBeInstanceOf(BookingCreatedEvent);
    });

    it("should generate a new UUID if not provided", () => {
      // Given
      const props = BookingFixture.validCreateProps();

      // When
      const booking = Booking.create(props);

      // Then
      expect(booking.id).toBeDefined();
      expect(booking.id.value).toMatch(
        /^[0-9a-f]{8}-[0-9a-f]{4}-[1-5][0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i
      );
    });
  });

  describe("confirm", () => {
    it("should change status to CONFIRMED", () => {
      // Given
      const booking = BookingFixture.pending();

      // When
      booking.confirm();

      // Then
      expect(booking.status).toBe(BookingStatus.CONFIRMED);
    });

    it("should throw when booking is already cancelled", () => {
      // Given
      const booking = BookingFixture.cancelled();

      // When/Then
      expect(() => booking.confirm()).toThrow(BookingAlreadyCancelledError);
    });
  });

  describe("cancel", () => {
    it("should change status to CANCELLED", () => {
      // Given
      const booking = BookingFixture.pending();

      // When
      booking.cancel();

      // Then
      expect(booking.status).toBe(BookingStatus.CANCELLED);
    });

    it("should emit BookingCancelledEvent", () => {
      // Given
      const booking = BookingFixture.pending();
      booking.clearDomainEvents(); // Clear creation event

      // When
      booking.cancel();

      // Then
      expect(booking.domainEvents).toHaveLength(1);
      expect(booking.domainEvents[0].eventName).toBe("booking.cancelled");
    });

    it("should throw when already cancelled", () => {
      // Given
      const booking = BookingFixture.cancelled();

      // When/Then
      expect(() => booking.cancel()).toThrow(BookingAlreadyCancelledError);
    });
  });
});
```

---

## Tests de Value Objects

```typescript
// tests/contexts/bookings/domain/value-objects/date-range.vo.test.ts
import { describe, it, expect } from "vitest";
import { DateRange } from "@/bookings/domain/value-objects/date-range.vo.js";
import { InvalidDateRangeError } from "@/bookings/domain/errors/booking.error.js";

describe("DateRange Value Object", () => {
  describe("create", () => {
    it("should create a valid date range", () => {
      // Given
      const startDate = new Date("2024-03-01");
      const endDate = new Date("2024-03-05");

      // When
      const dateRange = DateRange.create(startDate, endDate);

      // Then
      expect(dateRange.startDate).toEqual(startDate);
      expect(dateRange.endDate).toEqual(endDate);
    });

    it("should throw when start date is after end date", () => {
      // Given
      const startDate = new Date("2024-03-05");
      const endDate = new Date("2024-03-01");

      // When/Then
      expect(() => DateRange.create(startDate, endDate)).toThrow(InvalidDateRangeError);
    });

    it("should throw when start date equals end date", () => {
      // Given
      const date = new Date("2024-03-01");

      // When/Then
      expect(() => DateRange.create(date, date)).toThrow(InvalidDateRangeError);
    });
  });

  describe("nights", () => {
    it("should calculate correct number of nights", () => {
      // Given
      const dateRange = DateRange.create(
        new Date("2024-03-01"),
        new Date("2024-03-05")
      );

      // When/Then
      expect(dateRange.nights).toBe(4);
    });
  });

  describe("overlaps", () => {
    it("should detect overlapping ranges", () => {
      // Given
      const range1 = DateRange.create(new Date("2024-03-01"), new Date("2024-03-10"));
      const range2 = DateRange.create(new Date("2024-03-05"), new Date("2024-03-15"));

      // When/Then
      expect(range1.overlaps(range2)).toBe(true);
      expect(range2.overlaps(range1)).toBe(true);
    });

    it("should return false for non-overlapping ranges", () => {
      // Given
      const range1 = DateRange.create(new Date("2024-03-01"), new Date("2024-03-05"));
      const range2 = DateRange.create(new Date("2024-03-10"), new Date("2024-03-15"));

      // When/Then
      expect(range1.overlaps(range2)).toBe(false);
    });

    it("should return false for adjacent ranges", () => {
      // Given
      const range1 = DateRange.create(new Date("2024-03-01"), new Date("2024-03-05"));
      const range2 = DateRange.create(new Date("2024-03-05"), new Date("2024-03-10"));

      // When/Then
      expect(range1.overlaps(range2)).toBe(false);
    });
  });

  describe("equals", () => {
    it("should return true for equal date ranges", () => {
      // Given
      const range1 = DateRange.create(new Date("2024-03-01"), new Date("2024-03-05"));
      const range2 = DateRange.create(new Date("2024-03-01"), new Date("2024-03-05"));

      // When/Then
      expect(range1.equals(range2)).toBe(true);
    });

    it("should return false for different date ranges", () => {
      // Given
      const range1 = DateRange.create(new Date("2024-03-01"), new Date("2024-03-05"));
      const range2 = DateRange.create(new Date("2024-03-01"), new Date("2024-03-06"));

      // When/Then
      expect(range1.equals(range2)).toBe(false);
    });
  });
});
```

---

## Tests de Casos de Uso

```typescript
// tests/contexts/bookings/application/use-cases/create-booking.usecase.test.ts
import { describe, it, expect, beforeEach, vi } from "vitest";
import { CreateBookingUseCase } from "@/bookings/application/use-cases/create-booking.usecase.js";
import { CreateBookingDto } from "@/bookings/application/dtos/create-booking.dto.js";
import { RoomNotAvailableError } from "@/bookings/domain/errors/booking.error.js";
import { MockBookingRepository } from "../../mocks/booking.repository.mock.js";
import { MockRoomRepository } from "@/test/contexts/rooms/mocks/room.repository.mock.js";
import { RoomFixture } from "@/test/contexts/rooms/fixtures/room.fixture.js";

describe("CreateBookingUseCase", () => {
  let useCase: CreateBookingUseCase;
  let bookingRepository: MockBookingRepository;
  let roomRepository: MockRoomRepository;

  beforeEach(() => {
    bookingRepository = new MockBookingRepository();
    roomRepository = new MockRoomRepository();
    useCase = new CreateBookingUseCase(bookingRepository, roomRepository);
  });

  describe("execute", () => {
    it("should create a booking when room is available", async () => {
      // Given
      const room = RoomFixture.available();
      roomRepository.setFindByIdResult(room);
      bookingRepository.setExistsOverlappingResult(false);

      const input: CreateBookingDto = {
        userId: "550e8400-e29b-41d4-a716-446655440001",
        roomId: room.id.value,
        checkIn: "2024-03-01",
        checkOut: "2024-03-05",
      };

      // When
      const result = await useCase.execute(input);

      // Then
      expect(result.id).toBeDefined();
      expect(result.userId).toBe(input.userId);
      expect(result.roomId).toBe(input.roomId);
      expect(result.nights).toBe(4);
      expect(result.status).toBe("PENDING");
      expect(bookingRepository.saveCalled).toBe(true);
    });

    it("should throw RoomNotAvailableError when room is not available", async () => {
      // Given
      const room = RoomFixture.unavailable();
      roomRepository.setFindByIdResult(room);

      const input: CreateBookingDto = {
        userId: "550e8400-e29b-41d4-a716-446655440001",
        roomId: room.id.value,
        checkIn: "2024-03-01",
        checkOut: "2024-03-05",
      };

      // When/Then
      await expect(useCase.execute(input)).rejects.toThrow(RoomNotAvailableError);
    });

    it("should throw RoomNotAvailableError when dates conflict with existing booking", async () => {
      // Given
      const room = RoomFixture.available();
      roomRepository.setFindByIdResult(room);
      bookingRepository.setExistsOverlappingResult(true);

      const input: CreateBookingDto = {
        userId: "550e8400-e29b-41d4-a716-446655440001",
        roomId: room.id.value,
        checkIn: "2024-03-01",
        checkOut: "2024-03-05",
      };

      // When/Then
      await expect(useCase.execute(input)).rejects.toThrow(RoomNotAvailableError);
    });

    it("should calculate correct total price based on nights and room price", async () => {
      // Given
      const room = RoomFixture.withPricePerNight(100);
      roomRepository.setFindByIdResult(room);
      bookingRepository.setExistsOverlappingResult(false);

      const input: CreateBookingDto = {
        userId: "550e8400-e29b-41d4-a716-446655440001",
        roomId: room.id.value,
        checkIn: "2024-03-01",
        checkOut: "2024-03-05", // 4 nights
      };

      // When
      const result = await useCase.execute(input);

      // Then
      expect(result.totalPrice).toBe(400); // 4 nights * $100
    });
  });
});
```

### Test con Either

```typescript
// tests/contexts/bookings/application/use-cases/cancel-booking.usecase.test.ts
import { describe, it, expect, beforeEach } from "vitest";
import { CancelBookingUseCase } from "@/bookings/application/use-cases/cancel-booking.usecase.js";
import { BookingNotFoundError, BookingAlreadyCancelledError } from "@/bookings/domain/errors/booking.error.js";
import { MockBookingRepository } from "../../mocks/booking.repository.mock.js";
import { BookingFixture } from "../../fixtures/booking.fixture.js";

describe("CancelBookingUseCase", () => {
  let useCase: CancelBookingUseCase;
  let bookingRepository: MockBookingRepository;

  beforeEach(() => {
    bookingRepository = new MockBookingRepository();
    useCase = new CancelBookingUseCase(bookingRepository);
  });

  describe("execute", () => {
    it("should return Right with cancelled booking when successful", async () => {
      // Given
      const booking = BookingFixture.pending();
      bookingRepository.setFindByIdResult(booking);

      // When
      const result = await useCase.execute({ bookingId: booking.id.value });

      // Then
      expect(result.isRight()).toBe(true);
      if (result.isRight()) {
        expect(result.value.status).toBe("CANCELLED");
      }
    });

    it("should return Left with BookingNotFoundError when booking does not exist", async () => {
      // Given
      bookingRepository.setFindByIdResult(null);

      // When
      const result = await useCase.execute({ bookingId: "non-existent-id" });

      // Then
      expect(result.isLeft()).toBe(true);
      if (result.isLeft()) {
        expect(result.value).toBeInstanceOf(BookingNotFoundError);
      }
    });

    it("should return Left with BookingAlreadyCancelledError when already cancelled", async () => {
      // Given
      const booking = BookingFixture.cancelled();
      bookingRepository.setFindByIdResult(booking);

      // When
      const result = await useCase.execute({ bookingId: booking.id.value });

      // Then
      expect(result.isLeft()).toBe(true);
      if (result.isLeft()) {
        expect(result.value).toBeInstanceOf(BookingAlreadyCancelledError);
      }
    });
  });
});
```

---

## Fixtures y Object Mothers

```typescript
// tests/contexts/bookings/fixtures/booking.fixture.ts
import { Uuid } from "@/shared/domain/uuid.js";
import { Booking } from "@/bookings/domain/entities/booking.entity.js";
import { DateRange } from "@/bookings/domain/value-objects/date-range.vo.js";
import { BookingStatus } from "@/bookings/domain/value-objects/booking-status.vo.js";
import { DateRangeFixture } from "./date-range.fixture.js";

export class BookingFixture {
  static readonly DEFAULT_USER_ID = "550e8400-e29b-41d4-a716-446655440001";
  static readonly DEFAULT_ROOM_ID = "550e8400-e29b-41d4-a716-446655440002";
  static readonly DEFAULT_BOOKING_ID = "550e8400-e29b-41d4-a716-446655440003";

  static validCreateProps(overrides?: Partial<{
    userId: Uuid;
    roomId: Uuid;
    dateRange: DateRange;
    totalPrice: number;
  }>) {
    return {
      userId: overrides?.userId ?? Uuid.fromString(this.DEFAULT_USER_ID),
      roomId: overrides?.roomId ?? Uuid.fromString(this.DEFAULT_ROOM_ID),
      dateRange: overrides?.dateRange ?? DateRangeFixture.nextWeek(),
      totalPrice: overrides?.totalPrice ?? 400,
    };
  }

  static pending(overrides?: Partial<{ id: Uuid }>): Booking {
    const props = this.validCreateProps();
    const booking = Booking.create(props, overrides?.id);
    booking.clearDomainEvents();
    return booking;
  }

  static confirmed(): Booking {
    const booking = this.pending();
    booking.confirm();
    return booking;
  }

  static cancelled(): Booking {
    return Booking.reconstitute(
      Uuid.fromString(this.DEFAULT_BOOKING_ID),
      {
        userId: Uuid.fromString(this.DEFAULT_USER_ID),
        roomId: Uuid.fromString(this.DEFAULT_ROOM_ID),
        dateRange: DateRangeFixture.nextWeek(),
        status: BookingStatus.CANCELLED,
        totalPrice: 400,
        createdAt: new Date(),
        updatedAt: new Date(),
      }
    );
  }

  static withDates(checkIn: Date, checkOut: Date): Booking {
    const props = this.validCreateProps({
      dateRange: DateRange.create(checkIn, checkOut),
    });
    return Booking.create(props);
  }
}

// tests/contexts/bookings/fixtures/date-range.fixture.ts
import { DateRange } from "@/bookings/domain/value-objects/date-range.vo.js";

export class DateRangeFixture {
  static nextWeek(): DateRange {
    const today = new Date();
    const startDate = new Date(today);
    startDate.setDate(today.getDate() + 7);
    
    const endDate = new Date(startDate);
    endDate.setDate(startDate.getDate() + 4);
    
    return DateRange.create(startDate, endDate);
  }

  static custom(startOffset: number, endOffset: number): DateRange {
    const today = new Date();
    const startDate = new Date(today);
    startDate.setDate(today.getDate() + startOffset);
    
    const endDate = new Date(today);
    endDate.setDate(today.getDate() + endOffset);
    
    return DateRange.create(startDate, endDate);
  }

  static fromStrings(start: string, end: string): DateRange {
    return DateRange.create(new Date(start), new Date(end));
  }
}
```

---

## Mocks de Repositorios

```typescript
// tests/contexts/bookings/mocks/booking.repository.mock.ts
import { Uuid } from "@/shared/domain/uuid.js";
import { Booking } from "@/bookings/domain/entities/booking.entity.js";
import {
  BookingRepository,
  FindBookingsFilters,
  PaginationOptions,
  PaginatedResult,
} from "@/bookings/domain/repositories/booking.repository.js";
import { DateRange } from "@/bookings/domain/value-objects/date-range.vo.js";

export class MockBookingRepository implements BookingRepository {
  private findByIdResult: Booking | null = null;
  private findAllResult: Booking[] = [];
  private existsOverlappingResult = false;
  
  // Tracking calls
  saveCalled = false;
  savedBooking: Booking | null = null;
  deleteCalled = false;
  deletedId: Uuid | null = null;

  // Setup methods
  setFindByIdResult(booking: Booking | null): void {
    this.findByIdResult = booking;
  }

  setFindAllResult(bookings: Booking[]): void {
    this.findAllResult = bookings;
  }

  setExistsOverlappingResult(exists: boolean): void {
    this.existsOverlappingResult = exists;
  }

  // Reset
  reset(): void {
    this.findByIdResult = null;
    this.findAllResult = [];
    this.existsOverlappingResult = false;
    this.saveCalled = false;
    this.savedBooking = null;
    this.deleteCalled = false;
    this.deletedId = null;
  }

  // Implementation
  async findById(_id: Uuid): Promise<Booking | null> {
    return this.findByIdResult;
  }

  async findAll(_filters?: FindBookingsFilters): Promise<Booking[]> {
    return this.findAllResult;
  }

  async findPaginated(
    _filters: FindBookingsFilters,
    pagination: PaginationOptions
  ): Promise<PaginatedResult<Booking>> {
    return {
      data: this.findAllResult.slice(0, pagination.limit),
      total: this.findAllResult.length,
    };
  }

  async findByUserId(_userId: Uuid): Promise<Booking[]> {
    return this.findAllResult;
  }

  async findByRoomId(_roomId: Uuid): Promise<Booking[]> {
    return this.findAllResult;
  }

  async existsOverlapping(_roomId: Uuid, _dateRange: DateRange): Promise<boolean> {
    return this.existsOverlappingResult;
  }

  async save(booking: Booking): Promise<void> {
    this.saveCalled = true;
    this.savedBooking = booking;
  }

  async delete(id: Uuid): Promise<void> {
    this.deleteCalled = true;
    this.deletedId = id;
  }
}
```

---

## Tests de Integración

```typescript
// tests/integration/bookings/create-booking.integration.test.ts
import { describe, it, expect, beforeAll, afterAll, beforeEach } from "vitest";
import { MongoClient, Db } from "mongodb";
import { MongoBookingRepository } from "@/bookings/infrastructure/persistence/mongo-booking.repository.js";
import { MongoRoomRepository } from "@/rooms/infrastructure/persistence/mongo-room.repository.js";
import { CreateBookingUseCase } from "@/bookings/application/use-cases/create-booking.usecase.js";
import { RoomFixture } from "@/test/contexts/rooms/fixtures/room.fixture.js";

describe("CreateBooking Integration", () => {
  let client: MongoClient;
  let db: Db;
  let bookingRepository: MongoBookingRepository;
  let roomRepository: MongoRoomRepository;
  let useCase: CreateBookingUseCase;

  beforeAll(async () => {
    const uri = process.env.MONGO_URI || "mongodb://localhost:27017";
    client = new MongoClient(uri);
    await client.connect();
    db = client.db("test_hotel_booking");

    bookingRepository = new MongoBookingRepository(db);
    roomRepository = new MongoRoomRepository(db);
    useCase = new CreateBookingUseCase(bookingRepository, roomRepository);
  });

  afterAll(async () => {
    await db.dropDatabase();
    await client.close();
  });

  beforeEach(async () => {
    await db.collection("bookings").deleteMany({});
    await db.collection("rooms").deleteMany({});
  });

  it("should persist booking and retrieve it", async () => {
    // Given
    const room = RoomFixture.available();
    await roomRepository.save(room);

    const input = {
      userId: "550e8400-e29b-41d4-a716-446655440001",
      roomId: room.id.value,
      checkIn: "2024-03-01",
      checkOut: "2024-03-05",
    };

    // When
    const result = await useCase.execute(input);

    // Then
    const savedBooking = await bookingRepository.findById(
      Uuid.fromString(result.id)
    );
    expect(savedBooking).not.toBeNull();
    expect(savedBooking?.userId.value).toBe(input.userId);
  });
});
```

---

## Setup Global

```typescript
// tests/setup.ts
import { beforeAll, afterAll } from "vitest";

beforeAll(() => {
  // Setup global test environment
  process.env.NODE_ENV = "test";
});

afterAll(() => {
  // Cleanup
});
```

```typescript
// vitest.config.ts - agregar setup
export default defineConfig({
  test: {
    globals: true,
    environment: "node",
    setupFiles: ["./tests/setup.ts"],
    // ...
  },
});
```
