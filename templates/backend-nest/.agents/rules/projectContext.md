# Project Context: karigarJi Server Backend

## 1. Project Goal & Overview
The **karigarJi Server** is an enterprise-grade NestJS backend service serving as the single authoritative data hub and REST API for both the karigarJi administration panel and technician platforms.

### Core Domain Capabilities
* **Bookings & Dispatch**: Complete lifecycle management of service bookings, technician dispatching, status machine transitions, schedule tracking, and service tags.
* **Customers, Leads & Blacklist**: Segregated sub-domains for customer profiles, incoming lead ingestion/qualification, and fraud prevention/blacklisting.
* **Technician Operations**: Unified technician domain serving field technician workflows (bag inventory, job execution) without parallel codebase duplication.
* **Inventory Management**: Stock movements, warehouse purchasing, technician bag allocations, and consumption logging.
* **Invoices & Billing**: Synchronized two-stage financial calculations (line-item discounts, global discounts, GST, nearest-integer rounding), append-only payment recording, and decoupled physical stock consumption.
* **Accounts & Financial Ledger**: Double-entry style financial records, expense logging, and category accounting driven asynchronously via domain events.
* **Staff, Roles & RBAC**: Granular permissions and role-based access control (`SUPER_ADMIN`, `ADMIN`, `OPERATOR`, `STOCK_KEEPER`, `CASHIER`, `TECHNICIAN`).

---

## 2. Technology Stack & Architectural Standards
* **Framework**: NestJS 11+ (TypeScript, Node.js) organized in a Modular Domain Architecture (`src/modules/*`).
* **Database & ORM**: PostgreSQL database managed via Prisma ORM client with strict relational integrity (pure PostgreSQL; legacy MySQL artifacts deprecated).
* **ID Generation**: Distributed, chronological, 64-bit Snowflake IDs (`SnowflakeService`) for collision-free booking numbers, invoices, and transactions.
* **Service Layer Pattern**: 3-tier service segregation:
  1. *Application / Orchestration Service*: Workflow coordination, transaction boundaries, domain event dispatch.
  2. *Domain / Calculation Service*: Pure algorithmic calculations (pricing, GST, discounts) with zero database I/O.
  3. *Policy / Rule Verification Service*: Isolated precondition and business policy checks.
* **Data Access Layer**: Direct Prisma queries for simple lookups/CRUD; Pragmatic Repositories or Prisma Client Extensions (`$extends`) for complex queries, raw SQL, and heavy reusable relations.
* **Cross-Module Communication**: Asynchronous domain events via NestJS `EventEmitter2` (`@nestjs/event-emitter`) to prevent tight coupling across database tables.
* **API Style**: Idiomatic RESTful API conventions (HTTP verbs denote actions, plural nouns denote resources) with explicit `@HttpCode(...)`.
* **Authentication**: Global `JwtAuthGuard` (registered via `APP_GUARD`) with `@Public()` decorator for open endpoints; super admin initialized via database seed (`prisma/seed.ts`).
* **Authorization**: Declarative `@Roles(...)` and `@RequirePermissions(...)` decorators evaluated against system privileges.
* **Response Automation**: Global `TransformResponseInterceptor` producing standardized envelopes (`{ success, statusCode, message, data, meta }`); controllers return raw domain data.
* **Validation & Transformation**: Global `ValidationPipe({ transform: true, whitelist: true })`, input string sanitization via `TrimPipe`, and unified `PaginationQueryDto`.
* **Error Handling**: Centralized strongly-typed domain exceptions mapped via `GlobalExceptionFilter`, with automatic Prisma error code translation (`P2002`, `P2025`, `P2003`) and sensitive payload log masking.
* **API Documentation**: OpenAPI / Swagger (`@nestjs/swagger`) with typed DTO annotations (`@ApiProperty()`) and endpoint definitions.

---

## 3. Target Structural Map
* `/src/modules/`: High-cohesion domain modules (`auth/`, `accounts/`, `bookings/`, `customers/`, `leads/`, `blacklist/`, `inventory/`, `invoices/`, `technicians/`, `serviceTags/`).
* `/src/common/`: Cross-cutting application infrastructure:
  * `decorators/`: `@Public()`, `@CurrentUser()`, `@Roles()`, `@RequirePermissions()`
  * `dto/`: Base pagination, sorting, and date-range DTOs
  * `filters/`: `GlobalExceptionFilter` (Prisma error mapping, HTTP exceptions, payload masking)
  * `guards/`: `JwtAuthGuard`, `PermissionsGuard`
  * `interceptors/`: `TransformResponseInterceptor`, logging interceptors
  * `pipes/`: `TrimPipe`, date parsers
  * `services/`: `SnowflakeService`
* `/src/config/`: Type-safe environment and database configurations (`database.config.ts`, `app.config.ts`).
* `/src/database/`: `PrismaModule` and `PrismaService`.
* `/prisma/`: Prisma schema (`schema.prisma`), migrations, and seed scripts (`seed.ts`).
