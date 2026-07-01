# PUG Mobile Student Development Guide

Back to [README.md](https://github.com/Plataforma-Universidade-Gratuita/pug-docs/blob/main/pug-mobile-student/README.md).

## 🧩 Established code patterns

### Thin route wrappers

Expo Router route files are intentionally thin. They usually import a feature entry and return it directly.

Representative route wrappers include:

- `app/(protected)/(tabs)/activity/index.tsx` -> `features/activity/ActivityScreen`
- `app/(protected)/(tabs)/discover/index.tsx` -> `features/discover/DiscoverScreen`
- `app/(protected)/(tabs)/profile/index.tsx` -> `features/profile/ProfileScreen`
- `app/(public)/login.tsx` -> login feature entry

This keeps routing concerns in `app/` and view logic in `features/`.

### Provider-first application shell

Global behavior is centralized in:

- [root-layout/RootLayout.tsx](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/root-layout/RootLayout.tsx)
- [root-layout/RootNavigator.tsx](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/root-layout/RootNavigator.tsx)
- [contexts/providers.tsx](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/contexts/providers.tsx)

New cross-cutting concerns should usually enter through the provider stack or root navigator instead of being repeated per screen.

### Feature-sliced screen composition

The repository uses feature directories under [features/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/features).

Common structure:

- screen entry component
- feature sections
- feature-local constants
- feature-local `utils.ts`
- feature-local `styles.ts`
- feature-local form or filter components when needed

Keep screen-specific mapping and presentation helpers near the feature unless they are reused across multiple domains.

### API domain split

The [api/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/api) directory mirrors backend domains:

1. `academic`
2. `geo`
3. `identity`
4. `partner`
5. `project`

Typical API slice contents:

- `endpoints.ts`
- `queries.ts`
- `mutations.ts` when writes exist
- `keys.ts`
- `index.ts`

Client components should call the exported API slice functions and hooks, not `fetch(...)` directly.

### Validation-first requests

Request and response parsing is not optional in the repository.

Current standards:

- parse request bodies with Zod before sending them
- parse successful backend envelopes before exposing data to feature screens
- parse error envelopes instead of treating backend errors as opaque strings
- normalize search/filter payloads through utilities before building query keys

Representative files:

- [api/utils.ts](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/api/utils.ts)
- [api/project/projects/endpoints.ts](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/api/project/projects/endpoints.ts)
- [api/project/enrollments/endpoints.ts](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/api/project/enrollments/endpoints.ts)
- [api/project/attendances/endpoints.ts](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/api/project/attendances/endpoints.ts)

## 🏷️ Naming conventions

Patterns visible in the repo:

- **Feature screens** named `HomeScreen`, `DiscoverScreen`, `ProfileScreen`, `NewAttendanceModalScreen`
- **Feature sections** grouped under directories such as `home-sections`, `discover-sections`, and `profile-sections`
- **Query hooks** prefixed with `use` and suffixed with `Query`
- **Mutation hooks** prefixed with `use` and suffixed with `Mutation`
- **Schemas** ending in `Schema`
- **API types** ending in `Request`, `Response`, or `ComplexSearchResponse`
- **Form value types** ending in `FormValues`
- **Constants** in screaming snake case
- **Imports** favor the `@/*` alias

Route naming should follow Expo Router conventions:

- `[id]` for single-resource detail routes
- `[projectId]` when the project identifier is the domain key
- route groups for shell structure, such as `(public)`, `(protected)`, `(tabs)`, and `(modals)`

## 📦 Package and module conventions

### `api/*`

Use the existing endpoint/query/mutation/key split.

For a new domain resource, prefer:

```text
api/<domain>/<resource>/
├── endpoints.ts
├── index.ts
├── keys.ts
├── queries.ts
└── mutations.ts
```

Only add `mutations.ts` when the mobile app actually writes to that resource.

### `schemas` and `types`

Schemas and types mirror the backend/domain split:

- `academic`
- `geo`
- `identity`
- `partner`
- `project`
- shared/root exports

Use schemas as the runtime source of truth and infer or align TypeScript types from them when possible.

### `features`

Feature directories should own screen-level composition. Reusable presentational components can remain feature-local until they are needed by another feature.

Move shared UI to:

- [components/primitives/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/components/primitives) for low-level reusable controls
- [components/composite/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/components/composite) for higher-level composed building blocks

Use existing primitives before introducing new one-off controls.

### `stores`

Zustand stores should hold app-level state that must survive across route boundaries or coordinate multiple features.

Current major stores include:

- auth session
- current former-student context
- locale
- theme

Avoid putting short-lived form state into Zustand when React Hook Form or local component state is enough.

## ⚠️ Error handling patterns

### API errors

API utilities parse backend error envelopes and throw structured errors that feature screens can map to localized messages.

Do not handle raw `Response` objects in feature screens unless a new API utility is being created.

### Session invalidation

Invalid or expired sessions should flow through the API session provider and auth store.

Expected behavior:

- clear persisted tokens
- clear current former-student context
- mark the user as unauthenticated
- let protected routing redirect to login

### Feature screen errors

Screens generally use state cards or footer errors instead of throwing UI-breaking exceptions.

Examples:

- home renders a state card when current student, enrollment, attendance, or project queries fail
- discover renders profile/project error states
- new attendance uses `useServerErrorState` for form-level server feedback

## ✅ Validation patterns

Current standards in code:

- define Zod schemas before wiring API calls
- parse outbound mutation bodies before `JSON.stringify(...)`
- build complex search request objects through API utilities
- serialize normalized filter payloads into query keys
- validate forms through localized Zod schema factories
- keep user-facing validation messages in translation files

Representative files:

- [schemas/api/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/schemas/api)
- [schemas/client/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/schemas/client)
- [hooks/useLocalizedZodForm.ts](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/hooks/useLocalizedZodForm.ts)

## 🧪 Testing expectations for new code

Automated tests are **not configured in this repository**.

What is enforced today:

- `npm run format:check`
- `tsc --noEmit`
- `npm run trans`
- `npm run build:web`

That full chain is wrapped by:

```bash
npm run verify
```

When adding new code, at minimum keep `npm run verify` green.

If you introduce automated tests, keep the repo consistent:

- add an explicit script to [package.json](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/package.json)
- wire it into GitHub Actions
- document whether the suite is unit, component, integration, or end-to-end
- decide whether native simulators are required or whether the suite can run in Node/web CI

## ✍️ How to add new code without breaking local standards

1. Add routes in `app/` and keep route files thin.
2. Put screen composition and feature logic in `features/`.
3. Add or extend Zod schemas before wiring fetch calls.
4. Route backend calls through `api/*` endpoint wrappers.
5. Use React Query hooks and query keys for server state.
6. Use Zustand only for cross-feature app state.
7. Add translation keys to **all** supported locale files.
8. Reuse primitives and composite components before creating new UI pieces.
9. Run `npm run verify` before treating the change as done.

## 🚫 Anti-patterns to avoid

- calling `fetch(...)` directly from feature screens
- accepting backend JSON without Zod parsing
- storing short-lived form state globally
- mixing route wrapper concerns into feature components
- bypassing the auth store when clearing or replacing sessions
- allowing non-former-student tokens into the student app
- hardcoding user-facing text without translation entries
- adding one-off UI components when an existing primitive/composite already matches the need
- creating duplicate query keys manually instead of using the slice key factory

## 🪤 Practical pitfalls

- `EXPO_PUBLIC_API_URL` is public and is baked into Expo web builds; changing only Azure runtime variables does not rebuild already-exported static assets.
- `dev:mock` only swaps environment variables through `scripts/with-env.mjs`; a mock server bootstrap was not included here.
- The app rejects non-former-student tokens locally, so admin credentials should not be used to test student login.
- New translated text must be added to every supported locale file because `npm run trans` is part of `verify`.
- Attendance creation assumes approved enrollment data is available and current; backend authorization must still enforce the same rule.
- Expo web, Android, and iOS can differ around storage and navigation behavior; verify the target platform when changing auth or routing.
