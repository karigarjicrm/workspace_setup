# Milestone Steps: Service Stock Separation & Outward Service Sale Isolation

- **Milestone ID**: `MLS-BE-OUT02-ISOLATION`
- **Subsystem**: Backend (`karigarji_server`)
- **Status**: 🟢 **COMPLETED**
- **Date Initiated**: 2026-11-10
- **Date Completed**: 2026-11-10

---

## 1. Phase Progress Status Board

| Phase | Description | Scope & Target Files | Verification Criteria | Status |
| :--- | :--- | :--- | :--- | :---: |
| **Phase 1** | Architectural Planning & Alignment | `milestones/out-02-service-sale-isolation/milestone.md` | Flow diagrams, domain boundaries, zero aliases, approval | 🟢 `COMPLETED` |
| **Phase 2** | Serial Lifecycle Domain: Reservation Engine & Endpoints | `serialLifecycle.engine.ts`, `stockSerial.service.ts`, `serialLifecycle.controller.ts`, DTOs | Unit tests pass for reserve, release, getReserved; REST endpoints live | 🟢 `COMPLETED` |
| **Phase 3** | Direct Outward Movement: `executeServiceSale` on `StockOutgoingService` | `stockOutgoing.service.ts`, `stockOutgoing.controller.ts`, `executeServiceSale.dto.ts` | Van custody deduction, batch updates, serials $\to$ `INSTALLED`, journals; 0 aliases | 🟢 `COMPLETED` |
| **Phase 4** | Programmatic Facade Synchronization | `inventoryIntegration.service.ts` | Direct delegation: serials $\to$ `SerialLifecycleEngine`, outward $\to$ `StockOutgoingService` | 🟢 `COMPLETED` |
| **Phase 5** | Middle-Layer Decommissioning & Dead Code Removal | Delete `serviceStockConsumption.engine.*`, `staffServiceCommit.dto.ts`, clean modules | `StockOperationsCommonModule` pruned; 0 orphaned files | 🟢 `COMPLETED` |
| **Phase 6** | Unit & Invariant Test Suite Alignment | `stockSerial.service.spec.ts`, `stockOutgoing.service.spec.ts`, `inventoryIntegration.service.spec.ts` | 100% test pass rate across `stockOperations` | 🟢 `COMPLETED` |
| **Phase 7** | Zero-Debt Self-Audit & Strict TypeScript Verification | `npx tsc --noEmit`, `npm test`, `npm run build` | 0 TS errors, 0 test failures, clean Nest & Vite builds | 🟢 `COMPLETED` |

---

## 2. Active Resumption Checkpoint

- **Current Phase**: All Phases 1–7 Completed
- **In-Flight Step**: Milestone fully executed, verified, and audited with zero tech debt.
- **Next Immediate Action**: Hand back execution to user with complete audit report.

---

## 3. Itemized Implementation Checklist

### Phase 2: Serial Lifecycle Domain (Reservation & Custody Lock)
- [x] Implement `reserveSerial`, `releaseSerial`, and `getReservedSerials` directly in `SerialLifecycleEngine` and `StockSerialService`
- [x] Create clean DTOs in `staff/dto/serial/`:
  - `ReserveSerialDto` (`serialNumber`, `reservationToken`, `staffId?`, `notes?`)
  - `ReleaseSerialDto` (`serialNumber`, `reservationToken`, `staffId?`, `reason?`)
- [x] Add REST endpoints in `StockSerialController` (`serialLifecycle.controller.ts`):
  - `POST /staff/branches/:branchId/serials/reserve`
  - `POST /staff/branches/:branchId/serials/release`
  - `GET  /staff/branches/:branchId/serials/reserved/:reservationToken`
- [x] Decorate with OpenAPI Swagger documentation and envelope decorators.

### Phase 3: Outward Movement Domain (`OUT-02` Service Sale)
- [x] Integrate service sale execution directly into `StockOutgoingService`:
  - Atomically lookup ground-truth reserved serials by `referenceId`
  - Validate staff custody (`IN_FIELD_CUSTODY` + `fieldStaffId`) or branch stock
  - Transition serials to `SerialStatus.INSTALLED` (`installedReference = referenceId`, `reservationToken = null`, `installedAt = now()`)
  - Decrement technician on-hand custody (`FieldStaffStock.quantityOnHand`), increment `serviceQuantitySold`
  - Decrement batch lot balances (`InventoryBatch.quantityOnHand`)
  - Log `InventoryMovement(OUTWARD__SERVICE_SALE)` or `OUTWARD__CUSTOMER_WARRANTY_ISSUE`
  - Post double-entry financial journals (Revenue + Landed COGS, or Warranty Expense)
- [x] Clean up `StockOutgoingController`:
  - Remove all legacy aliases (`service-commit`, `workshop/commit`, `counter-sale`, `fulfill`)
  - Expose clean endpoint: `POST /staff/branches/:branchId/stock/outward/service-sale`
  - Expose clean endpoint: `POST /staff/branches/:branchId/stock/outward/direct-sale`

### Phase 4: Programmatic Export Facade (`InventoryIntegrationService`)
- [x] Update `InventoryIntegrationService` to eliminate intermediate engine:
  - `reserveSerial` $\to$ calls `serialLifecycleEngine.reserveSerial`
  - `releaseSerial` $\to$ calls `serialLifecycleEngine.releaseSerial`
  - `getReservedSerials` $\to$ calls `serialLifecycleEngine.getReservedSerials`
  - `consumeServiceSale` $\to$ calls `stockOutgoingService.executeServiceSale`
  - `consumeServiceConsumable` $\to$ calls `stockOutgoingService.consumeServiceConsumable`

### Phase 5: Decommissioning & Dead Code Removal
- [x] Delete `serviceStockConsumption.engine.ts` and `serviceStockConsumption.engine.spec.ts`
- [x] Delete `staffServiceCommit.dto.ts`
- [x] Remove `ServiceStockConsumptionEngine` from `StockOperationsCommonModule` providers and exports

### Phase 6 & 7: Test Suite & Zero-Debt Verification
- [x] Update unit tests in `stockSerial.service.spec.ts` / `serialLifecycle.controller.spec.ts`
- [x] Update unit tests in `stockOutgoing.service.spec.ts` / `stockOutgoing.controller.spec.ts`
- [x] Update unit tests in `inventoryIntegration.service.spec.ts`
- [x] Run `npx tsc --noEmit` to verify 0 TypeScript compiler warnings or errors
- [x] Run `npm test` across all 81 test suites to verify 100% pass rate (764 passing tests)
- [x] Run `npm run build` in `karigarji_server` (NestJS) and `karigarji_admin_panel` (Vite)
