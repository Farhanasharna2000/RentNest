---
name: backend-module
description: Add or extend a RentNest Express API module (route, controller, service, interface) following the project pattern. Use when adding an endpoint, a new feature module, or changing backend business logic in backend/src/modules.
---

# Backend module

Every feature lives in `backend/src/modules/<feature>/` with four files.
Use `backend/src/modules/categories/` as the reference implementation.

## Steps

1. Read the module you are changing and `backend/src/app.ts` before editing.
2. Put types for request bodies and query params in `<feature>.interface.ts`.
3. Put all Prisma access and business rules in `<feature>.service.ts`.
   Import the client with `import { prisma } from "../../lib/prisma";`.
   Export one object: `export const <feature>Service = { fnA, fnB };`.
4. In `<feature>.controller.ts`, wrap every handler in `catchAsync` and reply with `sendResponse`:

   ```ts
   const getAll = catchAsync(async (req: Request, res: Response) => {
     const result = await featureService.getAll();
     sendResponse(res, {
       success: true,
       statusCode: httpStatus.OK,
       message: "Items retrieved successfully",
       data: result,
     });
   });
   export const featureController = { getAll };
   ```

5. In `<feature>.route.ts`, protect routes with `auth("ADMIN" | "LANDLORD" | "TENANT")` from `../../middlewares/auth`.
   Export `export const featureRoutes = router;`.
6. Mount new routers in `backend/src/app.ts` under `/api/...`.
   Keep the Stripe webhook route above `express.json()`.
7. Read settings only through `backend/src/config/index.ts`.
   Add any new variable to `backend/.env.example`.
8. Update the route table in `docs/API_INTEGRATION.md`.

## Rules

- Ownership checks belong in the service. A landlord may only touch their own properties and requests, and a tenant only their own rentals and payments.
- Do not change existing response shapes. The frontend depends on them.
- Use `http-status` constants, never raw numbers.

## Verify

```bash
cd backend && npx tsc --noEmit && npm run build
```

Report the real output.
