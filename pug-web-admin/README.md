# PUG Web Admin

PUG Web Admin is the operational frontend for the PUG platform. It is a Next.js App Router application that gives administrators a browser-based workspace for the same business areas exposed by `pug-service`: academic, geo, identity, partner, and project.

## 🎯 Project purpose

This repository provides:

- a protected admin UI with login and cookie-based session handling
- a command-center style home dashboard
- typed local API routes under `/api/v1/*`
- typed backend service wrappers targeting `NEXT_PUBLIC_API_URL`
- reusable primitives, composite components, i18n, and theme support

Representative files:

- [package.json](../../pug-web-admin/package.json)
- [app/layout.tsx](../../pug-web-admin/app/layout.tsx)
- [proxy.ts](../../pug-web-admin/proxy.ts)
- [features/home/HomeCommandCenterPage.tsx](../../pug-web-admin/features/home/HomeCommandCenterPage.tsx)

## ✨ High-level feature summary

- **Authentication and route protection** through `/login`, `proxy.ts`, and refresh-token recovery
- **Operational module navigation** across academic, geo, identity, partner, and project
- **Typed browser-facing API layer** under `app/api/v1`
- **Typed client data layer** through `api/web` and React Query
- **Typed backend integration** through `api/services`
- **Localization and theming** via i18next, cookies, and provider composition
- **Dashboard aggregation** that combines multiple module queries in the home command center

## 🧰 Tech stack

- **Framework:** Next.js 16 App Router
- **Language:** TypeScript with strict compiler options
- **UI:** React 19, Tailwind CSS 4, Radix UI, Lucide, Sonner
- **Data fetching:** TanStack React Query
- **Forms and validation:** React Hook Form + Zod
- **State:** React context + Zustand
- **Localization:** i18next + `react-i18next`
- **Containerization:** multi-stage Docker build on `node:22-alpine`
- **CI:** GitHub Actions

Key config files:

- [next.config.ts](../../pug-web-admin/next.config.ts)
- [tsconfig.json](../../pug-web-admin/tsconfig.json)
- [Dockerfile](../../pug-web-admin/Dockerfile)

## 🗂️ Repository and module overview

### Top-level structure

| Path | Role |
| --- | --- |
| [app/](../../pug-web-admin/app) | App Router layouts, route groups, pages, and local route handlers |
| [app/api/v1/](../../pug-web-admin/app/api/v1) | Browser-facing API routes |
| [api/web/](../../pug-web-admin/api/web) | Client-side endpoints and React Query hooks |
| [api/services/](../../pug-web-admin/api/services) | Backend service wrappers |
| [auth/](../../pug-web-admin/auth) | Cookie helpers, token validation, refresh logic |
| [features/](../../pug-web-admin/features) | Page composition and feature logic |
| [components/primitives/](../../pug-web-admin/components/primitives) | Reusable low-level UI |
| [components/composite/](../../pug-web-admin/components/composite) | Higher-level UI composition |
| [schemas/](../../pug-web-admin/schemas) | Zod schemas |
| [stores/](../../pug-web-admin/stores) | Zustand state |
| [scripts/](../../pug-web-admin/scripts) | Translation and env helper scripts |
| [public/locales/](../../pug-web-admin/public/locales) | `pt-BR` and `en-US` translations |

### Domain coverage

The current route and API structure covers:

- `academic`
- `geo`
- `identity`
- `partner`
- `project`

There is also an `app/(app)/docs` route-group directory, but route files were **not found in the current codebase** under that tree.

## ▶️ How to run locally

1. Install dependencies:

```bash
npm ci
```

2. Configure the backend URL:

- create `.env` from [.env.example](../../pug-web-admin/.env.example), or set the variable yourself
- required backend base URL: `NEXT_PUBLIC_API_URL`

Example:

```env
NEXT_PUBLIC_API_URL=http://localhost:8080
```

3. Start the development server:

```bash
npm run dev
```

4. Open `http://localhost:3000`.

Mock-oriented local mode:

```bash
npm run dev:mock
```

That command loads [mock-api.env](../../pug-web-admin/mock-api.env), which points `NEXT_PUBLIC_API_URL` to `http://localhost:8090`.

Mock API server bootstrap was **not found in the current codebase**. I checked [package.json](../../pug-web-admin/package.json) scripts and [scripts/](../../pug-web-admin/scripts).

## 🏗️ How to build

Application build:

```bash
npm run build
npm run start
```

Container build:

```bash
docker build -t pug-web-admin .
```

The Docker image uses Next standalone output and runs `server.js` on port `3000`. See [Dockerfile](../../pug-web-admin/Dockerfile).

## ✅ How to test and verify

Dedicated automated tests were **not found in the current codebase**.

Current verification commands are:

```bash
npm run verify
npm run lint
npm run format:check
npm run trans
```

`npm run verify` is the main quality gate and currently covers:

- format check
- lint
- type-checking
- translation checks
- production build

## 📚 Documentation

- [Development Guide](./DEVELOPMENT.md)
- [Architecture](./ARCHITECTURE.md)
- [CI/CD](./CICD.md)
