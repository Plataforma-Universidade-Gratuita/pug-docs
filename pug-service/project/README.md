# Project Module

The `project` package is the execution-side module of `pug-service`. It manages partner projects, former-student enrollments, attendance validation, and the area-of-expertise links that connect academic eligibility to project participation.

## Module purpose

- Expose the project workflow APIs under `/v1/projects/*` and the academic-facing association API under `/v1/academic/areas-of-expertise/{areaOfExpertiseId}/projects`.
- Model the lifecycle rules for `Project`, `Enrollment`, and `Attendance`.
- Keep completed hours synchronized between projects and former students when attendances are validated.
- Gate enrollments by project status and area-of-expertise compatibility.
- Publish audit events after create, update, and delete operations.

## Main responsibilities

- 🚀 Manage [`Project`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/domain/Project.java) aggregates through [`ProjectServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectServiceImpl.java).
- 🎓 Manage [`Enrollment`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/domain/Enrollment.java) rows keyed by `(projectId, formerStudentId)` through [`EnrollmentsServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/impl/EnrollmentsServiceImpl.java).
- ✅ Manage [`Attendance`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/domain/Attendance.java) creation and QR validation through [`AttendancesServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/impl/AttendancesServiceImpl.java).
- 🧩 Manage project-to-area links through [`ProjectAreaOfExpertiseServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectAreaOfExpertiseServiceImpl.java) and [`ProjectAreaOfExpertiseReadServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectAreaOfExpertiseReadServiceImpl.java).
- 📖 Serve joined read models and paginated search results through the query implementations under [`infra/read/impl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/main/java/br/org/catolicasc/pug/project/infra/read/impl).

## Public API

### Projects

| Endpoint | Purpose | Access |
| --- | --- | --- |
| `GET /v1/projects/{id}` | Load one project by UUIDv7. | Any authenticated user |
| `GET /v1/projects` | List all projects, or restrict by `ids`. | Any authenticated user |
| `GET /v1/projects/entities/{entityId}` | List projects for one partner entity. | Any authenticated user |
| `GET /v1/projects/creators/{createdById}` | List projects created by one account. | Any authenticated user |
| `POST /v1/projects/search?page=&size=` | Paginated project search. | Any authenticated user |
| `POST /v1/projects` | Create a project. | `ADMIN`, `PARTNER` |
| `PUT /v1/projects/{id}` | Update name, description, capacity, or offered hours. | `ADMIN`, `PARTNER` |
| `PATCH /v1/projects/{id}/status` | Change lifecycle status. | `ADMIN`, `PARTNER` |
| `DELETE /v1/projects/{id}` | Hard-delete a project after enrollment checks. | `ADMIN`, `PARTNER` |

Concrete request examples live in [`requests/project/project`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/requests/project/project). The status endpoint accepts a raw JSON enum string, not an object. [Start Project.bru](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/requests/project/project/Start%20Project.bru) sends:

```json
"IN_PROGRESS"
```

### Enrollments

| Endpoint | Purpose | Access |
| --- | --- | --- |
| `GET /v1/projects/{projectId}/enrollments/{formerStudentId}` | Load one enrollment by composite key. | `ADMIN`, `PARTNER` |
| `GET /v1/projects/{projectId}/enrollments/me` | Load the current former student's enrollment in one project. | `FORMER_STUDENT` |
| `GET /v1/projects/enrollments` | List all enrollments, or filter by `projectId` or `formerStudentId`. | `ADMIN`, `PARTNER` |
| `GET /v1/projects/enrollments/me` | List every enrollment for the current former student. | `FORMER_STUDENT` |
| `POST /v1/projects/{projectId}/enrollments` | Create an enrollment. | `ADMIN`, `FORMER_STUDENT` |
| `POST /v1/projects/enrollments/search?page=&size=` | Paginated enrollment search. | `ADMIN`, `PARTNER` |
| `PATCH /v1/projects/{projectId}/enrollments/{formerStudentId}` | Administrative enrollment transition. | `ADMIN`, `PARTNER` |
| `PATCH /v1/projects/{projectId}/enrollments/me` | Former-student self-service exit. | `FORMER_STUDENT` |
| `DELETE /v1/projects/{projectId}/enrollments/{formerStudentId}` | Hard-delete one enrollment. | `ADMIN` |

