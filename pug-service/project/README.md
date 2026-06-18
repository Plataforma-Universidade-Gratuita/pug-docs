# Project Module

The `project` package is the execution-side module of `pug-service`. It manages partner projects, former-student enrollments, attendance validation, and the area-of-expertise links that connect academic eligibility to project participation.

## Module purpose

- Expose the project workflow APIs under `/v1/projects/*` and the academic-facing association API under `/v1/academic/areas-of-expertise/{areaOfExpertiseId}/projects`.
- Model the lifecycle rules for `Project`, `Enrollment`, and `Attendance`.
- Keep completed hours synchronized between projects and former students when attendances are validated.
- Gate enrollments by project status and area-of-expertise compatibility.
- Publish audit events after create, update, and delete operations.

## Main responsibilities

- 🚀 Manage [`Project`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/domain/Project.java) aggregates through [`ProjectServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectServiceImpl.java).
- 🎓 Manage [`Enrollment`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/domain/Enrollment.java) rows keyed by `(projectId, formerStudentId)` through [`EnrollmentsServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/service/impl/EnrollmentsServiceImpl.java).
- ✅ Manage [`Attendance`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/domain/Attendance.java) creation and QR validation through [`AttendancesServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/service/impl/AttendancesServiceImpl.java).
- 🧩 Manage project-to-area links through [`ProjectAreaOfExpertiseServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectAreaOfExpertiseServiceImpl.java) and [`ProjectAreaOfExpertiseReadServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectAreaOfExpertiseReadServiceImpl.java).
- 📖 Serve joined read models and paginated search results through the query implementations under [`infra/read/impl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/infra/read/impl).

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

Concrete request examples live in [`requests/project/project`](../../../pug-service/requests/project/project). The status endpoint accepts a raw JSON enum string, not an object. [Start Project.bru](<../../../pug-service/requests/project/project/Start Project.bru>) sends:

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

Concrete request behavior from [Create Enrollment.bru](<../../../pug-service/requests/project/enrollment/Create Enrollment.bru>): the endpoint has no JSON body. Admin callers can provide `formerStudentId` as a query parameter; former-student callers use the authenticated account when the query parameter is absent.

### Attendances

| Endpoint | Purpose | Access |
| --- | --- | --- |
| `GET /v1/projects/attendances/{id}` | Load one attendance by UUIDv7. | Any authenticated user |
| `GET /v1/projects/attendances` | List all attendances, or restrict by `ids`. | Any authenticated user |
| `POST /v1/projects/attendances/search?page=&size=` | Paginated attendance search. | Any authenticated user |
| `POST /v1/projects/attendances` | Create a waiting attendance. | Any authenticated user |
| `PATCH /v1/projects/attendances/{id}/validate` | Validate an attendance as `PRESENT` or `ABSENT`. | Any authenticated user |
| `DELETE /v1/projects/attendances/{id}` | Hard-delete one attendance. | Any authenticated user |

Important nuance from the code: [`AttendancesResource`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/presenter/AttendancesResource.java) is only annotated with `@Authenticated`. There are no `@RolesAllowed` checks on create, search, or delete. Validation does enforce a service-layer rule that rejects current `FORMER_STUDENT` accounts through [`AuthService.requireCurrentAccountNotOfType(...)`](../../../pug-service/src/main/java/br/org/catolicasc/pug/identity/service/AuthService.java).

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

- The `projects` table enforces `UNIQUE (entity_id, name)` plus non-negative hour checks in [`V010__create_projects_table.sql`](../../../pug-service/src/main/resources/db/migration/V010__create_projects_table.sql).
- [`ProjectServiceImpl.save(...)`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectServiceImpl.java) blocks current `FORMER_STUDENT` accounts, resolves the creator from `AuthService.getCurrentAccountId()`, and always creates a project in `PLANNED` state.
- [`ProjectServiceImpl.delete(...)`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectServiceImpl.java) refuses deletion when any enrollment exists and clears project-area links before removing the project row.
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
- [`AttendancePresenter`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/presenter/mappers/AttendancePresenter.java) returns `qrValidationInfo.qrValidationHash` in canonical and search responses. That is the current presenter contract in code.
- Search ordering is concrete in the JPQL query layer:
  - projects: `order by p.name asc`
  - enrollments: `order by en.createdAt desc`
  - attendances: `order by a.createdAt desc`
