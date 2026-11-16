---
name: prisma-schema-change
description: Change the RentNest database schema with Prisma 7 (multi-file schema, committed generated client, migrations). Use when adding or changing models, fields, enums or relations in backend/prisma/schema.
---

# Prisma schema change

## Layout

- Schema files: `backend/prisma/schema/*.prisma`, one per model plus `enums.prisma` and `schema.prisma` (generator and datasource).
- Migrations: `backend/prisma/migrations/`.
- Generated client: `backend/prisma/generated/prisma/`, committed to git and imported by `backend/src/lib/prisma.ts`.
- Config: `backend/prisma.config.ts` reads `DATABASE_URL`.

## Steps

1. Edit the matching `.prisma` file.
   New enums go in `enums.prisma`.
2. Make new required columns safe for existing rows with a `@default(...)` or make them optional.
3. Create the migration against a local database only:

   ```bash
   cd backend
   npm run db:migrate -- --name <short_snake_case_name>
   npm run db:generate
   ```

4. Read the generated `migration.sql` and confirm it does not drop columns or tables you did not intend to drop.
5. Update services, interfaces and `frontend/lib/types.ts` if the shape exposed by the API changed.
6. Commit the schema, migration folder and regenerated client together.

## Rules

- Never run migrations against a production `DATABASE_URL`.
- Never edit a migration that has already been applied. Add a new one.
- Never hand-edit files in `prisma/generated/`.

## Verify

```bash
cd backend && npx tsc --noEmit && npm run build
```
