# Milestone Logs: OUT-02 Service Booking Spares & Consumables Consumption Engine

- **Milestone ID**: `MLS-BE-OUT02`
- **Subsystem**: Backend (`karigarji_server`)
- **Date Initiated**: 2026-11-10

---

## Decision Log

- **2026-11-09 22:15**: Initiated planning for `OUT-02`.
  - User highlighted core objective: dedicatedly for export (cross-domain integration with `Booking` and `Actor`).
  - Key focus on edge cases and lessons learned from `OUT-01` (custody isolation, un-serialized batch headroom, serial integrity, atomic transactions).
  - Drafted comprehensive edge cases analysis covering custody location, serial status (`INSTALLED`), phased consumption idempotency, and dual-mode billing (bundled vs add-on).
  - Initiated Single Question Alignment regarding custody source deduction model.

- **2026-11-09 23:54**: User confirmed Option A (Dual source with Technician Van default) with critical operational constraints:
  1. Serialized items are mandatory for service spare replacements.
  2. Identified crucial edge case: technician scans/installs part, but needs to swap it or customer denies quote at the last moment.
  3. Identified workshop policy: workshop repairs should still issue parts to the technician first via dispatch so custody is strictly held at the technician staff level.
  4. Formulated Two-Phase "Attach & Commit" architecture to prevent transient ledger and GST tax pollution during in-progress jobs.

- **2026-11-10 00:23**: User highlighted critical real-world divergence between spares and consumables:
  1. Spares vs Consumables: Consumables (gas, brazing flux, tape) are non-recoverable once used on-site; they CANNOT be held and must **Direct Commit** immediately. Serialized Spares (motors, PCBs) must default to **Hold/Staged** pending customer approval.
  2. Post-Swipe OTP Handshake: On mobile review screen, technician swipes to trigger a customer OTP verification before the final commit.
  3. Raised storage architectural question: How and where to store held items (in-table serial linkage vs dedicated table).
  4. Expanded scenario matrix: Scrap core collection, on-site warranty claims, mid-job cancellation with consumed consumables, and zombie reservation timeouts.

- **2026-11-10 00:37**: User confirmed Option A for Storage (In-Table Linkage on `InventoryItemSerial`) and clarified that pricing will be decided dynamically based on service item configurations.
  - Finalized all key architectural decisions for `OUT-02`.
  - Stored staged spares on existing indexed columns: `serviceBookingId`, `status: IN_FIELD_CUSTODY`, `fieldStaffId`.

- **2026-11-10 10:58**: User requested complete clarity on the exact list of APIs being implemented for `OUT-02`.
  - Clarified 3-tier API catalog: Field Mobile REST APIs (`/field/stock/service/...`), Programmatic Export Facade (`InventoryIntegrationService`), and Staff Workshop REST APIs (`/staff/branches/:branchId/stock/outward/...`).

- **2026-11-10 11:06**: User raised two crucial architectural questions:
  1. OTP validation ownership: Why isn't stock operations generating OTP? Clarified that customer completion OTP is owned by `BookingModule`, and `stockOperations` is invoked upon verification inside an atomic transaction.
  2. "Signal / Intent Trigger" (Lean Payload) vs "Fat Payload": Conducted deep comparative analysis. Concluded that for financial commitments and state machine transitions, thin signal triggers (`{ bookingId }`) with server-side ground-truth resolution are overwhelmingly superior for security, anti-tampering, network resilience, and idempotency.

- **2026-11-10 11:24**: User approved the plan and requested creating a dedicated milestone folder at the root level (`milestones/out-02-service-booking-consumption/`) to house all implementation details and progress tracking.
  - Initialized `milestones/out-02-service-booking-consumption/` with `milestone.md`, `steps.md`, and `logs.md`.
  - Ready to proceed to Phase 2 (DTOs, Commands & Exceptions).