- Paging defaults to `page=0&size=25`. Query tests also cover the repository convention where `size=1` behaves as fetch-all through the shared paging utilities.
- Seed/test data for this module exists in [`V018__seed_test_data.sql`](../../../pug-service/src/main/resources/db/migration/V018__seed_test_data.sql). Concrete examples include projects such as `Portal Comunitario`, `Laboratorio de Acessibilidade`, `Oficina UX Social`, `Trilha de Dados Aplicados`, and `Rede de Cuidado Comunitario`.
- Scheduled jobs or message consumers: Not found in the current codebase. Checked `src/main/java/br/org/catolicasc/pug/project` for `@Scheduled`, `@Incoming`, `@Outgoing`, and `@ConsumeEvent` and found none.

## Important classes and files

- REST resources:
  - [`ProjectsResource`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/presenter/ProjectsResource.java)
  - [`EnrollmentsResource`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/presenter/EnrollmentsResource.java)
  - [`AttendancesResource`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/presenter/AttendancesResource.java)
  - [`ProjectsAreasOfExpertiseResource`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/presenter/ProjectsAreasOfExpertiseResource.java)
  - [`AreasOfExpertiseProjectsResource`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/presenter/AreasOfExpertiseProjectsResource.java)
- Write services:
  - [`ProjectServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectServiceImpl.java)
  - [`EnrollmentsServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/service/impl/EnrollmentsServiceImpl.java)
  - [`AttendancesServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/service/impl/AttendancesServiceImpl.java)
  - [`ProjectAreaOfExpertiseServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectAreaOfExpertiseServiceImpl.java)
- Read side:
  - [`ProjectQueriesImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/infra/read/impl/ProjectQueriesImpl.java)
  - [`EnrollmentsQueriesImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/infra/read/impl/EnrollmentsQueriesImpl.java)
  - [`AttendancesQueriesImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/infra/read/impl/AttendancesQueriesImpl.java)
  - [`ProjectAreaOfExpertiseReadServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectAreaOfExpertiseReadServiceImpl.java)
- Domain and value objects:
  - [`Project`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/domain/Project.java)
  - [`Enrollment`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/domain/Enrollment.java)
  - [`Attendance`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/domain/Attendance.java)
  - [`ProjectAreaOfExpertise`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/domain/ProjectAreaOfExpertise.java)
  - [`EnrollmentIdentifier`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/domain/vos/EnrollmentIdentifier.java)
  - [`ProjectInfo`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/domain/vos/ProjectInfo.java)
  - [`AttendanceInfo`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/domain/vos/AttendanceInfo.java)
  - [`QrValidationInfo`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/domain/vos/QrValidationInfo.java)
- Schema and request examples:
  - [`V010__create_projects_table.sql`](../../../pug-service/src/main/resources/db/migration/V010__create_projects_table.sql)
  - [`V011__create_projects_areas_of_expertise_table.sql`](../../../pug-service/src/main/resources/db/migration/V011__create_projects_areas_of_expertise_table.sql)
  - [`V012__create_enrollments_table.sql`](../../../pug-service/src/main/resources/db/migration/V012__create_enrollments_table.sql)
  - [`V013__create_attendances_table.sql`](../../../pug-service/src/main/resources/db/migration/V013__create_attendances_table.sql)
  - [`V018__seed_test_data.sql`](../../../pug-service/src/main/resources/db/migration/V018__seed_test_data.sql)
  - [`requests/project`](../../../pug-service/requests/project)

## Dependencies on other modules

