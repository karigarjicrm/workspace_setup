## 2026-10-16T06:50:37Z

You are the SWE Light Orchestrator.
Your working directory is: /Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/.agents/teamwork/teamwork_preview_swe_1/
The project working directory is: /Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/karigarji_server
The authoritative user request is recorded in: /Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/.agents/teamwork/ORIGINAL_REQUEST.md

Please read the original request from /Users/karigarji/Documents/Yaantriki/repos/karigarji_workspace/.agents/teamwork/ORIGINAL_REQUEST.md.
Task Summary:
Refactor branch-scoped physical stock, custody, movement transactions, serialization, and double-entry GST financial accounts from `src/modules/inventory/` to `src/modules/branches/` with direct operational routes, complying with:
- R1: Relocate Branch Operational Subsystems
- R2: Direct Branch Operational Route Restructuring (/staff/branches/:branchId/...)
- R3: Unidirectional Architecture & Decoupled Domain Contracts
- Acceptance Criteria: `npm run build`, `npm test`, `npm run openapi:export`.

Follow the SWE Light loop: dispatch the implementer, conduct adversarial review rounds, verify via tests and compilation, and notify me upon completion.
