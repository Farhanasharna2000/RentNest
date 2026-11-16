---
name: frontend-feature
description: Add or change a RentNest Next.js page, component or API wrapper while keeping the existing design. Use for work in frontend/app, frontend/components or frontend/lib.
---

# Frontend feature

## Where things go

| What | Where |
|---|---|
| Route | `frontend/app/<path>/page.tsx` |
| UI component | `frontend/components/<area>/<Name>.tsx` |
| API call | `frontend/lib/api/<domain>.ts` |
| Shared type | `frontend/lib/types.ts` |
| Auth helpers | `frontend/lib/auth.ts` |

## Steps

1. Confirm the backend endpoint exists in `docs/API_INTEGRATION.md` and in `backend/src/modules`.
   Never call an endpoint that does not exist.
2. Add a typed wrapper in `lib/api/<domain>.ts` that calls `apiFetch`:

   ```ts
   export function getThings() {
     return apiFetch<ApiResponse<Thing[]>>("/things");
   }
   ```

   Paths omit the `/api` prefix because `apiFetch` adds the base URL.
3. Components that use hooks, localStorage or event handlers start with `"use client"`.
4. Show API errors and success with `toast` from `sonner`.
   Show loading with the existing skeletons in `components/ui/PageSkeleton.tsx`.
5. For a new protected dashboard route, check that `frontend/middleware.ts` already covers it, and that the role check matches the backend `auth(...)` roles.
6. Reuse existing Tailwind classes and components.
   Do not change colors, spacing, layout or copy unless the task asks.

## Verify

```bash
cd frontend && npx tsc --noEmit && npm run build
```

Report the real output.