- [`shared`](../../../pug-service/src/main/java/br/org/catolicasc/pug/shared): API envelopes, paging, UUIDv7 validation, locale handling, audit publishing, and shared exceptions.
- [`partner`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner): [`ProjectServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/service/impl/ProjectServiceImpl.java) validates project ownership context through [`EntitiesService`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/service/EntitiesService.java), and the project read model joins `EntityEntity` to expose entity names.
- [`academic`](../../../pug-service/src/main/java/br/org/catolicasc/pug/academic): enrollment creation and attendance validation call [`FormerStudentsService`](../../../pug-service/src/main/java/br/org/catolicasc/pug/academic/service/FormerStudentsService.java); association reads call [`AreasOfExpertiseQueries`](../../../pug-service/src/main/java/br/org/catolicasc/pug/academic/infra/read/AreasOfExpertiseQueries.java).
- [`identity`](../../../pug-service/src/main/java/br/org/catolicasc/pug/identity): resources and services use [`AuthService`](../../../pug-service/src/main/java/br/org/catolicasc/pug/identity/service/AuthService.java); read models join `AccountEntity` and `UserEntity` to expose creator, student, and validator names/emails.

### Inbound use from other modules

- [`AreasOfExpertiseServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/academic/service/impl/AreasOfExpertiseServiceImpl.java) uses [`ProjectAreaOfExpertiseService`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/service/ProjectAreaOfExpertiseService.java) to clear links before deleting an area of expertise.
- [`FormerStudentsServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/academic/service/impl/FormerStudentsServiceImpl.java) uses [`EnrollmentsService`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/service/EnrollmentsService.java).
- [`EntitiesServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/service/impl/EntitiesServiceImpl.java) and [`StaffServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/service/impl/StaffServiceImpl.java) depend on project services for deletion and validation guards.
- [`AdminsServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/identity/service/impl/AdminsServiceImpl.java) depends on [`ProjectService`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/service/ProjectService.java).

## How to test the module

- Tests live under [`src/test/java/br/org/catolicasc/pug/project`](../../../pug-service/src/test/java/br/org/catolicasc/pug/project).
- Representative test groups:
  - domain and lifecycle rules: [`ProjectTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/project/domain/ProjectTest.java), [`EnrollmentTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/project/domain/EnrollmentTest.java), [`AttendanceTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/project/domain/AttendanceTest.java)
  - read/query coverage: [`ProjectQueriesImplTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/project/infra/read/impl/ProjectQueriesImplTest.java), [`EnrollmentQueriesImplTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/project/infra/read/impl/EnrollmentQueriesImplTest.java), [`AttendanceQueriesImplTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/project/infra/read/impl/AttendanceQueriesImplTest.java)
  - resource coverage: [`ProjectResourceTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/project/presenter/ProjectResourceTest.java), [`EnrollmentResourceTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/project/presenter/EnrollmentResourceTest.java), [`AttendanceResourceTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/project/presenter/AttendanceResourceTest.java), [`ProjectAreaOfExpertiseResourceTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/project/presenter/ProjectAreaOfExpertiseResourceTest.java)
  - service coverage: [`ProjectServiceImplTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/project/service/impl/ProjectServiceImplTest.java), [`EnrollmentsServiceImplTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/project/service/impl/EnrollmentsServiceImplTest.java), [`AttendanceServiceImplTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/project/service/impl/AttendanceServiceImplTest.java), [`ProjectAreaOfExpertiseServiceImplTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/project/service/impl/ProjectAreaOfExpertiseServiceImplTest.java)
- Commands:
  - full suite: `./mvnw test`
  - module-focused examples: `./mvnw -Dtest=ProjectTest,EnrollmentTest,AttendanceTest,ProjectQueriesImplTest,EnrollmentQueriesImplTest,AttendanceQueriesImplTest,ProjectResourceTest,EnrollmentResourceTest,AttendanceResourceTest,ProjectAreaOfExpertiseResourceTest,ProjectServiceImplTest,EnrollmentsServiceImplTest,AttendanceServiceImplTest,ProjectAreaOfExpertiseServiceImplTest test`

## Links

- [Project architecture](./ARCHITECTURE.md)
- [Back to pug-service docs](../README.md)
