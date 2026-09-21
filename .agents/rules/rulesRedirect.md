# Workspace Rules Router

When operating inside this parent folder, always read and follow the specific coding conventions, naming structures, and design guidelines defined in the Git-tracked sub-directories:

- **Backend Repository Rules (karigarji\_server)**:
  - [Project Context & Architectural Overview](file:///Users/apple/Documents/Yaantriki/karigarji_workspace/karigarji_server/.agents/rules/projectContext.md): System goals, modular domain architecture, tech stack, and structural map.
  - [General Workspace Rules](file:///Users/apple/Documents/Yaantriki/karigarji_workspace/karigarji_server/.agents/rules/generalRules.md): File/folder naming (camelCase with role suffixes & dots, no hyphens), no automated migrations, no automated linting.
  - [Backend & Database Architecture Rules](file:///Users/apple/Documents/Yaantriki/karigarji_workspace/karigarji_server/.agents/rules/backendRules.md): Modular domain design (`src/modules/*`), thin controllers, 3-tier services, pragmatic repositories, domain events (`EventEmitter2`), RESTful conventions, response interceptors, validation, Snowflake IDs, soft deletes, OpenAPI/Swagger standards, and pure PostgreSQL configuration.
  - [Suggestions Registry](file:///Users/apple/Documents/Yaantriki/karigarji_workspace/karigarji_server/.agents/suggestions.md): Persistent architectural backlog and tech debt tracker.
- **Frontend Repository Rules (karigarji\_admin\_panel)**:
  - [Project Context](file:///Users/apple/Documents/Yaantriki/karigarji_workspace/karigarji_admin_panel/.agents/rules/projectContext.md)
  - [General Workspace Rules](file:///Users/apple/Documents/Yaantriki/karigarji_workspace/karigarji_admin_panel/.agents/rules/generalRules.md)
  - [Frontend Development Rules](file:///Users/apple/Documents/Yaantriki/karigarji_workspace/karigarji_admin_panel/.agents/rules/frontendRules.md)
  - [React & State Rules](file:///Users/apple/Documents/Yaantriki/karigarji_workspace/karigarji_admin_panel/.agents/rules/reactRules.md)
  - [Suggestions Registry](file:///Users/apple/Documents/Yaantriki/karigarji_workspace/karigarji_admin_panel/.agents/suggestions.md): Persistent frontend architectural backlog and optimizations.
