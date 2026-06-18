# Academic Module

The `academic` package owns the university-side reference data and former-student records in `pug-service`. Unlike `geo` or `partner`, this package contains three related academic slices in one module:

- `areas-of-expertise`
- `courses`
- `former-students`

## Module purpose

- Maintain the academic catalog used by the rest of the system: areas of expertise and courses.
- Manage former-student academic records linked to identity accounts.
- Expose academic REST APIs under `/v1/academic/*`.
- Enforce academic rules before project enrollments, attendance validation, and deletion flows proceed.
- Publish audit events after create, update, and delete operations.

## Main responsibilities

- Manage [`AreaOfExpertise`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/domain/AreaOfExpertise.java) through [`AreasOfExpertiseServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/service/impl/AreasOfExpertiseServiceImpl.java).
- Manage [`Course`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/domain/Course.java) through [`CoursesServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/service/impl/CoursesServiceImpl.java).
- Manage [`FormerStudent`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/domain/FormerStudent.java) plus its academic value objects through [`FormerStudentsServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/service/impl/FormerStudentsServiceImpl.java).
- Provide dedicated read projections and search queries for all three slices.
- Bridge academic records with identity accounts and project enrollments.

## Public API

### Areas of expertise

| Endpoint | Purpose | Access |
| --- | --- | --- |
| `GET /v1/academic/areas-of-expertise/{id}` | Load one area of expertise by UUIDv7. | Any authenticated user |
| `GET /v1/academic/areas-of-expertise` | List all areas, or restrict by `ids`. | Any authenticated user |
| `POST /v1/academic/areas-of-expertise/search?page=&size=` | Paginated name search. | Any authenticated user |
| `POST /v1/academic/areas-of-expertise` | Create an area of expertise. | `ADMIN` |
| `PUT /v1/academic/areas-of-expertise/{id}` | Update the area name. | `ADMIN` |
| `DELETE /v1/academic/areas-of-expertise/{id}` | Hard-delete the area after course checks. | `ADMIN` |

### Courses

| Endpoint | Purpose | Access |
| --- | --- | --- |
| `GET /v1/academic/courses/{id}` | Load one course by UUIDv7. | Any authenticated user |
| `GET /v1/academic/courses` | List all courses, or restrict by `ids`. | Any authenticated user |
| `POST /v1/academic/courses/search?page=&size=` | Paginated search by `name` and `areaOfExpertiseIds`. | Any authenticated user |
| `POST /v1/academic/courses` | Create a course. | `ADMIN` |
| `PUT /v1/academic/courses/{id}` | Update course name and/or area of expertise. | `ADMIN` |
| `DELETE /v1/academic/courses/{id}` | Hard-delete the course after former-student checks. | `ADMIN` |

### Former students

| Endpoint | Purpose | Access |
| --- | --- | --- |
| `GET /v1/academic/former-students/{id}` | Load one former-student record by linked account UUIDv7. | Any authenticated user |
| `GET /v1/academic/former-students/me` | Resolve the record linked to the current authenticated account. | Any authenticated user |
| `GET /v1/academic/former-students` | List all former students, or restrict by linked account IDs. | Any authenticated user |
| `POST /v1/academic/former-students/search?page=&size=` | Paginated search by identity, academic registration, campus, period, timestamps, course, and area of expertise. | Any authenticated user |
| `POST /v1/academic/former-students` | Create one former-student record and its linked identity account. | `ADMIN` |
| `POST /v1/academic/former-students/bulk` | Create multiple former-student records from a JSON array. | `ADMIN` |
| `PUT /v1/academic/former-students/{id}` | Update former-student academic data and linked account data. | `ADMIN` |
| `PATCH /v1/academic/former-students/{id}/status` | Toggle the linked account active flag. | `ADMIN` |
| `DELETE /v1/academic/former-students/{id}` | Hard-delete the academic record and linked account. | `ADMIN` |

### Internal service APIs used by other modules

These are real public service methods in code, but they are **not** exposed as REST endpoints:

- [`FormerStudentsService.addCompletedHours(...)`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/service/FormerStudentsService.java)
- [`FormerStudentsService.removeCompletedHours(...)`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/service/FormerStudentsService.java)
- [`FormerStudentsService.getAreaOfExpertise(...)`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/service/FormerStudentsService.java)

Those methods are called from the `project` module when attendances are validated and when enrollments are created.

## Concrete behavior from the repository

- Area-of-expertise writes are admin-only, and deletion is blocked when [`CoursesService.existsAnyByAreaOfExpertiseId(...)`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/service/CoursesService.java) reports linked courses.
- Successful area deletion also removes project-to-area links through [`ProjectAreaOfExpertiseService.deleteAllByAreaOfExpertiseId(...)`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/ProjectAreaOfExpertiseService.java).
- Course writes validate the referenced area of expertise through [`AreasOfExpertiseService.getById(...)`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/service/AreasOfExpertiseService.java).
- Course deletion is blocked when [`FormerStudentsService.existsAnyByCourseId(...)`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/service/FormerStudentsService.java) finds enrolled former students.
- Former-student creation builds an `AccountType.FORMER_STUDENT` account command with a `null` password through [`FormerStudentPresenter`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/presenter/mappers/FormerStudentPresenter.java).
- Former-student bulk creation validates all distinct `courseId` values first, rejects duplicate academic registrations in the payload or database, then creates accounts in bulk and persists the academic rows.
- Former-student search defaults to `activeOnly = true` and `includeConcluded = false` when the request omits those flags or the request body is missing.
- `FormerStudentPresenter` calculates response-only fields such as `missingHours`, `progress`, `remainingDays`, and a localized `remainingDaysFormatted` string.
- There is no REST endpoint for adding or removing completed hours. Those service methods are called from [`AttendancesServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/project/service/impl/AttendancesServiceImpl.java).