Concrete request behavior from [Create Enrollment.bru](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/requests/project/enrollment/Create%20Enrollment.bru): the endpoint has no JSON body. Admin callers can provide `formerStudentId` as a query parameter; former-student callers use the authenticated account when the query parameter is absent.

### Attendances

| Endpoint | Purpose | Access |
| --- | --- | --- |
| `GET /v1/projects/attendances/{id}` | Load one attendance by UUIDv7. | Any authenticated user |
| `GET /v1/projects/attendances` | List all attendances, or restrict by `ids`. | Any authenticated user |
| `POST /v1/projects/attendances/search?page=&size=` | Paginated attendance search. | Any authenticated user |
| `POST /v1/projects/attendances` | Create a waiting attendance. | Any authenticated user |
| `PATCH /v1/projects/attendances/{id}/validate` | Validate an attendance as `PRESENT` or `ABSENT`. | Any authenticated user |
| `DELETE /v1/projects/attendances/{id}` | Hard-delete one attendance. | Any authenticated user |

Important nuance from the code: [`AttendancesResource`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/presenter/AttendancesResource.java) is only annotated with `@Authenticated`. There are no `@RolesAllowed` checks on create, search, or delete. Validation does enforce a service-layer rule that rejects current `FORMER_STUDENT` accounts through [`AuthService.requireCurrentAccountNotOfType(...)`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/identity/service/AuthService.java).

### Project and area-of-expertise associations

| Endpoint | Purpose | Access |
| --- | --- | --- |
| `GET /v1/projects/{projectId}/areas-of-expertise` | List the academic areas linked to one project. | Any authenticated user |
| `POST /v1/projects/{projectId}/areas-of-expertise` | Create one or more project-area links. | `ADMIN`, `PARTNER` |
| `DELETE /v1/projects/{projectId}/areas-of-expertise/{areaOfExpertiseId}` | Delete one project-area link. | `ADMIN`, `PARTNER` |
| `DELETE /v1/projects/{projectId}/areas-of-expertise` | Delete all area links for one project. | `ADMIN`, `PARTNER` |
| `GET /v1/academic/areas-of-expertise/{areaOfExpertiseId}/projects` | List projects linked to one academic area. | Any authenticated user |
| `DELETE /v1/academic/areas-of-expertise/{areaOfExpertiseId}/projects` | Delete all links for one academic area. | `ADMIN`, `PARTNER` |

## Concrete behavior from the repository

- The `projects` table enforces `UNIQUE (entity_id, name)` plus non-negative hour checks in [`V010__create_projects_table.sql`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/resources/db/migration/V010__create_projects_table.sql).
- [`ProjectServiceImpl.save(...)`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectServiceImpl.java) blocks current `FORMER_STUDENT` accounts, resolves the creator from `AuthService.getCurrentAccountId()`, and always creates a project in `PLANNED` state.
- [`ProjectServiceImpl.delete(...)`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectServiceImpl.java) refuses deletion when any enrollment exists and clears project-area links before removing the project row.
- Project status transitions propagate enrollment changes:
  - `CANCELED` propagates `CANCELED`
  - `COMPLETED` propagates `COMPLETED`
  - `ON_HOLD` turns approved enrollments into `ON_HOLD`
  - returning from hold reopens held enrollments to `APPROVED`
- Enrollment creation validates all of the following before persisting the row:
  - the project exists
  - the project can still receive enrollments
  - the former student can still enroll
  - the former student's area of expertise matches one of the project's associated areas
