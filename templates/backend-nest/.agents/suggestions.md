# Backend Architectural Backlog & Suggestions (`karigarji_server`)

Persistent registry for out-of-scope backend architectural improvements, tech debt, performance optimizations, and design patterns.

## Registry

| ID | Domain / Component | Suggestion & Rationale | Priority | Status | Discovered In Task | Date |
|---|---|---|---|---|---|---|
| **SUG-001** | `src/modules/auth` | Replace legacy `throw new ApiError(...)` with strongly-typed Domain Exceptions (`UserNotFoundException`, `EmailAlreadyInUseException`, etc.) per Section 12 of `backendRules.md`. | Medium | Proposed | `modular-architecture-cleanup` | 2026-09-20 |
