# General Workspace Rules

* **Filenames & Folder Names**: Use camelCase for folder names. For files, use **camelCase with role suffixes and dots** (e.g., `bookingCalculator.service.ts`, `createBooking.dto.ts`, `booking.controller.ts`, `booking.module.ts`, `booking.repository.ts`, `bookingCreated.event.ts`). **NEVER use hyphens (`-`)** in directories or file names.
* **Database Migrations**: **NEVER run Prisma migrations** (`npx prisma migrate dev`, `npx prisma db push`, etc.) programmatically. Always prompt the developer to inspect the schema changes and run migrations manually in their shell.
* **Linting & Formatting**: **NEVER run code linting or formatting commands** (such as Prettier, ESLint) programmatically. Respect and preserve existing editor and repository formatting conventions.
* **Preserve Documentation**: Always maintain existing code docstrings, JSDoc annotations, and code comments unless explicitly requested to change them.
