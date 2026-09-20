# Frontend Development Rules

- **Global Error Handler**:
  - Always configure global catch-all handlers on the `QueryCache` and `MutationCache` in `new QueryClient` to toast errors dynamically.
  - Automatically parse standard and structured error arrays (e.g., status 400, 420, 422) containing `firstError.message` fields.
- **Design & Styling**:
  - Use custom, premium color palettes (e.g., tailwind theme colors like `text-primary`, `bg-card`, `bg-muted/10`).
  - Never use hyphens in new component folders or filenames (comply with general camelCase rules).
- **Code Splitting & Component Structure**:
  - Keep dashboard modules readable and maintainable by dividing components logically.
  - The entry page file (e.g. `XPage.tsx`) should act as the coordinator and orchestrator: it handles routing, breadcrumbs, top-level layout wrapper, and imports main views.
  - Place large functional blocks, tab contents, and complex modals inside a `uiBlocks/` subdirectory.
  - Place small, reusable presentation components (e.g., table rows, custom selectors, input wrappers) inside a `uiElements/` subdirectory.
  - Avoid creating massive single files (e.g., over 500 lines) with all logic inline.
