# Project Context: Karigarji Admin Panel

## 1. Project Goal & Overview
The **Karigarji Admin Panel** is the primary management dashboard used by service coordinators and administrators. It connects to the NestJS CMS backend and displays:
* **Interactive Bookings & Leads Directories**: Advanced search, filtering, and real-time lifecycle tracking.
* **Technician & Dispatch Center**: Active technician status boards, schedules, and work maps.
* **Inventory Control Panel**: Tracking inventory stock lists, creating unified ledger transactions, and recording vendor purchases.
* **Billing & Branding Controls**: Management of global invoice template rules, bank payout options, and layout customizer controls.
* **Staff User Control**: Full status management (active/inactive) and roles configuration.

---

## 2. Technology Stack
* **Core Libraries**: React 18, Vite.
* **Routing**: React Router v6.
* **State Management**: Tanstack React Query (for async server caching, query invalidation) and Zustand (for client state, user tokens, and technician status).
* **Styling**: TailwindCSS utility framework combined with custom Vanilla CSS components and theme-aligned variables.
* **Component Primitives**: Radix UI widgets, Lucide React icons, and Sonner toast alerts.
* **PDF Exporter**: html2canvas and canvas draw wrappers (`pdfHelper.ts`) to render and save crisp document prints directly from the screen DOM.

---

## 3. Structural Map
* `/src/app/admin/`: Admin pages and sections (e.g. `booking/`, `inventory/`, `invoice/`, `users/`, `vendors/`).
* `/src/service/`: Axios fetch wrappers connecting page triggers to backend routes.
* `/src/stores/`: Client stores holding transient state data.
* `/src/components/`: Common atomic widgets, buttons, and layout triggers.
