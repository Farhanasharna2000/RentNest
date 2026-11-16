# RentNest

Full-stack rental marketplace.
Tenants browse properties, send rental requests and pay with Stripe.
Landlords manage listings and approve or reject requests.
Admins moderate users, properties and rentals.

| App | Path | Stack | Package manager |
|---|---|---|---|
| API | `backend/` | Express 5, TypeScript (ESM), Prisma 7 with `@prisma/adapter-pg`, PostgreSQL, JWT, Stripe | npm |
| Web | `frontend/` | Next.js 16 App Router, React 19, TypeScript, Tailwind CSS 3, lucide-react, sonner | npm |

The API contract lives in [docs/API_INTEGRATION.md](docs/API_INTEGRATION.md).
Keep it in sync whenever a route, payload or response shape changes.

## Commands

### Backend (`cd backend`)

```bash
npm run dev           # tsx watch src/server.ts, port 5000
npm run build         # prisma generate && tsup -> dist/server.js
npm run db:generate   # regenerate Prisma client into prisma/generated
npm run db:migrate    # prisma migrate dev
npx tsc --noEmit      # typecheck
```

### Frontend (`cd frontend`)

```bash
npm run dev           # port 3000
npm run build
npm run lint
npx tsc --noEmit      # typecheck
```

Known gaps (as of 2026-11-16):

- `npm run lint` in the frontend fails because Next.js 16 removed `next lint`. An ESLint setup is still to be added.
- `npx tsc --noEmit` in the backend reports one error at `src/modules/auth/auth.service.ts:55` (JWT `expiresIn` type). The tsup build still succeeds.

There is no automated test suite yet.
Before calling a change done, run typecheck in each app you touched and `npm run build` in the frontend.
Report real results and never claim a check passed if you did not run it.

## Backend architecture

- `src/server.ts` starts the server, `src/app.ts` mounts every router under `/api`.
- The Stripe webhook (`POST /api/payments/webhook`) is registered before `express.json()` with `express.raw()`.
  Never move it below the JSON parser, or signature verification breaks.
- Each feature is a module in `src/modules/<feature>/` with four files:
  - `<feature>.route.ts` declares routes and applies `auth(...roles)`.
  - `<feature>.controller.ts` wraps handlers in `catchAsync` and replies with `sendResponse`.
  - `<feature>.service.ts` holds business logic and all Prisma calls.
  - `<feature>.interface.ts` holds request and payload types.
- Modules export a single object, for example `export const categoryService = { ... }` and `export const categoryRoutes = router`.
- Responses use `{ success, statusCode, message, data, meta? }` from `src/utils/sendResponse.ts`.
- Config is read only through `src/config/index.ts`. Never read `process.env` elsewhere in new code.
- Prisma schema is split across `prisma/schema/*.prisma`.
  The generated client in `prisma/generated/` is committed. Regenerate it with `npm run db:generate` after any schema change and commit the result together with the migration.
- Roles are `ADMIN`, `LANDLORD`, `TENANT`.

## Frontend architecture

- Routes live in `app/`, UI in `components/<area>/`, shared logic in `lib/`.
- All HTTP calls go through `apiFetch` in `lib/api/client.ts`, via one wrapper file per domain in `lib/api/`.
  Paths passed to `apiFetch` omit the `/api` prefix.
- Shared types are in `lib/types.ts`.
- Auth state is in `lib/auth.ts`.
  The token lives in localStorage for API calls and in the `rentnest_token` and `rentnest_role` cookies for `middleware.ts`, which guards `/dashboard/*` by role.
- Use `sonner` toasts for API success and error feedback, and `components/ui/StatusBadge.tsx` for request status.
- Keep the existing Tailwind design. Do not restyle or restructure UI unless asked.

## Rules

- Change behavior only when the task asks for it. Refactors must keep every route, payload, response and screen the same.
- Never add or call APIs that do not already exist in the backend.
- Never commit `.env` files, Stripe keys or JWT secrets. Add new variables to the matching `.env.example`.
- Treat auth, RBAC, payments and the Stripe webhook as critical. Changes there need a careful review with the `rentnest-code-reviewer` agent.
- No em dashes in prose. In long Markdown, put each full sentence on its own line.

## Git commit rules

- Use Conventional Commits: `feat:`, `fix:`, `refactor:`, `docs:`, `chore:`, `style:`, `test:`.
- Never add `Co-Authored-By` lines or any AI attribution to commits or PRs.
- Never set an AI tool as commit author.
- Commit messages are short, specific and human-style.
- Hooks in `.githooks/` enforce this. Enable them once per clone with `git config core.hooksPath .githooks`.

## Skills and agents

Project skills in `.claude/skills/`:

- `backend-module`: add or extend an Express module following the route, controller, service and interface pattern.
- `frontend-feature`: add a page, component or API wrapper on the Next.js side.
- `prisma-schema-change`: change the database schema safely with a migration and a regenerated client.

Project agents in `.claude/agents/`:

- `rentnest-code-reviewer`: reviews a diff for correctness, security and convention issues. Read-only.
- `rentnest-backend-tester`: checks the API with typecheck, build and requests against a local server. Reports bugs, never fixes app code.
- `rentnest-frontend-tester`: checks the web app with lint, typecheck, build and role-based flows. Reports bugs, never fixes app code.