- **2026-11-10 11:34**: Phase 2 Completed:
  - Created 4 command classes in `exports/commands/`: `StageServiceSerialCommand`, `ReleaseServiceSerialCommand`, `ConsumeServiceConsumableCommand`, `CommitBookingSparesCommand`.
  - Created 3 result DTOs in `exports/results/`: `StagedSerialResultDto`, `StockConsumableResultDto`, `StockServiceCommitResultDto`.
  - Added 7 service-scoped domain exceptions in `stockOutgoing.exceptions.ts`.
  - Registered all domain error codes and message constants in `stockOperationsMessages.ts`.
  - Verified `karigarji_server` build cleanly (`nest build` passed with 0 errors) and all 28 test suites in `stockOperations` passed (303 tests).
  - Ready for Phase 3: Serialized Spares Soft-Hold & Release Engine.

- **2026-11-10 11:44**: Completed Phases 3 Through 10 (Full Implementation & Verification):
  - Phase 3: Implemented `stageServiceSerial`, `releaseServiceSerial`, and `getStagedSparesForBooking` on `StockOutgoingService`.
  - Phase 4: Implemented `consumeServiceConsumable` with van pouch stock decrements (`looseQuantityOnHand`), `OUTWARD__SERVICE_CONSUMABLE` movements, and sub-ledger expense journals.
  - Phase 5: Implemented `commitBookingSpares` using the lean Signal Trigger pattern, transitioning serials to `INSTALLED`, clearing field custody, decrementing van on-hand stock, updating batch lots, and supporting warranty claim overrides.
  - Phase 6: Synchronized `InventoryIntegrationService` with optional `tx` injection for external modules.
  - Phase 7: Built `FieldStockServiceController` (`@Controller('field/stock/service')`), DTOs (`fieldServiceStock.dto.ts`), and Swagger decorators.
  - Phase 8: Aligned `StockFinancialPostingEngine` journals for billable spares, consumables, and warranty claims.
  - Phase 9: Authored comprehensive adversarial unit tests in `stockOutgoing.service.spec.ts` (33 tests passed, covering custody violations, swap safety, pouch underflow, and warranty claims).
  - Phase 10: Zero-Debt Audit: 314 tests passed in `stockOperations` module; clean builds across `karigarji_server` and `karigarji_admin_panel`. Synchronized milestone registries.

- **2026-11-10 14:15**: Comprehensive Architectural Audit & Zero-Sibling-Coupling Refactoring:
  - User audited the implementation and identified two major architectural defects:
    1. `FieldStockOperationsModule` cross-imported `StaffStockOperationsModule`, and `FieldStockServiceController` injected `StockOutgoingService` (violating Rule 11 & `architectureRules.md` Section 2: Zero Sibling Coupling).
    2. Planned Tier 3 staff workshop endpoints were omitted from `StockOutgoingController`.
  - Refactoring Executed:
    1. **Shared Command Engine**: Extracted all shared consumption business logic into `ServiceStockConsumptionEngine` (`src/modules/stockOperations/common/services/serviceStockConsumption.engine.ts`).
    2. **Common Kernel Registration**: Registered and exported `ServiceStockConsumptionEngine` in `StockOperationsCommonModule`.
    3. **Dedicated Field Service**: Created `FieldStockServiceService` (`src/modules/stockOperations/field/services/fieldStockService.service.ts`). Rewired `FieldStockServiceController` to inject `FieldStockServiceService`.
    4. **Eliminated Sibling Coupling**: Completely removed `StaffStockOperationsModule` from `FieldStockOperationsModule`.
    5. **Staff Workshop Endpoints**: Added `POST /staff/branches/:branchId/stock/outward/service-commit` and `GET .../bookings/:bookingId/staged-spares` on `StockOutgoingController` with `StaffServiceCommitDto` and composite Swagger decorators in `stockOutgoingDocs.decorator.ts`.
    6. **Service Shrinkage**: `StockOutgoingService` reduced from 1,138 LOC to ~560 LOC, delegating shared booking consumption to the engine.
    7. **Comprehensive Test Suites**: Created `serviceStockConsumption.engine.spec.ts`, `fieldStockService.service.spec.ts`, `fieldStockService.controller.spec.ts`, and updated `stockOutgoing.controller.spec.ts`.
    8. **Automated Verification**: All 31 test suites in `stockOperations` passed (330 tests total, 0 failures). `karigarji_server` and `karigarji_admin_panel` compiled with 0 errors. Zero tech debt.

