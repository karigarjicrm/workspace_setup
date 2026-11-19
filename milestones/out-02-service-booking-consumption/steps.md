# Milestone Steps: OUT-02 Service Booking Spares & Consumables Consumption Engine

- **Milestone ID**: `MLS-BE-OUT02`
- **Subsystem**: Backend (`karigarji_server`)
- **Status**: 🟢 **COMPLETED**
- **Date Initiated**: 2026-11-10
- **Date Completed**: 2026-11-10

---

## 1. Phase Progress Status Board

| Phase | Description | Scope & Target Files | Verification Criteria | Status |
| :--- | :--- | :--- | :--- | :---: |
| **Phase 0** | Ground-Truth Analysis & Loopholes Audit | `schema.prisma`, `stockOutgoing.service.ts` | Van custody, serial staging, lean signal trigger | 🟢 `COMPLETED` |
| **Phase 1** | Architectural Alignment & Single Question Resolution | `steps.md`, `milestone.md` | Custody source, OTP ownership, in-table storage | 🟢 `COMPLETED` |
| **Phase 2** | Strongly-Typed Commands, DTOs & Domain Exceptions | `exports/commands/`, `exports/results/`, `stockOutgoing.exceptions.ts` | 7 domain exceptions, 4 command DTOs, clean build | 🟢 `COMPLETED` |
| **Phase 3** | Serialized Spares Soft-Hold & Release Engine | `serviceStockConsumption.engine.ts` (`stageServiceSerial`, `releaseServiceSerial`) | Unit tests pass verifying van custody, soft-lock, swap safety | 🟢 `COMPLETED` |
| **Phase 4** | Consumables Direct Commit Engine | `serviceStockConsumption.engine.ts` (`consumeServiceConsumable`) | Pouch stock decrement, `OUTWARD__SERVICE_CONSUMABLE` movement | 🟢 `COMPLETED` |
| **Phase 5** | Post-OTP Atomic Commit Engine (`OUT-02`) | `serviceStockConsumption.engine.ts` (`commitBookingSpares`) | Lean signal trigger, serial $\to$ `INSTALLED`, van custody release | 🟢 `COMPLETED` |
| **Phase 6** | Exported Cross-Domain Facade Synchronization | `exports/inventoryIntegration.service.ts` | Passthrough methods with optional `tx` support | 🟢 `COMPLETED` |
| **Phase 7** | Field & Staff Ingress Controllers & Dedicated Services | `fieldStockService.service.ts`, `fieldStockService.controller.ts`, `stockOutgoing.controller.ts` | Zero sibling coupling; 1:1 dedicated field service; staff workshop endpoints | 🟢 `COMPLETED` |
| **Phase 8** | Double-Entry Sub-Ledger Posting Engine Alignment | `stockFinancialPosting.engine.ts` | Revenue, landed COGS, consumable expense, warranty expense | 🟢 `COMPLETED` |
| **Phase 9** | Adversarial Unit & Invariant Test Suite | `serviceStockConsumption.engine.spec.ts`, `fieldStockService.service.spec.ts`, `stockOutgoing.controller.spec.ts` | 31 test suites, 330 passed tests across `stockOperations` | 🟢 `COMPLETED` |
| **Phase 10** | Zero-Debt Self-Audit & Architectural Hardening | `npm test`, `npm run build`, `karigarji_server/.agents/milestones.md` | 330 tests pass, 0 compile errors, zero sibling coupling verified | 🟢 `COMPLETED` |
| **Phase 11** | Option 2 DDD Bounded Context & Facade Refactoring | `schema.prisma`, `serviceStockConsumption.engine.ts`, `inventoryIntegration.service.ts` | Pruned foreign keys from serial, added generic `reservationToken`, deleted misplaced field HTTP transport, hardened TR-02/TR-03 physical locks, all 82 test suites (765 tests) pass | 🟢 `COMPLETED` |
| **Phase 12** | Database Schema Synchronization | `prisma db push` | Confirmed by user; local database synced in 135ms with 0 errors | 🟢 `COMPLETED` |
| **Phase 13** | Dedicated First-Class REST HTTP API | `stockOutgoing.controller.ts`, `executeServiceSale.dto.ts`, `stockOutgoingDocs.decorator.ts` | Added `POST .../stock/outward/service-sale` with OpenAPI envelope docs and unit tests | 🟢 `COMPLETED` |
| **Phase 14** | Strict TypeScript Zero-Warning Handoff | `npx tsc --noEmit` across all `.spec.ts` files | 0 errors across entire repository (`tsc --noEmit` exits 0, 767 tests pass) | 🟢 `COMPLETED` |

---

## 2. Active Resumption Checkpoint

- **Current Phase**: Milestone `MLS-BE-OUT02` Closed & Fully Verified (100% Strict Type Clean)
- **In-Flight Step**: All 14 phases implemented, audited, decoupled, database-synced, REST-exposed, and verified with 767 tests passing and 0 TypeScript errors.
- **Next Immediate Action**: Milestone completed. Ready for next scheduled initiative.

---