- Administrative enrollment updates only allow `REJECTED`, `APPROVED`, `REMOVED`, and `COMPLETED`. Former-student self-service updates only allow `EXITED`.
- Closing an enrollment with `CANCELED`, `COMPLETED`, `EXITED`, or `REMOVED` deletes waiting attendances for that enrollment.
- Attendance creation requires an approved enrollment and always starts in `WAITING` state.
- Attendance validation compares the submitted `qrValidationHash` with the stored hash. Marking an attendance `PRESENT` requires the project to be in progress and adds completed hours to both the former student and the project. Changing a previously `PRESENT` attendance to `ABSENT` removes those hours again.
- [`AttendancePresenter`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/presenter/mappers/AttendancePresenter.java) returns `qrValidationInfo.qrValidationHash` in canonical and search responses. That is the current presenter contract in code.
- Search ordering is concrete in the JPQL query layer:
  - projects: `order by p.name asc`
  - enrollments: `order by en.createdAt desc`
  - attendances: `order by a.createdAt desc`
- Paging defaults to `page=0&size=25`. Query tests also cover the repository convention where `size=1` behaves as fetch-all through the shared paging utilities.
- Seed/test data for this module exists in [`V018__seed_test_data.sql`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/resources/db/migration/V018__seed_test_data.sql). Concrete examples include projects such as `Portal Comunitario`, `Laboratorio de Acessibilidade`, `Oficina UX Social`, `Trilha de Dados Aplicados`, and `Rede de Cuidado Comunitario`.
- Scheduled jobs or message consumers: none in the `project` package.

## Important classes and files

- REST resources:
  - [`ProjectsResource`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/presenter/ProjectsResource.java)
  - [`EnrollmentsResource`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/presenter/EnrollmentsResource.java)
  - [`AttendancesResource`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/presenter/AttendancesResource.java)
  - [`ProjectsAreasOfExpertiseResource`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/presenter/ProjectsAreasOfExpertiseResource.java)
  - [`AreasOfExpertiseProjectsResource`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/presenter/AreasOfExpertiseProjectsResource.java)
- Write services:
  - [`ProjectServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectServiceImpl.java)
  - [`EnrollmentsServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/impl/EnrollmentsServiceImpl.java)
  - [`AttendancesServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/impl/AttendancesServiceImpl.java)
  - [`ProjectAreaOfExpertiseServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectAreaOfExpertiseServiceImpl.java)
- Read side:
  - [`ProjectQueriesImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/infra/read/impl/ProjectQueriesImpl.java)
  - [`EnrollmentsQueriesImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/infra/read/impl/EnrollmentsQueriesImpl.java)
  - [`AttendancesQueriesImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/infra/read/impl/AttendancesQueriesImpl.java)
  - [`ProjectAreaOfExpertiseReadServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectAreaOfExpertiseReadServiceImpl.java)
- Domain and value objects:
  - [`Project`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/domain/Project.java)
  - [`Enrollment`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/domain/Enrollment.java)
  - [`Attendance`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/domain/Attendance.java)
  - [`ProjectAreaOfExpertise`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/domain/ProjectAreaOfExpertise.java)
  - [`EnrollmentIdentifier`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/domain/vos/EnrollmentIdentifier.java)
  - [`ProjectInfo`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/domain/vos/ProjectInfo.java)
  - [`AttendanceInfo`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/domain/vos/AttendanceInfo.java)
  - [`QrValidationInfo`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/domain/vos/QrValidationInfo.java)
- Schema and request examples:
  - [`V010__create_projects_table.sql`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/resources/db/migration/V010__create_projects_table.sql)
  - [`V011__create_projects_areas_of_expertise_table.sql`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/resources/db/migration/V011__create_projects_areas_of_expertise_table.sql)
  - [`V012__create_enrollments_table.sql`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/resources/db/migration/V012__create_enrollments_table.sql)
  - [`V013__create_attendances_table.sql`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/resources/db/migration/V013__create_attendances_table.sql)
  - [`V018__seed_test_data.sql`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/resources/db/migration/V018__seed_test_data.sql)
  - [`requests/project`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/requests/project)

## Dependencies on other modules

