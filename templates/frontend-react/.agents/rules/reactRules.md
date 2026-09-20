# React & Tanstack Query Development Rules

This file defines React coding standards, Hook safety guidelines, Tanstack Query patterns, and internal utility component usage for the frontend panel project.

---

## 1. Loop Prevention & Hook Safety

* **Avoid Infinite Render Loops**:
  * Never place unstable inline references (such as `data || []`, objects created inline, or functions defined inline) directly in `useEffect` dependency arrays.
  * When resetting form state asynchronously inside a `useEffect` after data has finished loading, use a `useRef` initialization tracker flag (`initializedRef`) to ensure `reset(...)` is executed exactly once.
  * Guard form updates (e.g. `setValue(...)` from React Hook Form) inside `useEffect` blocks using strict logical guards (e.g. `if (type === "OUT" && isPurchase)`) to prevent updating states when they are already correct.

---

## 2. Tanstack Query (React Query) Best Practices

* **Query Keys**:
  * Always construct query keys as structured arrays starting with the module scope, followed by action types and variable IDs (e.g., `["office-stock", "items", selectedStoreId, debouncedSearch]`).
* **Conditional Queries**:
  * Set the `enabled` query parameter to prevent calls from executing with empty or undefined variables (e.g., `enabled: !!selectedStoreId`).
* **Cache Invalidation**:
  * Ensure every successful mutation updates the UI state by invalidating the appropriate query key on success (e.g., `queryClient.invalidateQueries({ queryKey: ["office-stock", "items"] })`).

---

## 3. Internal UI Utilities & Component Conventions

* **Conditional Rendering**:
  * Prefer using the project's internal `<TernaryRender>` component from `@/components/internal/ternaryRender` instead of raw Javascript inline ternary operators (`? :`) when rendering large blocks of React markup.
* **Role & Permission Access Guarding**:
  * Wrap restricted components, action buttons, or visual areas using the `<PermissionGuard>` component from `@/components/internal/PermissionGuard`. Avoid manually writing inline checks for roles or permissions.
* **Input Debouncing**:
  * Always use the custom `useDebounce` hook from `@/hooks/useDebounce` to throttle typing events in user search fields before sending them to query keys.
* **Forms & Default Values**:
  * Define explicit defaults for every single form field in `useForm` options to prevent React uncontrolled-to-controlled input warnings.
  * Bind custom inputs, select dropdowns, or custom checkboxes using the `Controller` wrapper from `react-hook-form`.

---

## 4. Code Splitting & File Limits

* **Folder Architecture**:
  * Keep module pages organized under a clean, three-layer architecture:
    1. **Layout Coordinator**: The page entry point (`XPage.tsx`) which acts as the orchestrator.
    2. **uiBlocks/**: A subdirectory housing large modal dialogue forms, tabs, or complex panels.
    3. **uiElements/**: A subdirectory containing smaller, presentation-only components (such as table rows, loaders, custom badges, or select dropdowns).
* **Length Limits**:
  * Do not write single files exceeding 300–400 lines of code. Proactively split views and forms out into `uiBlocks/` or `uiElements/`.

---

## 5. Commenting, Documentation & Error Handling Rules

* **Method, Hook & Function Comments**:
  * Write clear JSDoc-style comments for all custom hooks, API service functions, and helper methods.
  * Document all parameters (`@param`), return values (`@returns`), and exception throw cases.
* **Effect Comments**:
  * Precede every non-trivial `useEffect` block with a multi-line comment explaining *why* the effect is required, what state changes trigger it, and what side-effects it coordinates.
* **Component Comments**:
  * Provide a brief header comment for every functional component (especially inside `uiBlocks/` or `uiElements/`) stating its purpose, main query dependencies, and what props it expects.
* **Exceptional Handling & Error Flows**:
  * Always document explicit exception handling flows, fallback layouts, and API error catch blocks with comments.
  * Explain *why* a particular fallback UI target is rendered or *why* a specific user-facing error message is triggered.

