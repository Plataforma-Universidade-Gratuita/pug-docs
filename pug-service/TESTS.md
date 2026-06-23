# Testing Guide

This document describes how tests are organized in `pug-service`, how the current suite is executed, and what new code is expected to add.

## How tests are organized

Tests are grouped by the same module boundaries used in production code:

- [`shared`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/test/java/br/org/catolicasc/pug/shared)
- [`geo`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/test/java/br/org/catolicasc/pug/geo)
- [`identity`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/test/java/br/org/catolicasc/pug/identity)
- [`partner`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/test/java/br/org/catolicasc/pug/partner)
- [`academic`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/test/java/br/org/catolicasc/pug/academic)
- [`project`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/test/java/br/org/catolicasc/pug/project)
- [`helpers`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/test/java/br/org/catolicasc/pug/helpers)

The current tree contains about 220 test source files and helper classes.

Within a module, tests usually mirror the production package split:

- `domain/` for aggregate, enum, and value-object tests
- `presenter/` for REST resource tests
- `service/impl/` for application-service orchestration tests
- `infra/read/impl/` for query tests
- `infra/persistence/` or `infra/` for mapper and repository coverage

## Test types present in the repository

### Domain-style tests

These are the closest thing to unit tests in the repository. They focus on lifecycle rules, validation, and value-object behavior.

Examples:

- [`ProjectTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/domain/ProjectTest.java)
- [`EnrollmentTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/domain/EnrollmentTest.java)
- [`CounterpartHoursTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/academic/domain/vos/CounterpartHoursTest.java)
- [`UuidV7Test`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/shared/validation/UuidV7Test.java)

### Quarkus-backed integration tests

Most non-domain tests use `@QuarkusTest`. That includes:

- REST resource tests with `RestAssured`
- service tests with injected repositories/mocks
- query tests against the configured test database
- audit/i18n/filter tests in `shared`

Examples:

- [`ProjectResourceTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/presenter/ProjectResourceTest.java)
- [`FormerStudentsServiceImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/academic/service/impl/FormerStudentsServiceImplTest.java)
- [`ProjectQueriesImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/infra/read/impl/ProjectQueriesImplTest.java)
- [`AuditSystemTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/shared/infra/audit/AuditSystemTest.java)

## Naming conventions

The current naming is consistent and should be preserved:

- `*Test` for all test classes
- `*ResourceTest` for REST coverage
- `*ServiceImplTest` for application-service coverage
- `*QueriesImplTest` for read-query coverage
- `*MapperTest` for mapper coverage when present

Examples:

- `AttendanceResourceTest`
- `AccountsServiceImplTest`
- `EnrollmentQueriesImplTest`

## Common test helpers and patterns

### 1. `TestDataFactory` for graph setup

[`TestDataFactory`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/helpers/TestDataFactory.java) is the main fixture builder for persisted graphs. It creates users, accounts, entities, courses, former students, projects, enrollments, project-area links, and attendances in FK-safe order.

Prefer it over hand-wiring long persistence graphs in each test.

### 2. `BaseResourceTest` for resource tests

[`BaseResourceTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/helpers/BaseResourceTest.java) provides:

- injected `TestDataFactory`
- injected `EntityManager`
- injected `UserTransaction`
- `doInTransaction(...)` helper for setup
- `assertUnauthenticated(...)` helper for `401` checks

Resource tests typically:

- extend `BaseResourceTest`
- seed data inside `doInTransaction(...)`
- use `RestAssured.given()`
- apply `@TestSecurity`
- optionally mock auth behavior with `@InjectMock AuthService`

### 3. Builder helpers for commands, requests, and domain objects

The repository contains test builders under:

- [`helpers/builders/commands`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/test/java/br/org/catolicasc/pug/helpers/builders/commands)
- [`helpers/builders/requests`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/test/java/br/org/catolicasc/pug/helpers/builders/requests)
- [`helpers/builders/domain`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/test/java/br/org/catolicasc/pug/helpers/builders/domain)

Use these builders when adding tests for create/update/validate flows. That is the established pattern for keeping test setup concise.

## Mocking and stubbing patterns

`pug-service` mostly uses Quarkus-native mocking through `@InjectMock`.

Common patterns already in use:

- mock cross-module services in service tests
- mock `AuthService` in resource tests and service tests when the current account matters
- verify audit publishing with mocked [`AuditPublisher`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/shared/infra/audit/AuditPublisher.java)
- stub security restrictions like `requireCurrentAccountNotOfType(...)` when the test is about another branch

