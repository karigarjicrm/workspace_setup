# Milestone Logs: Service Stock Separation & Outward Service Sale Isolation

- **Milestone ID**: `MLS-BE-OUT02-ISOLATION`
- **Subsystem**: Backend (`karigarji_server`)
- **Date Initiated**: 2026-11-10

---

## Decision Log

- **2026-11-10 16:32**: Architectural Repositioning & De-layering Alignment:
  - User highlighted the fundamental conceptual distinction between **Serial Lifecycle** and **Inventory Movement**:
    1. `reserveSerial`, `releaseSerial`, and `getReservedSerials` are NOT movements. They do not alter inventory valuation, do not transfer physical custody, and do not create movement ledger rows. They are pure serial metadata and custody allocation state, belonging naturally to **Serial Lifecycle** (`SerialLifecycleEngine` / `StockSerialService`).
    2. Only `outward/service-sale` is an inventory movement API (`OUTWARD__SERVICE_SALE`).
    3. The intermediate `ServiceStockConsumptionEngine` was identified as an unnecessary abstraction layer and is scheduled for decommissioning.
    4. `InventoryIntegrationService` will directly export both domains:
       - Serial soft-allocation directly delegating to `SerialLifecycleEngine`.
       - Physical outward sales directly delegating to `StockOutgoingService`.
    5. Greenfield Zero-Legacy Policy: Remove all backward-compatibility route aliases (`service-commit`, `workshop/commit`, etc.) in favor of clean 1:1 REST endpoints.
  - Created dedicated milestone folder `milestones/out-02-service-sale-isolation/` with `milestone.md`, `steps.md`, and `logs.md`.
  - Ready to present detailed architecture and execution plan to user for explicit confirmation before making code changes.

- **2026-11-10 16:53**: Implementation Execution & Self-Audit:
  - User confirmed: "yes proceed now."
  - **Phase 2 (Serial Lifecycle Domain)**:
    - Implemented `reserveSerial`, `releaseSerial`, and `getReservedSerials` directly inside `SerialLifecycleEngine`.
    - Added corresponding methods to `StockSerialService`.
    - Added DTOs: `ReserveSerialDto` and `ReleaseSerialDto`.
    - Added REST endpoints to `StockSerialController` (`serialLifecycle.controller.ts`):
      - `POST /staff/branches/:branchId/serials/reserve`
      - `POST /staff/branches/:branchId/serials/release`
      - `GET  /staff/branches/:branchId/serials/reserved/:reservationToken`
    - Documented with Swagger OpenAPI decorators (`serialDocs.decorator.ts`).
  - **Phase 3 (Outward Movement Domain)**:
    - Embedded `executeServiceSale` and `consumeServiceConsumable` directly inside `StockOutgoingService`.
    - Cleaned up `StockOutgoingController`: single explicit routes `POST direct-sale` and `POST service-sale`; removed all route aliases.
    - Embedded `StaffSparePricingOverrideDto` directly into `executeServiceSale.dto.ts`.
  - **Phase 4 (Programmatic Export Facade)**:
    - Updated `InventoryIntegrationService` to eliminate intermediate layers: delegates reservations directly to `SerialLifecycleEngine` and service sales/consumables directly to `StockOutgoingService`.
  - **Phase 5 (Decommissioning)**:
    - Deleted `serviceStockConsumption.engine.ts`, `serviceStockConsumption.engine.spec.ts`, and `staffServiceCommit.dto.ts`.
    - Unregistered `ServiceStockConsumptionEngine` from `StockOperationsCommonModule`.
  - **Phase 6 & 7 (Verification & Self-Audit)**:
    - Updated unit test suites across `stockSerial.service.spec.ts`, `serialLifecycle.controller.spec.ts`, `serialLifecycle.engine.spec.ts`, `stockOutgoing.service.spec.ts`, `stockOutgoing.controller.spec.ts`, and `inventoryIntegration.service.spec.ts`.
    - `npx tsc --noEmit` exited with 0 errors across `karigarji_server`.
    - `npm test` passed 81/81 test suites (764/764 tests passed).
    - `npm run build` passed in both `karigarji_server` (NestJS) and `karigarji_admin_panel` (Vite, 10.17s).
    - Status updated to `COMPLETED`. Zero technical debt.
