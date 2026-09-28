# Workspace Rules Router

When operating inside this parent folder, always read and follow the specific coding conventions, naming structures, and design guidelines defined in the Git-tracked sub-directories:

- **Backend Repository Rules (karigarji\_server)**:
  - [Project Context & Architectural Overview](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_server/.agents/rules/projectContext.md): System goals, modular domain architecture, tech stack, and structural map.
  - [General Workspace Rules](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_server/.agents/rules/generalRules.md): File/folder naming (camelCase with role suffixes & dots, no hyphens), no automated migrations, no automated linting.
  - [Backend & Database Architecture Rules](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_server/.agents/rules/backendRules.md): Modular domain design (`src/modules/*`), thin controllers, 3-tier services, pragmatic repositories, domain events (`EventEmitter2`), RESTful conventions, response interceptors, validation, Snowflake IDs, soft deletes, OpenAPI/Swagger standards, and pure PostgreSQL configuration.
  - [Suggestions Registry](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_server/.agents/suggestions.md): Persistent architectural backlog and tech debt tracker.
- **Frontend Repository Rules (karigarji\_admin\_panel)**:
  - [Project Context](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_admin_panel/.agents/rules/projectContext.md)
  - [General Workspace Rules](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_admin_panel/.agents/rules/generalRules.md)
  - [Frontend Development Rules](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_admin_panel/.agents/rules/frontendRules.md)
  - [React & State Rules](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_admin_panel/.agents/rules/reactRules.md)
  - [Suggestions Registry](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_admin_panel/.agents/suggestions.md): Persistent frontend architectural backlog and optimizations.
- **Customer Mobile App Repository Rules (karigarji\_customer)**:
  - [Project Context](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_customer/.agents/rules/projectContext.md): System goals, mobile architecture, Expo SDK 57, Expo Router, and structural map.
  - [General Workspace Rules](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_customer/.agents/rules/generalRules.md): File naming (camelCase with role suffixes & dots, no hyphens), Continuous Native Generation (CNG), and compilation boundaries.
  - [React Native & Expo Development Rules](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_customer/.agents/rules/reactNativeRules.md): Anti-deprecation matrix (strict ban on RN SafeAreaView, vector-icons, AsyncStorage JWTs), 3-tier domain architecture, Expo Router standards, and native UI rules.
  - [React, State & Network Rules](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_customer/.agents/rules/reactRules.md): TanStack Query v5 standards (createQueryKeyFactory), 4-tier state separation, Zustand session stores, and hardware-backed SecureStore persistence.
  - [Suggestions Registry](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_customer/.agents/suggestions.md): Persistent customer mobile architectural backlog and optimizations.
- **Shared Workspace Skills (`.agents/skills/*`)**:
  - [Archify Diagramming Skill](file:///Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/.agents/skills/archify/SKILL.md): Interactive system architecture, sequence, workflow, dataflow, and lifecycle diagramming engine. Produces standalone HTML artifacts with SVG rendering, route tracing, and theme toggling.

