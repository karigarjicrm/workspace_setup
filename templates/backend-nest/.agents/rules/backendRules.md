# Backend & Database Architecture Rules

This file defines the authoritative engineering rules, architectural patterns, controller contracts, 3-tier service responsibilities, database query layers, and business calculation logic for the NestJS backend (`karigarji_cms`).

---

## 1. Modular Architecture & Directory Standards

Every backend domain lives in `src/modules/<domainName>/` and adheres to single-responsibility segregation:

```javascript
src/modules/bookings/
├── bookings.module.ts              # NestJS module definition & dependency injection (exports public providers)
├── controllers/                    # HTTP protocol adapters (thin controllers)
│   ├── booking.controller.ts       # Core collection routes (/bookings)
│   ├── bookingNotes.controller.ts  # Nested resource routes (/bookings/:id/notes)
│   └── bookingStatus.controller.ts # State machine transition routes (/bookings/:id/status)
├── dto/                            # Strict validation & query schemas
│   ├── createBooking.dto.ts        # Creation payload schema
│   ├── updateBooking.dto.ts        # Partial update schema
│   ├── bookingQuery.dto.ts         # Filter, search & pagination schema
│   └── bookingResponse.dto.ts      # Explicit output serialization
├── services/                       # 3-Tier business logic segregation
│   ├── booking.service.ts          # Tier 1: Application / Orchestration service
│   ├── bookingCalculator.service.ts# Tier 2: Pure domain calculation service (zero DB I/O)
│   └── bookingDuplicate.service.ts # Tier 3: Policy & precondition verification service
├── repositories/                   # Pragmatic data access layer (optional)
│   └── booking.repository.ts       # Encapsulated complex queries, joins & raw SQL
├── events/                         # Outgoing domain events
│   ├── bookingCreated.event.ts
│   └── bookingStatusUpdated.event.ts
├── listeners/                      # Incoming domain event listeners
│   └── technicianAssigned.listener.ts
└── types/                          # Internal domain interfaces and enums
    └── booking.types.ts
```

### Module Boundary Principles

1. **Sub-Resource Controller Separation**: Avoid monolithic controllers. Split sub-resources (`/bookings/:id/notes`) and explicit state-machine transitions (`/bookings/:id/status`) into dedicated controller files.
2. **No Barrel Files (index.ts) & Explicit Module Boundaries**: Do not use `index.ts` barrel files inside domain modules. NestJS natively manages runtime dependency boundaries through `@Module({ exports: [...] })`. Avoiding barrel files eliminates circular dependency traps and reduces unnecessary boilerplate. External modules must import explicitly from the module file (e.g. `import { BookingsModule } from './modules/bookings/bookings.module'`).
3. **Sub-Domain Slicing**: Divide complex domains with distinct functional areas (e.g. `invoices`) into sub-domains: `core/`, `payments/`, `templates/`, `settings/`.
4. **Domain Separation**: Segregate independent lifecycles into dedicated modules (e.g. `leads/` and `blacklist/` must be separate from `customers/`). Avoid parallel sub-app duplication (`technicianPortal` must not duplicate core domain logic; share domain services across controllers).

---

## 2. Controllers vs. Services (Strict Separation of Concerns)

### Thin Controllers (HTTP Protocol Translators)

Controllers act strictly as HTTP adapters and protocol translators:

- **Allowed in Controllers**:
  - Route declarations (`@Controller('bookings')`, `@Get()`, `@Post()`, `@Patch()`, `@Delete()`).
  - Explicit HTTP status codes (`@HttpCode(HttpStatus.CREATED)`, `@HttpCode(HttpStatus.OK)`).
  - Route guards & RBAC decorators (`@Roles(...)`, `@RequirePermissions(...)`, `@Public()`).
  - Request parameter binding (`@Body()`, `@Param()`, `@Query()`, `@CurrentUser()`).
  - OpenAPI decorators (`@ApiTags()`, `@ApiOperation()`, `@ApiResponse()`).
  - Invoking a single orchestration service method and returning raw domain data directly.
- **FORBIDDEN in Controllers**:
  - Direct database calls (`this.prisma.booking.findMany(...)`).
  - Business policy validations (e.g. checking status or permissions inside controller logic).
  - Multi-step cross-service orchestration.
  - Database transaction management (`$transaction`).
  - Manual response envelope packaging (handled automatically by `TransformResponseInterceptor`).

### The 3-Tier Service Breakdown

Business logic must be segregated into three distinct tiers:

- **Tier 1: Application / Orchestration Service (booking.service.ts)**:
  - Coordinates end-to-end use-case workflows.
  - Manages database transaction boundaries (`prisma.$transaction`).
  - Coordinates domain services, policy checkers, and repositories.
  - Dispatches domain events via `EventEmitter2` for decoupled side-effects.
- **Tier 2: Domain / Calculation Service (bookingCalculator.service.ts)**:
  - Pure domain formulas, pricing calculations, GST taxes, and discounts.
  - **ZERO database I/O**. Must take inputs and return pure outputs for deterministic unit testing.