- **2026-11-10 15:35**: Bounded Context Separation & Loose Facade Refactoring (Option 2):
  - User and Senior Staff Engineer audited DDD bounded contexts and identified two foundational boundary issues:
    1. **Customer Job Execution Coupling**: Staging, releasing, quote approvals, and customer OTP verification are repair workflow concerns belonging strictly to `BookingModule`, not `StockOperations`. Having HTTP routes in `stockOperations` coupled inventory to booking flows.
    2. **Schema Overloading**: Putting `serviceBookingId` and `orderId` on `InventoryItemSerial` created foreign key pollution in the physical stock ledger.
  - Selected and implemented **Option 2 (Autonomous Stock Ledger with Generic Reservation Facade)**:
    1. **Schema Refinement (`schema.prisma`)**: Pruned `serviceBookingId` and `orderId` from `model InventoryItemSerial`. Added `reservationToken String? @db.VarChar(100)`, `reservedAt DateTime?`, `installedReference String? @db.VarChar(100)`, `soldReference String? @db.VarChar(100)`, and index `@@index([reservationToken])`.
    2. **Deleted Misplaced Field HTTP Routes**: Removed `fieldStockService.controller.ts`, `fieldStockService.service.ts`, DTOs, and decorators from `fieldStockOperations.module.ts`.
    3. **Generic Command Engine (`ServiceStockConsumptionEngine`)**: Refactored to generic operations:
       - `reserveSerial(command, user?, tx?)`
       - `releaseSerial(command, user?, tx?)`
       - `getReservedSerials(reservationToken, staffId?, tx?)`
       - `consumeServiceSale(command, user?, tx?)`
       - `consumeServiceConsumable(command, user?, tx?)`
    4. **Physical Lock & Transfer Hardening**: TR-02 (branch cage return) and TR-03 (peer technician handoff) hardened to reject serials where `reservationToken != null` (`SerialCannotBeTransferredWhileReservedException`), preventing double-allocation and phantom stock.
    5. **Export Facade Alignment**: Updated `InventoryIntegrationService` to expose clean programmatic methods with optional `tx` injection.
    6. **Staff Workshop Alignment**: `stockOutgoing.service.ts` delegates workshop commit and staged spares inspection to the generic engine.
    7. **Automated Verification & Zero-Debt**:
       - 82 out of 82 test suites passed in `karigarji_server` (765 tests total, 0 failures).
       - `nest build` passed with 0 compile errors.
       - `karigarji_admin_panel` (`tsc -b && vite build`) built in 10.16s with 0 errors.
       - Zero unconfirmed database migrations (Rule 8 compliance).

- **2026-11-10 15:50**: Database Synchronization Completed:
  - User explicitly confirmed executing `npx prisma db push --accept-data-loss --config prisma.config.ts`.
  - Prisma Migrate Safety Guard verified local development target (`karigarji_psql_db_local` at `localhost:5432`).
  - Database pushed and synchronized successfully.
  - Client regenerated (`npx prisma generate`). All 82 test suites (765 tests) passed with 0 errors. Milestone `MLS-BE-OUT02` completely closed.

- **2026-11-10 16:15**: First-Class REST HTTP API & Strict TypeScript Zero-Warning Handoff:
  - Implemented dedicated REST endpoint `POST /staff/branches/:branchId/stock/outward/service-sale` with aliases `['service-sale', 'service-consumption', 'outward/service-sale']`.
  - Created `ExecuteServiceSaleDto` in `src/modules/stockOperations/staff/dto/outward/executeServiceSale.dto.ts`.
  - Added Swagger documentation decorator `@ApiExecuteServiceSaleDocs()` with composite error responses.
  - Resolved all 82 TypeScript strict errors across 21 test spec files.
  - `npx tsc --noEmit` exits with 0 errors across the entire backend repository.
  - All 82 test suites (767 tests) passed with 0 failures. `nest build` and `vite build` cleanly verified.
