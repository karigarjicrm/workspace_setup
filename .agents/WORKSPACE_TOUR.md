# Karigarji Ecosystem: Complete Workspace Tour Guide & Architecture Map

> **For AI Agents & Developers**: This document provides the holistic architectural blueprint of the entire Karigarji multi-repository ecosystem. Read this guide to understand how all client applications, backend services, domain models, and security contracts interlock.

---

## 1. Executive Vision & Ecosystem Purpose

**Karigarji** is an enterprise-grade, tech-enabled on-demand **Home Appliances & Consumer Electronics Repair, Maintenance, and Logistics Platform** operating across India.

The platform organizes the fragmented home services sector by integrating:
1. **Customer Doorstep Discovery**: Transparent upfront pricing, warranty enforcement, and real-time technician dispatch.
2. **Multi-Branch Operations & Workshop Network**: Central warehouses, FOCO/company-owned regional repair workshops, and local spare parts depots.
3. **Dual-Force Field Operations**: Certified on-site technicians paired with rapid-response logistics runners delivering genuine parts to customer premises.
4. **Automated Tax Compliance & Logistics**: Closed-loop spare parts inventory tracking, 18% GST billing (`SAC 9987`), and presigned media verification.

### Core Appliance Domains
- **Air Conditioners (AC)**: Jet-pump servicing, gas leak testing & R32/R410A charging, PCB repairs, compressor replacement.
- **Washing Machines**: Drum bearing overhauls, drain pump replacements, inlet valve repairs, motor rewinding.
- **Refrigerators**: Defrost timers/bimetal replacement, PTC relay/capacitor repairs, sealed system gas charging.
- **Televisions**: Backlight array replacement, panel bonding/COF repair, motherboard power repairs, wall mounting.

---

## 2. Global Architecture & The 4 Active Repositories

The Karigarji meta-workspace houses 4 tightly coupled, production-grade repositories:

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                 KARIGARJI ECOSYSTEM                                     │
├───────────────────────────────┬──────────────────────────────┬──────────────────────────┤
│       CUSTOMER CLIENTS        │     OPERATIONS SURFACES      │      FIELD CLIENTS       │
│  - karigarji_customer         │  - karigarji_admin_panel     │  - karigarji_technician  │
│    (Expo SDK 57 Mobile App)   │    (React 19 / Vite Web)     │    (Expo SDK 57 Mobile)  │
└───────────────────────────────┴──────────────────────────────┴──────────────────────────┘
                                               │
                                 JSON REST APIs & Presigned URLs
                                               ▼
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                AUTHORITATIVE CORE HUB                                   │
│                                  karigarji_server                                       │
│          NestJS 11+ │ PostgreSQL (Prisma) │ Redis 7 │ Cloudflare R2 / AWS S3            │
│       4-Actor Architecture: Public (CDN) │ Customer │ Office Staff │ Field Staff        │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Deep-Dive: Repository Portfolios & Technologies

### 3.1 Authoritative Backend Hub (`karigarji_server`)
- **Role**: Single source of truth for business logic, persistence, transactions, and event orchestrations.
- **Stack**: NestJS 11+, TypeScript 5.7+, Prisma ORM, PostgreSQL, Redis 7 (sessions & OTP rate limiting), Cloudflare R2 / AWS S3.
- **The 4-Actor Routing Boundaries**:
  - `/api/v1/public/*`: Unauthenticated, high-speed catalog discovery with CDN media caching.
  - `/api/v1/customer/*`: Phone OTP authentication, customer appliances garage, booking requests, and digital invoices.
  - `/api/v1/staff/*`: Corporate credentials, RBAC/PBAC permission guards, service catalog studio, inventory stock ledger, and branch dispatch.
  - `/api/v1/field/*`: Field staff authentication, job dispatch queues, diagnostic checklists, van parts requisition, and proof-of-work uploads.
- **7-Tier Backend Hierarchy & Layer Isolation**:
  - `Controller`: Thin HTTP routing (~80–110 lines), composite route decorators (`@ApiSuccessResponseEnvelope`).
  - `Domain Service`: Persistence orchestration, transaction management, and cross-domain events (`EventEmitter2`).
  - `Pure Logic` (`*.logic.ts`): 100% pure algorithms and pricing formulas with **zero DB I/O**.
  - `Domain Policy` (`*.policy.ts`): State machine guards and action eligibility checks.
  - `Repository / Prisma`: Relational persistence using 64-bit Snowflake IDs (`SnowflakeService`) and soft deletes (`deletedAt`).
- **Database Safety**: **Zero automated migrations**. All database-altering commands require explicit pre-command review and manual confirmation.

### 3.2 Operations Command Center (`karigarji_admin_panel`)
- **Role**: Unified enterprise desktop dashboard for corporate headquarters, branch managers, and dispatch coordinators.
- **Stack**: React 19, Vite, React Router v7, TanStack Query v5, Radix UI headless primitives, Tailwind CSS v4, Lucide React, Sonner.
- **Key Architectural Paradigms**:
  - **Single Scroll Authority**: Viewport scrolling is owned strictly by `PageContainer` or dedicated data tables; nested scrollbars are forbidden.
  - **3-Tier Domain Slicing**: Modules reside in `src/app/<domain>/<subdomain>/` with clean `api/`, `schemas/`, `logic/`, and `pages/`.
  - **Service Catalog Studio (`/catalog`)**: Decoupled interactive hierarchy workspace with split-screen service and variant editors featuring real-time customer preview canvases.
  - **Multi-Branch Tenancy**: `TeamSwitcher` allowing workshop managers to operate within their branch context while HQ maintains global oversight.
  - **Pure In-Memory Access Tokens**: Access tokens live strictly in-memory; refresh tokens are held in browser `HttpOnly` cookies.

