# PUG Web Admin Development Guide

Back to [README.md](https://github.com/Plataforma-Universidade-Gratuita/pug-docs/blob/main/pug-web-admin/README.md).

## 🧩 Established code patterns

### Thin route wrappers

App Router page files are intentionally thin. They usually import a feature entry and return it directly.

Representative route wrappers live in `app/(app)/project/page.tsx` and `app/(app)/identity/users/page.tsx`. Their paired feature entries are:

- [features/project/ProjectOverviewPage.tsx](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/features/project/ProjectOverviewPage.tsx)
- [features/identity/users/UsersPage.tsx](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/features/identity/users/UsersPage.tsx)

This keeps routing concerns in `app/` and view logic in `features/`.

### Explicit client/server layering

The repo consistently separates:

1. **Client hooks and browser fetches** in [api/web/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/api/web)
2. **Browser-facing Next route handlers** in [app/api/v1/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/app/api/v1)
3. **Backend service wrappers** in [api/services/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/api/services)

Preserve that split. Client components should normally call `api/web`, not `api/services` directly.

### Provider-first application shell

Global behavior is centralized in:

- [app/layout.tsx](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/app/layout.tsx)
- [app/providers.tsx](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/app/providers.tsx)

New cross-cutting concerns should usually enter through this provider stack, not through ad hoc per-page setup.

### Feature-sliced UI composition

The repository uses domain directories under [features/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/features) and domain-matched route families under [app/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/app).

Common structure:

- module overview page
- collection page
- entity detail page
- feature-local utils and mapping helpers

### Validation-first requests

Request and response parsing is not optional in the repository.

- route handlers parse request bodies with `parseRouteBody(...)`
- service wrappers parse backend envelopes with Zod
- client fetchers parse `/api/v1/*` responses with Zod

See:

- [app/api/utils.ts](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/app/api/utils.ts)
- [api/services/utils.ts](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/api/services/utils.ts)
- [api/web/utils.ts](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/api/web/utils.ts)

## 🏷️ Naming conventions

Patterns visible in the repo:

- **Plural directories** for collections: `users`, `projects`, `courses`
- **Entity pages** named `UserPage`, `ProjectPage`, `CoursePage`
- **Collection pages** named `UsersPage`, `ProjectsPage`, `CoursesPage`
- **Overview pages** named `IdentityOverviewPage`, `ProjectOverviewPage`
- **Schema names** ending in `Schema`
- **API types** ending in `Request`, `Response`, or `ComplexSearchResponse`
- **React Query hooks** prefixed with `use`
- **Constants** in screaming snake case
- **Imports** favor the `@/*` alias from [tsconfig.json](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/tsconfig.json)

Route param naming is also consistent:

- `[userId]`
- `[projectId]`
- `[formerStudentId]`

## 📦 Package and module conventions

### `api/web`

Typical contents:

- `endpoints.ts`
- `keys.ts`
- `queries.ts`
- optional mutation helpers when the slice needs writes

Example:

- [api/web/identity/users/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/api/web/identity/users)

### `api/services`

Typical contents:

- `endpoints.ts`
- `index.ts`

These are backend-oriented wrappers that call `NEXT_PUBLIC_API_URL`.

### `schemas` and `types`

Schemas and types mirror the domain split:

- `academic`
- `geo`
- `identity`
- `partner`
- `project`
- shared/root exports

### `components`

- [components/primitives/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/components/primitives) for low-level reusable pieces
- [components/composite/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/components/composite) for higher-level composed building blocks

Use existing primitives before introducing new one-off controls.

## ⚠️ Error handling patterns

### Browser-facing errors

[api/web/utils.ts](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/api/web/utils.ts) throws `WebApiError` and performs forced logout redirect handling for `401` and `403` responses.

### Backend-facing errors

[api/services/utils.ts](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/api/services/utils.ts) throws `ApiError` after parsing backend error envelopes, including field details and correlation IDs when present.

### Route handler errors

[app/api/utils.ts](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/app/api/utils.ts) maps:

- `Not found` -> `404`
- `Forbidden` -> `403`
- `ApiError` -> passthrough status + structured payload
- unexpected failures -> `500`

It also centralizes cookie clearing when auth is no longer valid.

## ✅ Validation patterns

Current standards in code:

- validate request bodies with Zod before forwarding them
- parse successful responses with Zod before exposing them to pages
- parse error envelopes instead of treating backend errors as opaque text
- build complex search payloads through feature utilities, then serialize those filters for query keys

Representative files:

- [api/web/project/projects/queries.ts](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/api/web/project/projects/queries.ts)
- [app/api/v1/projects/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/app/api/v1/projects) for the catch-all project route handler family

## 🧪 Testing expectations for new code

Automated tests are **not configured in this repository**.

What is enforced today:

- `npm run format:check`
- `tsc --noEmit`
- `npm run trans`
- `npm run build`

That full chain is wrapped by:

```bash
npm run verify
```

When adding new code, at minimum keep `npm run verify` green.

If you introduce automated tests, keep the repo consistent:

- add an explicit script to [package.json](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/package.json)
- wire it into GitHub Actions
- document whether the suite is unit, component, integration, or e2e

## ✍️ How to add new code without breaking local standards

1. Add UI routes in `app/` and keep page files thin.
2. Put domain logic, page composition, and mappers in `features/`.
3. Add or extend Zod schemas before wiring fetch calls.
4. Route browser calls through `app/api/v1/*`.
5. Route backend calls through `api/services/*`.
6. Reuse `api/web/*` query keys and fetch helpers.
7. Add translation keys to **both** locale files.
8. Run `npm run verify` before treating the change as done.

## 🚫 Anti-patterns to avoid

- calling `NEXT_PUBLIC_API_URL` directly from client components
- bypassing Zod parsing in route handlers or fetch wrappers
- mixing page routing concerns into `features/` wrappers
- storing auth token logic outside the helpers in [auth/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/auth)
- hardcoding user-facing text without adding translation entries
- creating duplicate primitives when an existing primitive/composite already matches the need

## 🪤 Practical pitfalls

- `npm run trans` is part of both `format` and `verify`, so translation drift breaks the pipeline quickly.
- [scripts/checkUnusedTranslations.js](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/scripts/checkUnusedTranslations.js) scans `store` (singular), while the repo directory is [stores/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/stores). If a key is referenced only from Zustand store files, verify that the checker still sees it.
- `dev:mock` only swaps environment variables through [scripts/with-env.mjs](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/scripts/with-env.mjs); a mock server bootstrap was not included here.
