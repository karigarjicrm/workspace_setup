# Karigarji Platform Master Project Context

## 1. Executive Summary & Vision

**Karigarji** is an enterprise-grade, tech-enabled on-demand **Home Appliances & Consumer Electronics Repair, Maintenance, and Logistics Platform**.

The platform organizes the fragmented, unstandardized home services industry in India by unifying customer doorstep service discovery, certified technician field dispatch, rapid-response spare parts logistics, multi-branch physical workshop operations, and automated GST billing into a cohesive digital ecosystem.

### Core Domain Taxonomy
Karigarji focuses on high-frequency, mission-critical household appliances and electronics:
* **Air Conditioners (AC)**: Split ACs, Window ACs, Inverter units (Jet-pump deep servicing, gas leak testing & R32/R410A charging, PCB repairs, compressor replacement).
* **Washing Machines**: Front Load, Top Load, Semi-Automatic (Drum bearing overhauls, drain pump replacements, inlet valve repairs, motor rewinding).
* **Refrigerators**: Single Door, Double Door, Side-by-Side / Multi-Door (Defrost timer/bimetal replacement, PTC relay/capacitor repairs, sealed system gas charging).
* **Televisions**: LED, OLED, QLED, Smart TVs (Backlight array replacement, panel bonding/COF repair, motherboard power supply repairs, wall mounting).

---

## 2. Problem Statement & Market Friction

The traditional home appliance servicing industry suffers from severe structural inefficiencies across all three stakeholders:

### 2.1 Customer Pain Points
1. **Pricing Opacity & Bait-and-Switch Tactics**: Local unorganized handymen frequently quote low initial inspection rates, only to inflate repair charges with arbitrary costs once the appliance is dismantled.
2. **Counterfeit & Unreliable Spare Parts**: Customers have zero visibility into whether installed capacitors, pumps, relays, or refrigerants are genuine, refurbished, or low-grade clones.
3. **No Warranty Enforcement or Post-Service Accountability**: Once an independent technician leaves, post-service defects or recurring failures are ignored, leaving the customer stranded.
4. **Disrupted Daily Schedules**: Broad, unreliable time windows ("technician will come sometime today") without real-time tracking or advance notice.

### 2.2 Field Technician & Partner Pain Points
1. **Inefficient Sourcing & Mid-Job Travel**: Technicians often diagnose an issue on-site but must abandon the customer premises to hunt for specific spare parts at distant wholesale markets, wasting hours in traffic.
2. **Lack of Specialized Workshop Tools On-Site**: Complex diagnostics (such as microscopic TV panel bonding or compressor degassing) cannot be safely performed in a living room without workshop bench equipment.
3. **Delayed & Disputed Payouts**: Manual paper receipts lead to discrepancies, cash leakage, and friction during payout reconciliation.

### 2.3 Workshop & Enterprise Operations Pain Points
1. **Inventory Leakage & Unaccounted Stock**: Spare parts (especially valuable copper coils, compressors, and refrigerants) frequently go missing or get diverted without rigorous ledger tracking.
2. **Multi-Branch Operational Chaos**: Regional franchises, company-owned workshops, and central warehouses struggle to maintain uniform quality standards, pricing models, and SLA compliance.
3. **Tax Compliance Deficits**: Unregistered mechanics rarely issue GST-compliant invoices (`SAC 9987`), preventing commercial and residential customers from claiming tax credits and warranties.

---

## 3. The Karigarji Solution Model

Karigarji bridges these gaps through a vertically integrated, hybrid doorstep-and-workshop operational model:

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                                 KARIGARJI ECOSYSTEM                                     │
├───────────────────────────────┬──────────────────────────────┬──────────────────────────┤
│       CUSTOMER SURFACES       │     OPERATIONS SURFACES      │      FIELD SURFACES      │
│  - Customer Mobile App (Expo) │  - Admin Operations Panel    │  - Field Technician App  │
│  - Public Catalog Web Portal  │    (React 19 / Vite Web)     │  - Logistics Runner App  │
└───────────────────────────────┴──────────────────────────────┴──────────────────────────┘
                                               │
                                       RESTful JSON APIs
                                               ▼
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                       AUTHORITATIVE CORE HUB: karigarji_server                          │
│          NestJS 11+ │ PostgreSQL (Prisma) │ Redis 7 │ Cloudflare R2 / AWS S3            │
│       4-Actor Architecture: Public (CDN) │ Customer │ Office Staff │ Field Staff        │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

