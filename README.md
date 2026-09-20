# Agentic Workspace Orchestrator

This repository serves as a lightweight, production-grade **Meta-Orchestrator** for multi-project development using AI agents (such as Google Antigravity / Gemini CLI).

It decouples agent operating guidelines, workflow routing, and task tracking from the underlying project codebases, allowing individual project repositories to maintain their own independent Git histories without coupling.

---

## 1. Architecture Overview

```text
karigarji_workspace/                        # Meta-Orchestrator Repository (Git Root)
├── .gitignore                              # Strict Allowlist: ignores child repos & dynamic tasks
├── AGENTS.md                               # Authoritative operating principles for AI agents
├── README.md                               # This setup and architecture guide
├── .agents/
│   └── rules/
│       └── rulesRedirect.md                # Global router directing agents to project rules
├── tasks/                                  # Ephemeral runtime task tracking (ignored in Git)
│   └── .gitkeep                            # Preserves folder shell in Git
│
├── karigarji_server/                       # Independent Git Repo (Backend - NestJS)
│   └── .agents/                            # Project-scoped agent context
│       ├── rules/                          # Backend architectural & domain rules
│       └── suggestions.md                  # Project-scoped tech debt & suggestions registry
│
└── karigarji_admin_panel/                  # Independent Git Repo (Frontend - React/Vite)
    └── .agents/                            # Project-scoped agent context
        ├── rules/                          # Frontend design, state & UI rules
        └── suggestions.md                  # Project-scoped tech debt & suggestions registry
```

---

## 2. Core Agent Guidelines (`AGENTS.md`)

All agents operating in this workspace strictly adhere to the guidelines codified in [`AGENTS.md`](./AGENTS.md):
1. **Clarification & Single-Question Focus**: Ask questions one at a time with structured choices; never assume ambiguous requirements.
2. **Mandatory Step Confirmation**: Never modify or create files without presenting proposed steps and receiving explicit user approval.
3. **Task Tracking (`tasks/`)**: Use phase status boards and active resumption checkpoints to ensure zero context loss across interruptions.
4. **Production-Grade Patterns**: Follow domain modularity and clean architecture standards.
5. **Continuous Rule Synchronization**: Automatically sync architectural decisions into `.agents/rules/*` so documentation never drifts.
6. **Critical Engineering Partnership (Zero Sycophancy)**: Rationally challenge flawed designs, debate trade-offs, and point out antipatterns.
7. **Persistent Suggestions Registry**: Record out-of-scope improvements in `<project>/.agents/suggestions.md` so they are never lost when `tasks/` are cleared.

---

## 3. How to Setup `.agents/` Inside Any Project Folder

When adding a new service or repository to this workspace, follow this 4-step setup guide:

### Step 1: Create the `.agents/` Directory Structure
Inside the child project folder, create:
```text
<project-folder>/
└── .agents/
    ├── rules/
    │   ├── projectContext.md      # Tech stack, goals, and high-level architectural map
    │   ├── generalRules.md        # File naming conventions, code hygiene, and tooling rules
    │   └── <domain>Rules.md       # Framework-specific rules (e.g. backendRules.md, reactRules.md)
    └── suggestions.md             # Project-specific backlog for out-of-scope improvements
```

### Step 2: Define Project Rules (`rules/*.md`)
* **`projectContext.md`**: Specify the framework version, runtime, directory anatomy, and core principles.
* **`generalRules.md`**: Enforce strict naming (e.g. camelCase with role suffixes and dots, no hyphens), linting policies, and migration rules.
* **Domain Rules (`backendRules.md` / `frontendRules.md`)**: Specify architectural patterns (e.g. 3-tier services, thin controllers, state management patterns, response envelopes).

### Step 3: Scaffold the Suggestions Registry (`suggestions.md`)
Create a persistent registry to capture out-of-scope tech debt and optimizations discovered during tasks:
```markdown
# Architectural Backlog & Suggestions (<project_name>)

Persistent registry for out-of-scope architectural improvements, tech debt, and optimizations.

## Registry
| ID | Domain / Component | Suggestion & Rationale | Priority | Status | Discovered In Task | Date |
|---|---|---|---|---|---|---|
```

### Step 4: Register the Project in the Global Router
Open the workspace router at [`.agents/rules/rulesRedirect.md`](./.agents/rules/rulesRedirect.md) and register the new project:
```markdown
* **<Project Name> Rules (`<project-folder>`)**:
  - [Project Context & Architecture](file:///.../<project-folder>/.agents/rules/projectContext.md)
  - [General Rules](file:///.../<project-folder>/.agents/rules/generalRules.md)
  - [Domain Rules](file:///.../<project-folder>/.agents/rules/<domain>Rules.md)
  - [Suggestions Registry](file:///.../<project-folder>/.agents/suggestions.md)
```

---

## 4. Task Execution Protocol (`tasks/`)

Every primary task runs inside an isolated directory: `tasks/<task-name>/`.

* **`steps.md`**: Contains a **Phase Progress Board**, an **Active Resumption Checkpoint** (Current Phase, In-Flight Step, Next Immediate Action), and an itemized checklist.
* **`logs.md`**: Contains phase-tagged execution logs with timestamps and architectural decision records.

Because tasks are ephemeral, the root `.gitignore` ignores all task directories while preserving `tasks/.gitkeep`. Any permanent architectural insights are transferred to `<project>/.agents/suggestions.md` before task completion.

---

## 5. Allowlist Git Strategy

The root `.gitignore` employs a strict **Deny-All / Allowlist** strategy:
* `/*`: Ignores everything at the root by default.
* Explicit negation (`!`) selectively tracks only:
  * `AGENTS.md`
  * `README.md`
  * `.gitignore`
  * `.agents/`
  * `tasks/.gitkeep`

Child repositories (`karigarji_server`, `karigarji_admin_panel`, and any future cloned services) are automatically ignored by default, ensuring zero accidental commits to the meta-orchestrator.
