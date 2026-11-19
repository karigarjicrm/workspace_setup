# Milestone Specification: OUT-02 Service Booking Spares & Consumables Consumption Engine

- **Milestone ID**: `MLS-BE-OUT02`
- **Domain**: Branch Stock Operations, Service Execution & Cross-Domain Inventory Export
- **Subsystem**: Backend (`karigarji_server`)
- **Status**: **IN_PROGRESS**
- **Priority**: Critical (P0)
- **Initiation Date**: 2026-11-10

---

## 1. Executive Summary & Architectural Invariants

`OUT-02` governs all physical inventory consumption, serial status transitions, and double-entry financial sub-ledger postings that occur during the execution of customer service bookings and appliance repairs.

Unlike `OUT-01` (a cashier-driven OTC counter walk-in sale from branch warehouse shelves), `OUT-02` operates in field and workshop servicing environments characterized by:
1. **Van Custody Isolation**: Physical spares originate from the assigned technician's van custody ([`FieldStaffStock`](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_server/prisma/schema.prisma#L1018)) rather than branch warehouse stock. Workshop jobs also issue to the working technician first to eliminate unaccounted workshop floor shrinkage.
2. **Item-Type Divergence**:
   - **Serialized Spares (Compressors, Motors, PCBs, Valves)**: **Soft-Hold by default**. Scanned during diagnosis and held in technician custody linked to `serviceBookingId`. If the customer rejects the quote or a part is swapped, it is released back to van stock with **zero financial or tax ledger churn**.
   - **Consumables (Gas/Refrigerant, Brazing Flux, Tape, Acid Wash)**: **Direct Commit by default**. Physical materials cannot be un-used once brazed or charged; they immediately decrement pouch stock ([`looseQuantityOnHand`](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_server/prisma/schema.prisma#L1025) $\to$ `looseQuantityConsumed`) under `OUTWARD__SERVICE_CONSUMABLE`.
3. **Signal Trigger Pattern (Lean Payload)**: The completion endpoint does not accept client-dictated price snapshots or serial lists. Instead, the mobile client sends an intent signal (`{ bookingId }`), and the backend queries the verified ground-truth from the database.
4. **Decoupled Customer OTP Handshake**: Customer completion OTP is owned and verified by [`BookingModule`](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_server/prisma/schema.prisma#L521). Upon OTP verification, `BookingModule` invokes the exported `commitBookingSpares` facade in an atomic `$transaction`.
5. **Dynamic Pricing Governance**: Commercials (billable spare vs. bundled service vs. warranty replacement) are determined dynamically by Service Item Configurations stored on the active booking.

---

## 2. Complete API Catalog & Interface Contracts

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                              OUT-02 INTEGRATION ARCHITECTURE                           │
└──────────────────────────────────────────┬─────────────────────────────────────────────┘
                                           │
         ┌─────────────────────────────────┴─────────────────────────────────┐
         ▼                                                                   ▼
┌──────────────────────────────────────┐            ┌──────────────────────────────────────┐
│ TIER 1: FIELD MOBILE REST APIS       │            │ TIER 2: EXPORTED FACADE (IN-PROCESS) │
│ (Technician App during Job Card)     │            │ (BookingModule & Job Lifecycle)      │
├──────────────────────────────────────┤            ├──────────────────────────────────────┤
│ POST /field/stock/service/stage-spare│            │ stageServiceSerial(cmd, tx?)         │
│ POST /field/stock/service/release-sp.│            │ releaseServiceSerial(cmd, tx?)       │
│ GET  /field/stock/service/staged-sp. │            │ consumeServiceConsumable(cmd, tx?)   │
│ POST /field/stock/service/consume-con│            │ commitBookingSpares(cmd, tx?)        │
│ POST /field/stock/service/commit-sp. │            │ getStagedSparesForBooking(id, tx?)   │
└──────────────────────────────────────┘            └──────────────────────────────────────┘
```

### Tier 1: Field Mobile REST Endpoints (`FieldStockServiceController`)
- **Base Route**: `/field/stock/service`
- **Auth Guard**: Authenticated `FIELD_STAFF` (`@CurrentUser() user`)

1. **`POST /field/stock/service/stage-spare`**:
   - *Payload*: `{ bookingId: string, serialNumber: string }`
   - *Behavior*: Validates serial is in caller's van kit (`IN_FIELD_CUSTODY`, `fieldStaffId === user.id`) and has `serviceBookingId === null`. Sets `serial.serviceBookingId = bookingId`.
2. **`POST /field/stock/service/release-spare`**:
   - *Payload*: `{ bookingId: string, serialNumber: string, reason?: string }`
   - *Behavior*: Verifies serial is held by caller on this booking. Resets `serviceBookingId = null`, returning serial to available van pool.
3. **`GET /field/stock/service/bookings/:bookingId/staged-spares`**:
   - *Params*: `bookingId: string`
   - *Behavior*: Returns all active staged serials for mobile review screen before the customer OTP swipe.
4. **`POST /field/stock/service/consume-consumable`**:
   - *Payload*: `{ bookingId: string, variantBrandId: string, quantity: number, idempotencyKey: string, notes?: string }`
   - *Behavior*: Immediate direct commit from van pouch stock. Decrements `looseQuantityOnHand`, increments `looseQuantityConsumed`. Writes `OUTWARD__SERVICE_CONSUMABLE`.
5. **`POST /field/stock/service/commit-spares`**:
   - *Payload*: `{ bookingId: string }`
   - *Behavior*: Lean signal trigger. Reconstructs all staged serials from DB ground-truth, transitions to `INSTALLED`, decrements van `quantityOnHand`, increments `serviceQuantitySold`, writes `OUTWARD__SERVICE_SALE`, and posts financial journals.

### Tier 2: Programmatic Export Facade (`InventoryIntegrationService`)
- **Location**: `src/modules/stockOperations/exports/inventoryIntegration.service.ts`
- **Methods**:
  - `stageServiceSerial(command: StageServiceSerialCommand, tx?: Prisma.TransactionClient): Promise<StagedSerialResultDto>`
  - `releaseServiceSerial(command: ReleaseServiceSerialCommand, tx?: Prisma.TransactionClient): Promise<StagedSerialResultDto>`
  - `consumeServiceConsumable(command: ConsumeServiceConsumableCommand, tx?: Prisma.TransactionClient): Promise<StockConsumableResultDto>`
  - `commitBookingSpares(command: CommitBookingSparesCommand, tx?: Prisma.TransactionClient): Promise<StockServiceCommitResultDto>`
  - `getStagedSparesForBooking(bookingId: string, tx?: Prisma.TransactionClient): Promise<StagedSerialResultDto[]>`

### Tier 3: Staff Workshop REST Endpoints (`StockOutgoingController`)
- **Base Route**: `/staff/branches/:branchId/stock/outward`
- **Auth Guard**: Authenticated `OFFICE_STAFF` with `MOVEMENT_CREATE` permission
  - `POST /staff/branches/:branchId/stock/outward/service-commit`
  - `GET /staff/branches/:branchId/stock/outward/bookings/:bookingId/staged-spares`

---

## 3. Mathematical State Transitions & Sub-Ledger Balances

### A. Serialized Spares Commit (`commitBookingSpares`)
$$\text{post}.\text{FieldStaffStock}.\text{quantityOnHand} = \text{pre}.\text{FieldStaffStock}.\text{quantityOnHand} - \text{Quantity}$$
$$\text{post}.\text{FieldStaffStock}.\text{serviceQuantitySold} = \text{pre}.\text{FieldStaffStock}.\text{serviceQuantitySold} + \text{Quantity}$$
$$\text{post}.\text{InventoryItemSerial}.\text{status} = \text{SerialStatus}.\text{INSTALLED}$$
$$\text{post}.\text{InventoryItemSerial}.\text{installedAt} = \text{now}()$$
$$\text{post}.\text{InventoryItemSerial}.\text{fieldStaffId} = \text{null}$$

### B. Consumables Direct Commit (`consumeServiceConsumable`)
$$\text{post}.\text{FieldStaffStock}.\text{looseQuantityOnHand} = \text{pre}.\text{FieldStaffStock}.\text{looseQuantityOnHand} - \text{Quantity}$$
$$\text{post}.\text{FieldStaffStock}.\text{looseQuantityConsumed} = \text{pre}.\text{FieldStaffStock}.\text{looseQuantityConsumed} + \text{Quantity}$$
$$\text{Movement} = \text{InventoryMovementType}.\text{OUTWARD\_\_SERVICE\_CONSUMABLE}$$
$$\text{Journal} = \text{FinancialEntryType}.\text{OUTWARD\_\_SERVICE\_CONSUMABLE\_\_EXPENSE}$$

### C. Financial Journal Balancing Equation
$$\Delta\text{Valuation} + \Delta\text{Cost} = \Delta\text{Revenue}$$
- For billable spares: $\text{ValuationDelta} = -(\text{Qty} \times \text{LandedCost})$, $\text{CostDelta} = \text{Qty} \times \text{LandedCost}$, $\text{RevenueDelta} = \text{GrossRevenue}$, $\text{GstType} = \text{OUTPUT\_LIABILITY}$.
- For bundled service parts: $\text{RevenueDelta} = 0$, $\text{Journal} = \text{OUTWARD\_\_SERVICE\_CONSUMPTION\_\_EXPENSE}$.
- For warranty replacements: $\text{RevenueDelta} = 0$, $\text{Journal} = \text{OUTWARD\_\_CUSTOMER\_WARRANTY\_\_EXPENSE}$.

---

## 4. Adversarial Edge Cases Handled

1. **Custody Hijacking**: Technician A attempts to stage a serial currently in Technician B's van kit $\to$ rejected with `SerialNotHeldByTechnicianException` (HTTP 409).
2. **Double Booking Allocation**: Technician attempts to stage a serial already staged for another active booking $\to$ rejected with `SerialAlreadyStagedForAnotherBookingException` (HTTP 409).
3. **Pouch Consumable Underflow**: Technician attempts to log 500g gas when van pouch only has 200g $\to$ rejected with `InsufficientConsumableStockException` (HTTP 409).
4. **Mismatched Item Types**: Attempting to stage a consumable via serial endpoint $\to$ rejected with `ItemMustBeSerializedException` (HTTP 400).
5. **Zero-Ledger Swap/Denial**: Customer rejects quote after testing $\to$ `releaseServiceSerial` restores serial to van stock without writing any movement or financial record.
6. **Concurrent Cashier/Tech Conflict**: Atomic conditional updates (`WHERE status = IN_FIELD_CUSTODY`) prevent race conditions.