### Key Solution Pillars
* **Upfront Transparent Rate Cards**:
  Every service adheres to standardized pricing structures (`FIXED`, `STARTING_FROM`, or `TIERED` by appliance capacity/tonnage/technology). Inspection visiting fees are transparently credited against the total bill when a customer approves a major repair.
* **Closed-Loop Spare Parts Inventory Master (`InventoryItem`)**:
  All parts (capacitors, relays, drain pumps, valves, gas cylinders) and professional tools (high-pressure jet pumps, vacuum pumps) are cataloged with distinct SKUs, standard costs, billable rate card prices, and compatibility mappings to specific appliance types.
* **Dual-Tier Field Force: Certified Technicians + Logistics Runners**:
  Instead of forcing certified technicians to leave job sites to buy parts, specialized **Logistics Runners** deliver required components directly to the customer doorstep or transport heavy defective assemblies (e.g., TV panels, PCB motherboards) to the nearest branch workshop for clean bench repair.
* **Multi-Branch Physical Node Network**:
  Operations are structured hierarchically into `HEADQUARTERS`, `WORKSHOP` (FOCO or company-owned regional repair hubs), `WAREHOUSE` (central parts distribution nodes), and `STORE` (local spare depots).
* **Digital Proof-of-Work & Media Verification**:
  Pre-repair diagnostic photos/videos and post-repair operational proof are captured on the mobile app, uploaded directly to cloud storage via presigned URLs with zero server egress, and verified before an invoice is finalized.

---

## 4. Platform Ecosystem & Application Portfolios

The Karigarji ecosystem consists of **5 specialized client applications** powered by a single authoritative backend engine:

### 4.1 Authoritative Core Backend (`karigarji_server`)
* **Role**: Single source of truth for all business logic, persistence, transactions, and event orchestrations.
* **Stack**: NestJS 11+, TypeScript 5.7+, Prisma ORM, PostgreSQL, Redis (sessions & OTP rate limiting), Cloudflare R2 / AWS S3 (media storage).
* **Actor-First Architectural Segmentation**:
  * `public/`: High-performance, unauthenticated guest catalog browsing (`/api/v1/public/catalog/*`).
  * `customer/`: SMS OTP authentication, address book, appliance profile management, booking submissions, and digital invoice retrieval.
  * `staff/`: Corporate credentials login, session rotation, RBAC/PBAC permission governance, service catalog configuration, inventory management, workshop administration, and dispatch control.
  * `field/`: Mobile credentials, job queue retrieval, diagnostic reporting, spare parts requisition, and proof-of-work asset submission.
  * `common/`: Pure mathematical calculators (`ServicePricingLogic`, `BookingRuleResolverLogic`), shared DTOs, and global filters/interceptors.

### 4.2 Customer Mobile App (`karigarji_customer`)
* **Role**: Primary consumer touchpoint for discovery, scheduling, real-time tracking, and payment.
* **Stack**: React Native 0.86+, React 19, Expo SDK 57 (Continuous Native Generation), Expo Router v57, TanStack Query v5, Zustand, `expo-secure-store`.
* **Core Features**:
  * **Phone OTP Onboarding**: Fast, passwordless phone number sign-in with Redis-backed rate limiting and automated profile provisioning.
  * **Interactive Service Catalog**: Hierarchical drill-down (`Device Category` → `Appliance Type` → `Service Item` → `Variant Tier`), complete with detailed inclusions, exclusions, and warranty terms.
  * **Dynamic Slot Booking & Address Manager**: Geocoded address selection, intelligent service buffer calculation, and calendar slot selection.
  * **Live Job Lifecycle Tracking**: Visual timeline tracking (*Dispatched → Arrived → Inspection In Progress → Quote Approved → Work In Progress → Quality Check → Completed*).
  * **My Appliances Digital Garage**: Save owned appliances (brand, model, purchase year, serial number, warranty status, and past service history).
  * **Digital Invoice & Secure Payments**: Detailed breakdown of labor charges, parts consumed, and applicable GST (`SAC 9987`), with instant digital receipts.

### 4.3 Customer Public Web Portal (Guest Catalog & Web Booking)
* **Role**: Responsive web client for organic search discovery, social media landing pages, and guest checkout.
* **Features**:
  * **SEO-Optimized Service Pages**: Server-side or statically rendered appliance repair pages targeting local intent (e.g., "Split AC Gas Charging Noida").
  * **Unauthenticated Catalog Exploration**: High-speed browsing consuming `@Public()` catalog endpoints with direct CDN media delivery.
  * **Seamless Web-to-App Handshake**: Quick guest booking flow that transitions seamlessly into SMS confirmation and app deep links.