## Example payloads

A bulk former-student request exists in [`requests/academic`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/requests/academic/former-student/Create%2520Former%2520Students%2520Bulk.bru):

```json
[
  {
    "cpf": "52998224059",
    "name": "Maria Oliveira",
    "email": "maria.oliveira@aluno.com",
    "academicRegistration": "20261002",
    "campus": "JOINVILLE",
    "courseId": "00000000-0000-0000-0000-000000000001",
    "requiredHours": 150.00,
    "startDate": "2026-03-15",
    "dueDate": "2028-12-15"
  }
]
```

A former-student search request can combine academic and identity filters:

```json
{
  "name": "Joao",
  "email": "joao",
  "academicRegistration": "2026",
  "campi": ["JOINVILLE"],
  "periodFrom": "2026-01-01",
  "periodTo": "2028-12-31",
  "includeConcluded": false,
  "activeOnly": true,
  "courseIds": ["00000000-0000-0000-0000-000000000001"],
  "areaOfExpertiseIds": ["00000000-0000-0000-0000-000000000001"]
}
```

Concrete seed data lives in:

- [`V017__seed_areas_of_expertise_and_courses.sql`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/resources/db/migration/V017__seed_areas_of_expertise_and_courses.sql)
- [`V018__seed_test_data.sql`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/resources/db/migration/V018__seed_test_data.sql)

Examples from those files include courses such as `Engenharia de Software`, `Design`, and `Direito`, plus former-student emails under `@former-student.pug.br`.

## Important classes and files

- Areas of expertise:
  - [`AreasOfExpertiseResource`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/presenter/AreasOfExpertiseResource.java)
  - [`AreasOfExpertiseServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/service/impl/AreasOfExpertiseServiceImpl.java)
  - [`AreasOfExpertiseQueriesImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/infra/read/impl/AreasOfExpertiseQueriesImpl.java)
- Courses:
  - [`CoursesResource`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/presenter/CoursesResource.java)
  - [`CoursesServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/service/impl/CoursesServiceImpl.java)
  - [`CoursesQueriesImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/infra/read/impl/CoursesQueriesImpl.java)