Examples:

- [`ProjectServiceImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/service/impl/ProjectServiceImplTest.java)
- [`EnrollmentsServiceImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/service/impl/EnrollmentsServiceImplTest.java)
- [`AttendanceServiceImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/service/impl/AttendanceServiceImplTest.java)

Mockito and AssertJ are already part of the test stack through `quarkus-junit5-mockito`, `mockito-core`, `mockito-junit-jupiter`, and `assertj-core`.

## Test environment and runtime behavior

### Local test profile

[`src/test/resources/application.properties`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/resources/application.properties) configures the test profile to:

- use random HTTP ports
- enable CORS for local frontend calls
- enable Quarkus Dev Services for PostgreSQL 16 and MongoDB 7
- clean and migrate Flyway at startup
- use test-only JWT keys and peppers

That means local `./mvnw test` expects a working container runtime, even if it does not require pre-started local databases.

### CI behavior

The [`verify.yml`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/.github/workflows/verify.yml) workflow disables Dev Services and injects explicit PostgreSQL and MongoDB connection settings that point at GitHub Actions service containers:

- PostgreSQL on `localhost:5434`
- MongoDB on `localhost:27019`

So the same suite runs in two slightly different modes:

- locally: Dev Services containers managed by Quarkus
- CI: external containers managed by GitHub Actions

## How to create new tests

### For domain changes

- add or update tests under the matching `domain/` package
- assert lifecycle transitions, error codes, and value-object invariants directly

### For new resources or endpoint changes

- add or update `*ResourceTest`
- seed the minimum required graph with `TestDataFactory`
- use `@TestSecurity` for role/auth cases
- assert both status code and response payload shape

### For new service orchestration

- add or update `*ServiceImplTest`
- mock collaborating services with `@InjectMock`
- verify important side effects such as audit publishing, cross-module calls, and status propagation

### For new search filters or read projections

- add or update `*QueriesImplTest`
- cover matching filters, null handling, paging metadata, and ordering where the query defines it

## Test commands

### Fast local suite

```bash
./mvnw test
```

Runs the test phase only. Useful when you do not need the full `verify` plugin chain.

### Full CI-equivalent local run

```bash
./mvnw clean verify
```

This is the closest match to the `Verify` GitHub Actions workflow and also triggers formatting, static checks, and JaCoCo verification.

### Targeted runs

```bash
./mvnw -Dtest=ProjectServiceImplTest test
./mvnw -Dtest=ProjectResourceTest,AttendanceResourceTest test
```

### Native profile

```bash
./mvnw verify -Dnative
```

The profile exists in `pom.xml`, but dedicated native integration tests are not currently present.

## JaCoCo constraints and coverage expectations

Coverage is enforced in [`pom.xml`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/pom.xml) through both `quarkus-jacoco` and `jacoco-maven-plugin`.

Current behavior:

- `jacoco.exec` and `jacoco-quarkus.exec` are merged into `target/jacoco-merged.exec`
- the report is generated during `verify`
- coverage is checked during `verify`
- the rule is package-level, not project-wide global average

Current enforced rule:

- **minimum instruction coverage per package: `0.85`**

Included packages in the rule:

- `br/org/catolicasc/pug/academic/**`
- `br/org/catolicasc/pug/geo/**`
- `br/org/catolicasc/pug/identity/**`
- `br/org/catolicasc/pug/partner/**`
- `br/org/catolicasc/pug/project/**`
- `br/org/catolicasc/pug/shared/**`

Practical implication:

- a new package with weak coverage can fail `verify` even if the overall project still looks well tested
- adding a new feature without module-level tests is likely to break CI

## Common pitfalls

- `./mvnw clean verify` is not read-only: `spotless:apply` is bound to `validate`, so source files may be reformatted automatically.
- Resource tests that seed data outside a transaction can fail intermittently because setup was not flushed.
- Tests that hit authenticated endpoints often need both `@TestSecurity` and `AuthService` stubbing, because some services read the current account directly.
- Search tests should assert page metadata as well as content, because paging is part of the API contract.
- If you invent raw FK values instead of using `TestDataFactory`, you can easily break UUIDv7 validation or FK assumptions.
- Local tests require a working container runtime because Dev Services is enabled in the test profile.
- There is no current `@QuarkusIntegrationTest` coverage, so native-mode regressions are less protected than JVM-mode regressions.

## Links

- [Back to README](https://github.com/Plataforma-Universidade-Gratuita/pug-docs/blob/main/pug-service/README.md)