### 4.4 Admin Operations Command Center (`karigarji_admin_panel`)
* **Role**: Unified command center for corporate headquarters, branch managers, dispatch coordinators, and inventory supervisors.
* **Stack**: React 19, Vite, React Router v7, TanStack Query v5, Radix UI primitives, Tailwind CSS v4, Lucide React, Sonner notifications.
* **Core Modules**:
  * **Multi-Workshop Tenancy**: `TeamSwitcher` allowing workshop managers to operate within their assigned branch while headquarters executives maintain global oversight across all cities.
  * **Bookings & Dispatch Board**: Real-time dispatch console matching customer bookings with available certified technicians based on appliance capability matrix (`FieldStaffCapability`) and geo-proximity.
  * **Service Catalog Studio (`/catalog`)**: Decoupled multi-page studio architecture featuring an interactive hierarchy workspace (`/catalog`) alongside dedicated full-page split-screen editors for services (`/catalog/services/new`, `/catalog/services/:id/edit`) and variants (`/catalog/services/:id/variants/new`, `/catalog/services/:id/variants/:id/edit`) with real-time customer live preview canvases.
  * **Inventory Master & Stock Ledger**: Spare parts catalog (`InventoryItem`), office stock levels, warehouse-to-workshop transfers, and technician van-stock replenishment.
  * **Accounts & Staff Governance (`/accounts/staff`, `/roles`)**: Granular RBAC and PBAC management, corporate staff provisioning, and permission assignment.
  * **Billing, Tax & Financial Reporting**: Automated GST calculation, commercial credit notes, revenue analytics, and technician payout logs.

### 4.5 Field Technician Mobile App (`karigarji_field_tech`)
* **Role**: On-ground execution tool for certified service engineers.
* **Core Features**:
  * **Daily Dispatch Queue**: Push notifications for newly assigned jobs with customer location, reported fault description, and appliance history.
  * **Standardized Diagnostic Checklist**: Interactive diagnostic questionnaire per appliance type (e.g., checking AC compressor running current, suction pressure, and coil temperature).
  * **On-Site Spare Parts Requisition**: Add required spare parts directly from the inventory rate card, instantly sending dispatch requests to logistics runners.
  * **Digital Proof-of-Work Capture**: Mandatory pre-repair and post-repair photos/videos uploaded directly via presigned URLs.
  * **Job Sign-Off & Cash/UPI Collection**: Customer digital signature capture and payment confirmation.

### 4.6 Logistics Runner Mobile App (`karigarji_field_runner`)
* **Role**: Rapid-response logistics client for spare parts runners and workshop couriers.
* **Core Features**:
  * **Urgent Parts Delivery Tickets**: Real-time routing from branch warehouse to technician on-site locations for requested spare parts.
  * **Defective Unit Pickup & Workshop Transport**: Secure chain-of-custody tracking (QR/barcode scanning) for appliances or heavy sub-assemblies (e.g., inverter AC motherboards or TV display panels) picked up for workshop bench repair.
  * **Workshop Return Dispatch**: Delivering repaired, QA-certified assemblies back to customer premises for final installation.
  * *(Roadmap Note: Core authentication and basic delivery tickets prioritized first; full barcode scanning and multi-leg route optimization to follow).*

---

## 5. End-to-End Service Lifecycle Workflow

```
[1. CUSTOMER DISCOVERY & BOOKING]
    Customer selects Appliance (e.g. Split AC 1.5 Ton) & Service (e.g. Jet-Pump Deep Clean)
    System computes advance notice window, slot duration, and booking buffer
    Customer selects address, time slot, and confirms booking
            │
            ▼
[2. DISPATCH & ALLOCATION]
    Admin/Auto-Dispatch matches booking with certified Field Technician (FieldStaffCapability)
    Technician receives job notification with route navigation and reported symptoms
            │
            ▼
[3. ARRIVAL & DIAGNOSTIC INSPECTION]
    Technician arrives at customer doorstep and initiates diagnostic checklist
    Pre-repair condition photos/videos uploaded via presigned URL to Cloudflare R2 / S3
    If additional repairs/parts needed, technician generates digital estimate from rate card
            │
            ▼
[4. ESTIMATE APPROVAL & PARTS LOGISTICS]
    Customer reviews and approves revised estimate on mobile app
    If parts are needed, Logistics Runner receives dispatch ticket from nearest warehouse
    Runner delivers genuine parts directly to the technician on-site
            │
            ▼
[5. REPAIR EXECUTION & BENCH WORKSHOP]
    Minor repairs: Technician executes repair on-site using supplied genuine parts
    Major repairs (PCB/Panel): Runner transports component to Branch Workshop for bench fix
    Post-repair testing executed; operational proof media uploaded
            │
            ▼
[6. INVOICING, PAYMENT & WARRANTY]
    System generates finalized GST-compliant invoice (Labor + Parts Consumed)
    Diagnostic fee is credited against final bill if repair was performed
    Customer pays via UPI/Card/Cash; digital receipt issued
    Automated 30-day workmanship warranty period initiated in system
```

