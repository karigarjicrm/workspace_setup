# General Workspace Rules

* **Filenames & Folder Names**: Use camelCase formatting for all folders and files. NEVER use hyphens (`-`) in directories or file names (e.g. use `invoiceService.ts` instead of `invoice-service.ts`).
* **Database Migrations**: NEVER run Prisma migrations (`npx prisma migrate dev` or similar) programmatically. Always ask the developer to check the schema changes and run migrations manually in their shell.
* **Linting & Formatting**: NEVER run code linting or formatting commands (such as prettier, eslint) programmatically. Respect and preserve the existing editor and repository formatting conventions.
* **Preserve Documentation**: Always maintain existing code docstrings, JSDoc annotations, and code comments unless explicitly requested to change them.
