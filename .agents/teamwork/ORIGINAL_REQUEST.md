# Original User Request

## Initial Request — 2026-10-16T06:49:21Z

This is a single self-contained fix; keep it small and focused.

Refactor branch-scoped physical stock, custody, movement transactions, serialization, and double-entry GST financial accounts from `src/modules/inventory/` to `src/modules/branches/` with direct operational routes.

Working directory: `/Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_server`
Integrity mode: development

## Requirements

### R1. Relocate Branch Operational Subsystems
Move physical stock, field technician custody, movement transaction ledger, serialization tracking, and branch double-entry GST financial accounting from `src/modules/inventory/` into `src/modules/branches/`. The `inventory` module must strictly retain central master data (Brands, Categories, Attribute Templates) and Catalog PIM (Items, Variants, Brand Mappings).

### R2. Direct Branch Operational Route Restructuring
Adopt direct REST sub-resource routing under `/staff/branches/:branchId/` for all branch operations (e.g. `/stock`, `/field-stock`, `/movements`, `/serials`, `/financial-account`, `/financial-journals`), eliminating the `/inventory` infix while maintaining PBAC permissions and transparent `'me'` parameter resolution for branch staff.

### R3. Unidirectional Architecture & Decoupled Domain Contracts
`src/modules/branches/` must import `InventoryCommonModule` for read-only catalog lookups. `src/modules/inventory/` must have zero reverse dependencies on `branches`, ensuring unidirectional layer flow and strict bounded context isolation.

## Acceptance Criteria

### Compilation & Type Safety
- [ ] `npm run build` compiles with 0 TypeScript or NestJS dependency injection errors.

### Test Coverage & Zero Regressions
- [ ] All existing and relocated unit test suites (policies, services, controllers) pass with 0 failures via `npm test`.

### API Contract & Documentation
- [ ] `npm run openapi:export` successfully exports the updated Swagger specification reflecting the new direct branch routes without orphaned endpoints.