---

## 6. Technology Stack & Architectural Standards Matrix

| Layer / Concern | Technology Selection | Architectural Standard / Principle |
| :--- | :--- | :--- |
| **Backend Framework** | NestJS 11+ (Node.js, TypeScript 5.7+) | Modular Domain Architecture (`src/modules/*`), Thin Controllers (~80–110 LOC), 3-Tier Services (`.service`, `.logic`, `.policy`). |
| **Database & ORM** | PostgreSQL (Prisma ORM) | Relational integrity, soft deletes (`deletedAt`), strict zero automated migrations in production, pragmatic repositories. |
| **Identity & IDs** | 64-bit Snowflake IDs (`SnowflakeService`) | Time-sortable, 64-bit integer numbers for zero collision and high indexing performance. |
| **Cache & Real-Time** | Redis 7+ (ioredis / RedisService) | Dedicated DB indices for Auth Sessions, SMS OTP rate limiting, and in-memory fallbacks for resilient unit testing. |
| **Media Storage** | Cloudflare R2 / AWS S3 | Direct presigned upload issuance (`PUT`), signed temporary viewing (`GET`), automated orphan asset reaper (`MediaReaperService`). |
| **Admin Web Frontend** | React 19, Vite, React Router v7 | URL-first state, TanStack Query v5 with centralized Query Key Factories, Radix UI headless primitives, Tailwind CSS v4. |
| **Customer Mobile** | React Native 0.86+, Expo SDK 57 | Expo Router v57 file-based routing, Continuous Native Generation (CNG), `expo-secure-store` hardware keychain tokens, `@expo/ui`. |
| **Authentication** | Dual-Token (Access JWT + HttpOnly Cookie / SecureStore) | Default-deny route guards (`APP_GUARD`), `@Public()` exceptions, capability-driven `@RequirePermissions()`, silent rotation mutex. |
| **Tax & Invoicing** | Indian GST Standards (`SAC 9987`) | Standard 18% GST calculation, itemized labor vs parts billing, and verified tax invoices. |

---

## 7. Multi-Repository Workspace Structure

This Git repository serves as the **Meta-Orchestrator** housing the interconnected Karigarji services:

```text
karigarji_workspace/                        # Meta-Orchestrator Root
├── projectContext.md                       # This Master Authoritative Document
├── AGENTS.md                               # Authoritative operating principles for AI agents
├── README.md                               # Workspace orchestration guide
├── .agents/
│   └── rules/
│       └── rulesRedirect.md                # Global rules router for AI agents
├── tasks/                                  # Ephemeral runtime task tracking (Git-ignored)
│
├── karigarji_server/                       # Backend Repository (NestJS API Hub)
│   ├── src/modules/                        # auth, accounts, rbac, media, serviceCatalog
│   ├── prisma/                             # schema.prisma, migrations, seed.ts
│   └── .agents/rules/                      # Backend architectural & domain rules
│
├── karigarji_admin_panel/                  # Web Frontend Repository (Operations Command Center)
│   ├── src/app/                            # dashboard, accounts, roles, catalog, bookings
│   └── .agents/rules/                      # Frontend architecture, state & UI rules
│
└── karigarji_customer/                     # Mobile Repository (Customer Expo App)
    ├── src/app/                            # (auth), (tabs), booking, tracking
    └── .agents/rules/                      # Mobile rules, React Native & Expo standards
```

---

## 8. Development & Operational Principles

1. **Zero Free-Text Strings**: User-facing messages, validation errors, and domain codes consume strongly-typed constants (`*Messages.ts`).
2. **Domain Boundaries & Zero Cross-Domain Bleed**: Each module maintains strict encapsulation. Modules communicate strictly via exported contracts or domain events (`EventEmitter2`).
3. **Database Safety First**: Zero automated migrations. All database-touching scripts require explicit pre-command review and user confirmation.
4. **Environment Isolation**: Strict zero access to real secrets. Configuration is documented and managed exclusively via `.env.example` templates and secure runtime injection.
5. **Continuous Documentation Synchronization**: Authoritative context files, rules, and task trackers are kept updated in lockstep with all code modifications.
