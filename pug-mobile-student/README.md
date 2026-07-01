# PUG Mobile Student

PUG Mobile Student is the former-student application for the PUG platform. It is an Expo Router application that gives students a mobile-first workspace for authentication, credential setup, counterpart-hour progress, project discovery, enrollment activity, attendance creation, QR access, profile data, localization, and theme preferences.

## 🎯 Project purpose

This repository provides:

- a protected former-student app shell with login and persisted session handling
- first-access credential setup through the password-wired flow
- a home dashboard centered on counterpart-hour progress and recent activity
- area-aware project discovery based on the student's course and area of expertise
- enrollment and attendance activity views
- attendance creation for approved enrollments and QR access after creation
- profile, language, and theme preferences
- typed backend API wrappers targeting `EXPO_PUBLIC_API_URL`
- reusable primitives, composite components, i18n, and theme support

Representative files:

- [package.json](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/package.json)
- [app.json](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/app.json)
- [root-layout/RootLayout.tsx](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/root-layout/RootLayout.tsx)
- [root-layout/RootNavigator.tsx](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/root-layout/RootNavigator.tsx)
- [stores/auth.ts](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/stores/auth.ts)
- [features/home/HomeScreen.tsx](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/features/home/HomeScreen.tsx)

## ✨ High-level feature summary

- **Authentication and route protection** through `/login`, protected route groups, token validation, and refresh-token recovery
- **Credential setup gating** through `/wire-credentials` when the backend marks the password as not wired
- **Current former-student context** loaded from identity, academic, and course APIs
- **Student home dashboard** with progress, quick actions, latest enrollment, and latest attendance information
- **Project discovery** filtered by area of expertise, project availability, and ongoing enrollment state
- **Attendance creation** for approved enrollments, followed by QR navigation
- **Profile and preferences** for identity data, academic data, theme, language, and logout actions
- **Typed API layer** through `api/`, Zod schemas, React Query, and backend envelope parsing
- **Localization and theming** through i18next, persisted locale state, persisted theme mode, and shared theme tokens

## 🧰 Tech stack

- **Framework:** Expo SDK 54 + Expo Router 6
- **Language:** TypeScript with strict compiler expectations
- **UI runtime:** React 19 + React Native 0.81
- **Navigation:** Expo Router + React Navigation
- **Data fetching:** TanStack React Query
- **Forms and validation:** React Hook Form + Zod
- **State:** Zustand
- **Localization:** i18next + `react-i18next`
- **Storage:** Expo SecureStore through shared storage utilities
- **UI:** React Native Paper, Lucide React Native, Expo Vector Icons
- **Containerization:** multi-stage Docker build with Expo web export served by Nginx
- **CI:** GitHub Actions

Key config files:

- [app.json](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/app.json)
- [package.json](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/package.json)
- [Dockerfile](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/Dockerfile)

## 🗂️ Repository and module overview

### Top-level structure

| Path | Role |
| --- | --- |
| [app/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/app) | Expo Router layouts, route groups, tabs, modals, and route wrappers |
| [root-layout/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/root-layout) | Root provider composition and bootstrap navigator |
| [api/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/api) | Backend endpoint wrappers, React Query hooks, mutations, and query keys |
| [features/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/features) | Feature screens and feature-local composition logic |
| [components/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/components) | Reusable primitive and composite UI components |
| [schemas/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/schemas) | Zod schemas for API and client payloads |
| [stores/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/stores) | Zustand stores for auth, current student context, locale, and theme |
| [styles/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/styles) | Shared style tokens and style helpers |
| [i18n/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/i18n) | i18next configuration and locale helpers |
| [locales/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/locales) | Translation bundles |
| [scripts/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/scripts) | Translation and environment helper scripts |
| [types/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/types) | API and client TypeScript types |
| [utils/](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/tree/main/utils) | Shared utility functions |

### Domain coverage

The current API and feature structure covers the student-facing parts of:

- `academic`
- `geo`
- `identity`
- `partner`
- `project`

The app is intentionally centered on the `FORMER_STUDENT` account type. Admin and partner operational flows belong to `pug-web-admin` and the future partner-facing app.

## ▶️ How to run locally

1. Install dependencies:

```bash
npm ci
```

2. Configure the backend URL:

- set `EXPO_PUBLIC_API_URL` in your local shell or environment file
- default fallback used by the API layer: `http://localhost:8080`

Example:

```env
EXPO_PUBLIC_API_URL=http://localhost:8080
```

3. Start the Expo development server:

```bash
npm run dev
```

The repository scripts start Expo on port `3001`.

Mock-oriented local mode:

```bash
npm run dev:mock
```

That command loads [mock-api.env](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/mock-api.env), which points `EXPO_PUBLIC_API_URL` to `http://localhost:8090`.

Mock API server bootstrap is not part of this repository. The documented `dev:mock` flow switches environment values only.

## 🏗️ How to build

Expo web export:

```bash
npm run build:web
```

Container build:

```bash
docker build \
  --build-arg EXPO_PUBLIC_API_URL=http://localhost:8080 \
  -t pug-mobile-student .
```

The Docker image exports the Expo web build and serves the generated `dist` directory through Nginx on port `80`. See [Dockerfile](https://github.com/Plataforma-Universidade-Gratuita/pug-mobile-student/blob/main/Dockerfile).

## ✅ How to test and verify

Dedicated automated tests are not part of the repository.

Current verification commands are:

```bash
npm run verify
npm run lint
npm run format:check
npm run trans
npm run build:web
```

`npm run verify` is the main quality gate and currently covers:

- format check
- lint
- TypeScript type-checking
- translation checks
- Expo web export

## 📚 Documentation

- [Development Guide](https://github.com/Plataforma-Universidade-Gratuita/pug-docs/blob/main/pug-mobile-student/DEVELOPMENT.md)
- [Architecture](https://github.com/Plataforma-Universidade-Gratuita/pug-docs/blob/main/pug-mobile-student/ARCHITECTURE.md)
- [CI/CD](https://github.com/Plataforma-Universidade-Gratuita/pug-docs/blob/main/pug-mobile-student/CICD.md)
