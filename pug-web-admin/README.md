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

- [package.json](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/package.json)
- [app/layout.tsx](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/app/layout.tsx)
- [proxy.ts](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/proxy.ts)
- [features/home/HomeCommandCenterPage.tsx](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/features/home/HomeCommandCenterPage.tsx)

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

- [next.config.ts](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/next.config.ts)
- [tsconfig.json](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/tsconfig.json)
- [Dockerfile](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/Dockerfile)

## 🗂️ Repository and module overview

### Top-level structure

| Path | Role |
| --- | --- |
| [app/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/app) | App Router layouts, route groups, pages, and local route handlers |
| [app/api/v1/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/app/api/v1) | Browser-facing API routes |
| [api/web/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/api/web) | Client-side endpoints and React Query hooks |
| [api/services/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/api/services) | Backend service wrappers |
| [auth/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/auth) | Cookie helpers, token validation, refresh logic |
| [features/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/features) | Page composition and feature logic |
| [components/primitives/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/components/primitives) | Reusable low-level UI |
| [components/composite/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/components/composite) | Higher-level UI composition |
| [schemas/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/schemas) | Zod schemas |
| [stores/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/stores) | Zustand state |
| [scripts/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/scripts) | Translation and env helper scripts |
| [public/locales/](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/public/locales) | `pt-BR` and `en-US` translations |

### Domain coverage

The current route and API structure covers:

- `academic`
- `geo`
- `identity`
- `partner`
- `project`

There is also an `app/(app)/docs` route-group directory, but no route files are present under that tree.

## ▶️ How to run locally

1. Install dependencies:

```bash
npm ci
```

2. Configure the backend URL:

- create `.env` from [.env.example](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/.env.example), or set the variable yourself
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

That command loads [mock-api.env](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/blob/main/mock-api.env), which points `NEXT_PUBLIC_API_URL` to `http://localhost:8090`.

Mock API server bootstrap is not part of this repository. The documented `dev:mock` flow switches environment values only.

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

The Docker image uses Next standalone output and runs `server.js` on port `3000`. See [Dockerfile](https://github.com/Plataforma-Universidade-Gratuita/pug-web-admin/tree/main/Dockerfile).

## ✅ How to test and verify

Dedicated automated tests are not part of the repository.

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

- [Development Guide](https://github.com/Plataforma-Universidade-Gratuita/pug-docs/blob/main/pug-web-admin/DEVELOPMENT.md)
- [Architecture](https://github.com/Plataforma-Universidade-Gratuita/pug-docs/blob/main/pug-web-admin/ARCHITECTURE.md)
- [CI/CD](https://github.com/Plataforma-Universidade-Gratuita/pug-docs/blob/main/pug-web-admin/CICD.md)