- **Tier 3: Policy / Rule Verification Service (bookingDuplicate.service.ts)**:
  - Isolates precondition checks, state transition validity, duplicate detection, and threshold verifications.

---

## 3. Database Query Layer: Pragmatic Repositories vs. Direct Prisma

### The Pragmatic Repository Rule

Avoid anemic 1:1 wrappers around Prisma methods (`findById`, `delete`). Use direct `PrismaService` in services for simple CRUD and single-table lookups.

Extract database operations to a dedicated **Repository** (`booking.repository.ts`) or **Prisma Client Extension** (`$extends`) **ONLY** when they meet one of these criteria:

1. **Complex Aggregations & Metrics**: Monthly revenue, reporting statistics, or multi-metric grouping.
2. **Raw SQL Operations**: `$queryRaw` queries, complex UNIONs, or full-text search.
3. **Heavy Reusable Relations**: Large, multi-level relational `include` blocks. Centralize strongly-typed include objects (`detailedInclude`) in the repository.
4. **Complex Transactional Workflows**: Atomic multi-entity batch updates.

---

## 4. Cross-Module Communication via Domain Events

- **No Direct Foreign Table Mutations**: A module must never directly mutate or query tables owned by another domain (e.g. `inventory` must not directly insert records into `accountTransaction` or `vendor`).
- **Decouple with EventEmitter2**: Emit strongly-typed domain events when state changes occur:
- **Handle in Domain Listeners**: Other modules consume events using `@OnEvent(...)` listeners and execute their own domain logic independently.

---

## 5. RESTful API Routing & HTTP Conventions

All endpoints must follow standard RESTful conventions:

- **HTTP Verbs Represent Actions**:
  - `GET /api/v1/bookings`: List bookings (filterable, paginated).
  - `POST /api/v1/bookings`: Create a new booking.
  - `GET /api/v1/bookings/:id`: Retrieve single booking details.
  - `PATCH /api/v1/bookings/:id`: Partially update a booking.
  - `DELETE /api/v1/bookings/:id`: Soft delete a booking.
  - `POST /api/v1/bookings/:id/notes`: Create sub-resource note.
  - `PATCH /api/v1/bookings/:id/status`: Explicit state machine transition.
- **No RPC-Style Paths**: Never use action verbs in routes (e.g. NO `/booking/create`, `/booking/list`, `/customer/add`, `/booking/detail/:id`, `/booking/update/:id`).
- **Resource Plurality**: Use plural nouns for entity collections (`/bookings`, `/customers`, `/invoices`, `/technicians`).
- **Explicit HTTP Status Codes**: Use `@HttpCode(...)` decorators on every controller method:
  - `HttpStatus.CREATED` (`201`) for resource creation.
  - `HttpStatus.OK` (`200`) for reads, updates, and deletes.

---

## 6. Standardized Response Formatting (Global Interceptor)

- **No Manual Envelope Construction**: Never write manual `{ success: true, message, body }` objects inside controller methods.
- **Global Interceptor**: The global `TransformResponseInterceptor` automatically standardizes all successful responses into the unified structure:
- Controllers simply return domain entities or plain objects directly.

---

## 7. Validation, Input Transformation & Pagination

- **Global Validation Pipe**: `ValidationPipe` must be configured globally in `main.ts` with `{ transform: true, whitelist: true }` to automatically cast query parameters into typed primitives.
- **Input Sanitization (TrimPipe)**: Use `TrimPipe` globally to strip leading and trailing whitespace from string inputs.
- **Standard Pagination DTO**: All list queries must inherit from or utilize `PaginationQueryDto`:
- Lists must always enforce maximum limits (e.g. max `100`) to prevent database memory exhaustion.

---

## 8. Authentication, Authorization & Security

- **Global JwtAuthGuard**: Register `JwtAuthGuard` globally via `APP_GUARD` in `AppModule`. Do not manually attach auth guards to individual controllers.
- **Public Endpoint Opt-Out (@Public())**: Public routes (e.g. login, webhook endpoints) must opt out using the `@Public()` custom decorator. Never use hardcoded route path strings inside guard code.
- **Super Admin Initialization**: Initialize the Super Admin user via the database seed script (`prisma/seed.ts`), never through an open unauthenticated registration endpoint.
- **Declarative RBAC**: Use `@Roles(...)` and `@RequirePermissions(...)` decorators for role and permission checks.
- **Log Sanitization**: Ensure the `GlobalExceptionFilter` masks sensitive fields (`password`, `token`, `authorization`, `creditCard`) before logging request payloads or headers.

---

## 9. Snowflake ID Generation

- **Distributed IDs**: Use the injectable `SnowflakeService` for distributed, 64-bit, chronological, collision-free numeric identifiers for all major business documents (e.g. bookings, orders, invoices, transactions).
- Never rely on sequential auto-incrementing integer IDs for public-facing business records.

---

## 10. Soft Delete & Database Integrity Policies