- [`shared`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/main/java/br/org/catolicasc/pug/shared): API envelopes, paging, UUIDv7 validation, locale handling, audit publishing, and shared exceptions.
- [`partner`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/main/java/br/org/catolicasc/pug/partner): [`ProjectServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectServiceImpl.java) validates project ownership context through [`EntitiesService`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/partner/service/EntitiesService.java), and the project read model joins `EntityEntity` to expose entity names.
- [`academic`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/main/java/br/org/catolicasc/pug/academic): enrollment creation and attendance validation call [`FormerStudentsService`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/service/FormerStudentsService.java); association reads call [`AreasOfExpertiseQueries`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/infra/read/AreasOfExpertiseQueries.java).
- [`identity`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/main/java/br/org/catolicasc/pug/identity): resources and services use [`AuthService`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/identity/service/AuthService.java); read models join `AccountEntity` and `UserEntity` to expose creator, student, and validator names/emails.

### Inbound use from other modules

- [`AreasOfExpertiseServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/service/impl/AreasOfExpertiseServiceImpl.java) uses [`ProjectAreaOfExpertiseService`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/ProjectAreaOfExpertiseService.java) to clear links before deleting an area of expertise.
- [`FormerStudentsServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/service/impl/FormerStudentsServiceImpl.java) uses [`EnrollmentsService`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/EnrollmentsService.java).
- [`EntitiesServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/partner/service/impl/EntitiesServiceImpl.java) and [`StaffServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/partner/service/impl/StaffServiceImpl.java) depend on project services for deletion and validation guards.
- [`AdminsServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/identity/service/impl/AdminsServiceImpl.java) depends on [`ProjectService`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/ProjectService.java).

## How to test the module

- Tests live under [`src/test/java/br/org/catolicasc/pug/project`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/test/java/br/org/catolicasc/pug/project).
- Representative test groups:
  - domain and lifecycle rules: [`ProjectTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/domain/ProjectTest.java), [`EnrollmentTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/domain/EnrollmentTest.java), [`AttendanceTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/domain/AttendanceTest.java)
  - read/query coverage: [`ProjectQueriesImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/infra/read/impl/ProjectQueriesImplTest.java), [`EnrollmentQueriesImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/infra/read/impl/EnrollmentQueriesImplTest.java), [`AttendanceQueriesImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/infra/read/impl/AttendanceQueriesImplTest.java)
  - resource coverage: [`ProjectResourceTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/presenter/ProjectResourceTest.java), [`EnrollmentResourceTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/presenter/EnrollmentResourceTest.java), [`AttendanceResourceTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/presenter/AttendanceResourceTest.java), [`ProjectAreaOfExpertiseResourceTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/presenter/ProjectAreaOfExpertiseResourceTest.java)
  - service coverage: [`ProjectServiceImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/service/impl/ProjectServiceImplTest.java), [`EnrollmentsServiceImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/service/impl/EnrollmentsServiceImplTest.java), [`AttendanceServiceImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/service/impl/AttendanceServiceImplTest.java), [`ProjectAreaOfExpertiseServiceImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/project/service/impl/ProjectAreaOfExpertiseServiceImplTest.java)
- Commands:
  - full suite: `./mvnw test`
  - module-focused examples: `./mvnw -Dtest=ProjectTest,EnrollmentTest,AttendanceTest,ProjectQueriesImplTest,EnrollmentQueriesImplTest,AttendanceQueriesImplTest,ProjectResourceTest,EnrollmentResourceTest,AttendanceResourceTest,ProjectAreaOfExpertiseResourceTest,ProjectServiceImplTest,EnrollmentsServiceImplTest,AttendanceServiceImplTest,ProjectAreaOfExpertiseServiceImplTest test`

## Links

- [Project architecture](https://github.com/Plataforma-Universidade-Gratuita/pug-docs/blob/main/pug-service/project/ARCHITECTURE.md)
- [Back to pug-service docs](https://github.com/Plataforma-Universidade-Gratuita/pug-docs/blob/main/pug-service/README.md)