## 3. Itemized Implementation Checklist

### Phase 2: DTOs, Commands & Exceptions
- [x] Create `StageServiceSerialCommand` in `exports/commands/stageServiceSerial.command.ts`
- [x] Create `ReleaseServiceSerialCommand` in `exports/commands/releaseServiceSerial.command.ts`
- [x] Create `ConsumeServiceConsumableCommand` in `exports/commands/consumeServiceConsumable.command.ts`
- [x] Create `CommitBookingSparesCommand` in `exports/commands/commitBookingSpares.command.ts`
- [x] Create `StagedSerialResultDto` in `exports/results/stagedSerialResult.dto.ts`
- [x] Create `StockServiceCommitResultDto` in `exports/results/stockServiceCommitResult.dto.ts`
- [x] Create `StockConsumableResultDto` in `exports/results/stockConsumableResult.dto.ts`
- [x] Add domain exceptions to `common/exceptions/stockOutgoing.exceptions.ts`:
  - `SerialNotHeldByTechnicianException`
  - `SerialAlreadyStagedForAnotherBookingException`
  - `SerialNotStagedForThisBookingException`
  - `InsufficientConsumableStockException`
  - `ItemMustBeSerializedException`
  - `ItemMustBeConsumableException`
  - `NoStagedSparesFoundForBookingException`
- [x] Register domain error codes and message constants in `stockOperationsMessages.ts`

### Phase 3: Spares Soft-Hold & Release (Shared Domain Engine)
- [x] Create `ServiceStockConsumptionEngine` in `common/services/serviceStockConsumption.engine.ts`
- [x] Implement `stageServiceSerial` on `ServiceStockConsumptionEngine`
- [x] Implement `releaseServiceSerial` on `ServiceStockConsumptionEngine`
- [x] Implement `getStagedSparesForBooking` on `ServiceStockConsumptionEngine`
- [x] Register and export `ServiceStockConsumptionEngine` in `StockOperationsCommonModule`

### Phase 4: Consumables Direct Commit
- [x] Implement `consumeServiceConsumable` on `ServiceStockConsumptionEngine`
- [x] Integrate with `FieldStaffStock.looseQuantityOnHand` / `looseQuantityConsumed`
- [x] Write immutable `InventoryMovement(OUTWARD__SERVICE_CONSUMABLE)`
- [x] Post double-entry financial journal (`OUTWARD__SERVICE_CONSUMABLE__EXPENSE`)

### Phase 5: Post-OTP Atomic Commit (`OUT-02`)
- [x] Implement `commitBookingSpares` on `ServiceStockConsumptionEngine`
- [x] Reconstruct ground-truth from DB (`serviceBookingId`, `status: IN_FIELD_CUSTODY`)
- [x] In atomic transaction:
  - Transition serials to `SerialStatus.INSTALLED` (`installedAt: now()`, `fieldStaffId: null`)
  - Decrement `FieldStaffStock.quantityOnHand`, increment `serviceQuantitySold`
  - Decrement batch stock if linked to `InventoryBatch`
  - Log `InventoryMovement(OUTWARD__SERVICE_SALE)` or `OUTWARD__CUSTOMER_WARRANTY_ISSUE`
  - Post financial journals (`OUTWARD__SERVICE_SALE__REVENUE_COGS` or warranty expense)

### Phase 6: Exported Facade
- [x] Wire methods into `InventoryIntegrationService` with optional `tx` injection delegating to `ServiceStockConsumptionEngine`

### Phase 7: REST Controllers & Dedicated Actor Services
- [x] Create `FieldStockServiceService` in `field/services/fieldStockService.service.ts`
- [x] Wire `FieldStockServiceController` (`@Controller('field/stock/service')`) to inject `FieldStockServiceService`
- [x] Remove `StaffStockOperationsModule` from `FieldStockOperationsModule` (restoring 100% actor isolation, zero sibling coupling)
- [x] Add staff workshop endpoints to `StockOutgoingController`:
  - `POST /staff/branches/:branchId/stock/outward/service-commit`
  - `GET /staff/branches/:branchId/stock/outward/bookings/:bookingId/staged-spares`
- [x] Add `StaffServiceCommitDto` and composite Swagger decorators (`ApiStaffServiceCommitDocs`, `ApiGetStaffStagedSparesDocs`)

### Phase 8: Financial Sub-Ledger Engine
- [x] Verify `StockFinancialPostingEngine` entries for service movements

### Phase 9: Adversarial Test Suite
- [x] Author comprehensive tests in `serviceStockConsumption.engine.spec.ts`
- [x] Author unit tests in `fieldStockService.service.spec.ts` and `fieldStockService.controller.spec.ts`
- [x] Update `stockOutgoing.controller.spec.ts` with workshop endpoints
- [x] 31 test suites, 330 tests pass

### Phase 10: Zero-Debt Audit & Sync
- [x] Run `npm test -- src/modules/stockOperations` (330 passed)
- [x] Run `npm run build` in `karigarji_server` (0 errors)
- [x] Synchronize `karigarji_server/.agents/milestones.md`