- **No Hard Cascades**: Never use `onDelete: Cascade` on master records or transaction history tables.
- **Soft Delete Pattern**: Master records implement soft deletion via a nullable `deletedAt DateTime?` field.
- **Query Filtering**: All queries, relation joins, and count operations must explicitly filter out deleted rows (`where: { deletedAt: null }`).
- **Foreign Key Constraints**: Use `onDelete: Restrict` or `onDelete: SetNull` to prevent orphaned child records. Verify child record counts are zero before permitting deletion.
- **Unique Constraints Against Race Conditions**: Never rely only on service-level existence checks (`findFirst`). Always declare database-level unique constraints (e.g. `@@unique([storeId, name])`) to guarantee integrity during concurrent requests.
- **Indexes**: Declare database indexes (`@@index(...)`) on all search/filter columns, foreign keys, and sorting fields.

---

## 11. ACID Transaction Safety & Performance

- **Narrow Transaction Scopes**: Keep `prisma.$transaction(...)` blocks minimal and focused strictly on atomic database mutations.
- **No Blocking Operations in Transactions**: NEVER execute external HTTP requests, file uploads, image processing, or password hashing *inside* a database transaction block. Resolve all computations before opening the transaction.
- **Select vs. Include**: Use `select: { ... }` instead of wide `include` blocks to query only required columns. This minimizes database wire traffic and prevents accidental leakage of sensitive attributes (e.g. password hashes).

---

## 12. Centralized Domain Exceptions

- **Typed Domain Exceptions**: Replace generic `throw new ApiError('...', ...)` and informal emoji error messages with strongly-typed domain exceptions (`BookingNotFoundException`, `InsufficientStockException`, `PaymentMismatchException`).
- **Global Exception Filter Prisma Mapping**: `GlobalExceptionFilter` automatically maps Prisma database errors to clean HTTP responses:
  - `P2002` -> `HttpStatus.CONFLICT` (409) with conflicting field details.
  - `P2025` -> `HttpStatus.NOT_FOUND` (404) record not found.
  - `P2003` -> `HttpStatus.CONFLICT` (409) foreign key constraint failure.

---

## 13. Invoice Calculations, Payments & Stock Decoupling

### Two-Stage Mathematical Flow

All invoice calculations (backend and frontend) must follow the synchronized mathematical specification:

- **Stage 1: Line-Item Calculation (calculateItemTotals)**:
  - $\text{grossAmount} = \text{quantity} \times \text{unitPrice}$
  - $\text{discountAmount} = \text{isFlat} ? (\text{quantity} \times \text{discountValue}) : (\text{grossAmount} \times \frac{\text{discountValue}}{100})$
  - Item Discount Cap: $\text{discountAmount} = \min(\text{discountAmount}, \text{grossAmount})$
  - Net Item Price: $\text{totalPrice} = \text{grossAmount} - \text{discountAmount}$
- **Stage 2: Invoice-Level Calculation (calculateInvoiceTotals)**:
  - $\text{subTotal} = \sum \text{totalPrice}\_{\text{items}}$
  - Global Discount: $\text{globalDiscountAmount} = \text{isFlat} ? \text{globalDiscountValue} : (\text{subTotal} \times \frac{\text{globalDiscountValue}}{100})$
  - Global Discount Cap: $\text{globalDiscountAmount} = \min(\text{globalDiscountAmount}, \text{subTotal})$
  - $\text{taxableAmount} = \text{subTotal} - \text{globalDiscountAmount}$
  - $\text{taxAmount} = \text{isGst} ? (\text{taxableAmount} \times \frac{\text{taxRate}}{100}) : 0$
  - **Payable Grand Total (Nearest Integer)**: $\text{totalAmount} = \text{Math.round}(\text{taxableAmount} + \text{taxAmount})$

### Payment State Machine & Append-Only Rules

- **Draft / Sent**: Payments optional.
- **Partially Paid**: Requires $0 < \sum \text{payments} < \text{totalAmount}$.
- **Paid**: Requires $\sum \text{payments} == \text{totalAmount}$ (within $\pm 0.01$ margin).
- **Immutable Historical Payments**: Once an invoice is saved under `PARTIALLY_PAID` or `PAID`, recorded payments cannot be updated or deleted. Operators may only append new payments.

### Decoupled Stock Consumption

- Line items represent billing services only. No spare parts are attached directly to line-item entries.
- Technician stock bag deductions are managed through a dedicated consumption ledger, independent from invoice state transitions.
- Consumed parts can be logged or reverted until the invoice is `PAID`, `CANCELLED`, or `VOID`.

---

## 14. OpenAPI / Swagger Documentation

- All controller classes must be decorated with `@ApiTags('<Domain>')`.
- Every endpoint handler must include `@ApiOperation({ summary: '...' })` and explicit `@ApiResponse(...)` definitions for standard success and error statuses.
- All DTO fields must include `@ApiProperty()` or `@ApiPropertyOptional()` with description, type, and example values.

---

## 15. Pure PostgreSQL Database Configuration

- Maintain pure PostgreSQL configuration (`database.config.ts`).
- Legacy MySQL artifacts (`mysqldb.config.ts`, `mysqlErrCodes.ts`, `mySqlError.helper.ts`, `mysql2`, `mariadb`, `@prisma/adapter-mariadb`) are obsolete and must not be used or reintroduced.