### 3.3 Customer Mobile App (`karigarji_customer`)
- **Role**: Consumer touchpoint for service discovery, calendar scheduling, real-time tracking, and GST invoicing.
- **Stack**: React Native `0.86.3`, React `19.2.3`, Expo SDK 57 (~57.0.25), Expo Router v57, TanStack Query v5, Zustand, `expo-secure-store`, `@expo/ui`.
- **Key Architectural Paradigms**:
  - **Continuous Native Generation (CNG)**: Native `ios/` and `android/` folders are generated via `app.json` plugins. Zero manual native edits.
  - **Dual-Platform Native Icons**: SF Symbols on iOS (0 KB), Material 3 XML on Android (~1 KB) via `@expo/ui` `Icon` consuming `AppIcons`.
  - **Hardware-Backed Tokens**: Refresh tokens stored in hardware keystores (`expo-secure-store`); access tokens in-memory.
  - **Interactive Drill-Down**: Category $\to$ Appliance $\to$ Service $\to$ Variant tier.

### 3.4 Field Staff Technician App (`karigarji_technician`)
- **Role**: On-ground execution tool for certified service engineers and mechanics.
- **Stack**: React Native `0.86.3`, React `19.2.3`, Expo SDK 57, Expo Router v57, TanStack Query v5, Zustand, `expo-secure-store`, `@expo/ui`.
- **Key Architectural Paradigms**:
  - **Strict Linear Job Lifecycle**: Discrete 6-stage finite state machine (`ASSIGNED` $\to$ `EN_ROUTE` $\to$ `ARRIVED` $\to$ `INSPECTING` $\to$ `WORK_IN_PROGRESS` $\to$ `COMPLETED`). Stage skipping and backward steps are strictly forbidden.
  - **Sticky Action Bars**: Operational action buttons float outside the scroll container with dynamic Android edge-to-edge padding (`Math.max(insets.bottom, Spacing.four)`).
  - **Zero OOM Camera Downscaling**: Mandatory on-device compression (`expo-image-manipulator` to 1280px / 80% JPEG) before upload to protect budget 3–4GB RAM Android devices.
  - **Biometric Step-Up Security**: Mandatory FaceID / BiometricPrompt (`expo-local-authentication`) for bank account and payout setting modifications.
  - **Offline Resilience**: Network connectivity detection with queued mutations when signal is lost on-site.

---

## 4. Cross-Repository Contracts & System Protocols

All 4 repositories synchronize seamlessly by adhering to universal contracts:

### A. Standardized API Response Envelope
The backend wraps all 2xx HTTP payloads into a predictable envelope:
```typescript
interface ApiResponse<T> {
  success: boolean;
  statusCode: number;
  message: string;
  data: T;
  meta?: ResponseMeta; // page, limit, total, totalPages
}
```
All frontend and mobile clients (`apiClient.ts`) unbox this envelope automatically into:
```typescript
export interface ResponseData<T> {
  body: T;              // Pure unboxed domain entity (data.data ?? data)
  message: string;
  meta?: ResponseMeta;
  status: number;
  ok: boolean;
}
```
Domain services and queries receive **pure domain data**, never artificial HTTP containers.

### B. Dual-Token Authentication Protocol
- **Web Admin (`karigarji_admin_panel`)**: Short-lived `accessToken` held in-memory; long-lived `refreshToken` stored in browser `HttpOnly` cookie scoped to `/api/v1/staff/auth`.
- **Mobile Apps (`karigarji_customer`, `karigarji_technician`)**: Short-lived `accessToken` held in-memory; `refreshToken` stored in hardware-backed `expo-secure-store`.
- **Mutex Replay Queue**: On concurrent 401 failures, an atomic `isRefreshing` mutex serializes the refresh call while other requests pause in `failedQueue` and replay automatically upon token refresh.

### C. Query Key Factory Pattern (`createQueryKeyFactory`)
Every client domain defines an authoritative factory:
```typescript
export const jobKeys = createQueryKeyFactory('technician-jobs', {
  list: (filters?: JobFilters) => [{ ...filters }],
  detail: (id: string) => [id],
});
```
This guarantees deterministic cache scoping and eliminates magic strings in cache invalidation.

### D. Zero Free-Text Magic Strings
All validation errors, user-facing notifications, and domain status codes consume strongly-typed constants (`*Messages.ts`, `Colors.*`, `AppIcons.*`).

---

## 5. Universal Agent Rules & Development Lifecycle

When operating anywhere in this workspace, every AI agent must strictly follow the rules codified in [`/AGENTS.md`](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/AGENTS.md):

1. **Pragmatic Senior Staff Engineering (Zero Sycophancy)**:
   - Critically evaluate trade-offs; respectfully push back against anti-patterns, scope creep, or architectural shortcuts.
2. **5-Step Planning & Mandatory Step Confirmation**:
   - Clarify Objective $\to$ Ground Truth Check $\to$ Align 1 Key Decision at a Time $\to$ Itemized Phase Plan $\to$ **Obtain Explicit User Confirmation Before Writing Any Code**.
3. **Continuous Task Tracking (`tasks/<task-name>/`)**:
   - Every primary task maintains `steps.md` (Phase Progress Board & Active Resumption Checkpoint) and `logs.md` (Decision and timestamp logs).
4. **Project-Specific Bug Registries**:
   - Always check `<project>/.agents/bugRegistry.md` first before debugging to prevent repeating known bugs.
5. **Mandatory Self-Audit**:
   - Verify TypeScript compilation (`npx tsc --noEmit`) and unit tests (`npm test`) with 0 errors before any task completion handoff.