- Former students:
  - [`FormerStudentsResource`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/presenter/FormerStudentsResource.java)
  - [`FormerStudentsServiceImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/service/impl/FormerStudentsServiceImpl.java)
  - [`FormerStudentsQueriesImpl`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/infra/read/impl/FormerStudentsQueriesImpl.java)
  - [`FormerStudentPresenter`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/presenter/mappers/FormerStudentPresenter.java)
  - [`FormerStudentProcessor`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/service/utils/FormerStudentProcessor.java)
- Persistence and schema:
  - [`AreaOfExpertiseEntity`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/infra/persistence/AreaOfExpertiseEntity.java)
  - [`CourseEntity`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/infra/persistence/CourseEntity.java)
  - [`FormerStudentEntity`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/infra/persistence/FormerStudentEntity.java)
  - [`V007__create_areas_of_expertise_table.sql`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/resources/db/migration/V007__create_areas_of_expertise_table.sql)
  - [`V008__create_courses_table.sql`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/resources/db/migration/V008__create_courses_table.sql)
  - [`V009__create_former_students_table.sql`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/resources/db/migration/V009__create_former_students_table.sql)

## Dependencies on other modules

- [`shared`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/main/java/br/org/catolicasc/pug/shared): response envelopes, pagination, locale handling, `Campi`, UUIDv7 validation, audit publishing, and shared persistence/search helpers.
- [`identity`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/main/java/br/org/catolicasc/pug/identity): former-student create/update/status/delete flows delegate to `AccountsService`, `/me` uses `AuthService`, and presenter mapping composes identity account responses.
- [`project`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/main/java/br/org/catolicasc/pug/project): area deletion clears project-area associations; former-student deletion is blocked by enrollments; attendance validation updates completed hours; enrollment creation checks area-of-expertise compatibility.

### Inbound use from other modules

- The `project` module imports academic domain/services such as [`FormerStudentsService`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/service/FormerStudentsService.java) and [`AreaOfExpertise`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/domain/AreaOfExpertise.java).
- The `project` presenter layer also reuses academic DTOs such as [`AreaOfExpertiseResponse`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/presenter/dtos/areasofexpertise/AreaOfExpertiseResponse.java) and [`FormerStudentSimpleComplexSearchResponse`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/main/java/br/org/catolicasc/pug/academic/presenter/dtos/formerstudents/FormerStudentSimpleComplexSearchResponse.java).

## How to test the module

- Tests live under [`src/test/java/br/org/catolicasc/pug/academic`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/tree/main/src/test/java/br/org/catolicasc/pug/academic).
- Representative test groups:
  - domain/value objects: [`AreaOfExpertiseTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/academic/domain/AreaOfExpertiseTest.java), [`CourseTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/academic/domain/CourseTest.java), [`FormerStudentTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/academic/domain/FormerStudentTest.java), [`CounterpartHoursTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/academic/domain/vos/CounterpartHoursTest.java), [`PeriodTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/academic/domain/vos/PeriodTest.java)
  - read/query coverage: [`AreasOfExpertiseQueriesImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/academic/infra/read/impl/AreasOfExpertiseQueriesImplTest.java), [`CoursesQueriesImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/academic/infra/read/impl/CoursesQueriesImplTest.java), [`FormerStudentsQueriesImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/academic/infra/read/impl/FormerStudentsQueriesImplTest.java)
  - resource/service coverage: [`AreasOfExpertiseResourceTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/academic/presenter/AreasOfExpertiseResourceTest.java), [`CoursesResourceTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/academic/presenter/CoursesResourceTest.java), [`FormerStudentsResourceTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/academic/presenter/FormerStudentsResourceTest.java), [`AreasOfExpertiseServiceImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/academic/service/impl/AreasOfExpertiseServiceImplTest.java), [`CoursesServiceImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/academic/service/impl/CoursesServiceImplTest.java), [`FormerStudentsServiceImplTest`](https://github.com/Plataforma-Universidade-Gratuita/pug-service/blob/main/src/test/java/br/org/catolicasc/pug/academic/service/impl/FormerStudentsServiceImplTest.java)
- Commands:
  - full suite: `./mvnw test`
  - module-focused examples: `./mvnw -Dtest=AreaOfExpertiseTest,CourseTest,FormerStudentTest,AreasOfExpertiseQueriesImplTest,CoursesQueriesImplTest,FormerStudentsQueriesImplTest,AreasOfExpertiseResourceTest,CoursesResourceTest,FormerStudentsResourceTest,AreasOfExpertiseServiceImplTest,CoursesServiceImplTest,FormerStudentsServiceImplTest test`

## Links

- [Academic architecture](./ARCHITECTURE.md)
- [Back to pug-service docs](../README.md)
