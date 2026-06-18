# Partner Module

The `partner` package owns the partner-organization side of `pug-service`. It manages partner organizations (`Entity`), the assignment of partner staff (`Staff`) to those organizations, and the REST APIs under `/v1/partners/entities` and `/v1/partners/staff`.

## Module purpose

- ?? Register and maintain partner organizations, including CNPJ, city, and address.
- ?? Attach `AccountType.PARTNER` accounts to partner organizations through the `staff` table.
- ?? Provide authenticated list and search endpoints for entities and staff.
- ?? Enforce cross-module rules before write operations reach the database.
- ?? Publish audit events after create, update, and delete operations.

## Main responsibilities

- Manage the `entities` table through the [`Entity`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/domain/Entity.java) aggregate and [`EntityRepositoryImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/infra/persistence/impl/EntityRepositoryImpl.java).
- Manage the `staff` table through the [`Staff`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/domain/Staff.java) aggregate and [`StaffRepositoryImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/infra/persistence/impl/StaffRepositoryImpl.java).
- Expose CRUD and search endpoints through [`EntitiesResource`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/presenter/EntitiesResource.java) and [`StaffResource`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/presenter/StaffResource.java).
- Validate CNPJ and aggregate invariants through [`Cnpj`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/domain/vos/Cnpj.java), [`EntityProcessor`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/service/utils/EntityProcessor.java), and [`StaffProcessor`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/service/utils/StaffProcessor.java).
- Orchestrate account provisioning and status updates through [`StaffServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/service/impl/StaffServiceImpl.java).

## Public API

### Entity endpoints

| Endpoint | Purpose | Access |
| --- | --- | --- |
| `GET /v1/partners/entities/{id}` | Load one partner entity by UUIDv7. | Any authenticated user |
| `GET /v1/partners/entities` | List all entities, or restrict by `ids`. | Any authenticated user |
| `POST /v1/partners/entities/search?page=&size=` | Paginated search by `name`, `cnpj`, `address`, `cityIds`, `dateFrom`, `dateTo`. | Any authenticated user |
| `POST /v1/partners/entities` | Create a partner entity. | `ADMIN`, `PARTNER` |
| `PUT /v1/partners/entities/{id}` | Partially update name, city, and address. | `ADMIN`, `PARTNER` |
| `DELETE /v1/partners/entities/{id}` | Hard-delete the entity after project and staff checks. | `ADMIN` |

### Staff endpoints

| Endpoint | Purpose | Access |
| --- | --- | --- |
| `GET /v1/partners/staff/{id}` | Load one staff assignment by account UUIDv7. | Any authenticated user |
| `GET /v1/partners/staff/me` | Resolve the current authenticated staff profile. | Any authenticated user |
| `GET /v1/partners/staff` | List all staff, or restrict by `ids`. | Any authenticated user |
| `POST /v1/partners/staff/search?page=&size=` | Paginated search by `name`, `cpf`, `email`, `entityIds`, `dateFrom`, `dateTo`, `activeOnly`. | Any authenticated user |
| `POST /v1/partners/staff` | Create a partner staff assignment and underlying account. | `ADMIN`, `PARTNER` |
| `PUT /v1/partners/staff/{id}` | Update account data and optionally transfer staff to another entity. | `ADMIN`, `PARTNER` |
| `PATCH /v1/partners/staff/{id}/status` | Toggle the linked account active flag. | `ADMIN`, `PARTNER` |
| `DELETE /v1/partners/staff/{id}` | Hard-delete the staff assignment and linked account. | `ADMIN` |

## Concrete behavior from the repository

- Entity search joins city data and returns a richer projection than the plain `GET` list. That behavior is implemented in [`EntitiesQueriesImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/infra/read/impl/EntitiesQueriesImpl.java) and mapped by [`EntityPresenter`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/presenter/mappers/EntityPresenter.java).
- Staff search defaults `activeOnly` to `true` when the request omits it. That default is applied in [`StaffResource`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/presenter/StaffResource.java).
- Staff creation builds an `AccountType.PARTNER` account command with a `null` password and then persists the partner assignment. The mapping lives in [`StaffPresenter`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/presenter/mappers/StaffPresenter.java).
- Entity deletion is blocked when [`ProjectService.existsAnyByEntityId(...)`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project/service/ProjectService.java) returns `true`, and successful deletion cascades to [`StaffService.deleteAllByEntityId(...)`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/service/StaffService.java).
- Staff deletion is blocked when the linked account still owns projects or validated attendances. The checks live in [`StaffServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/service/impl/StaffServiceImpl.java).
- Search filters use shared `JpaSearchUtils.containsClause(...)`, so the string filters are substring-based and follow the project-wide search conventions.

## Example payloads

A create-entity request exists in [`requests/partner`](../../../pug-service/requests/partner):

```json
{
  "cnpj": "12345678000195",
  "name": "Tech Corp Parceira",
  "cityId": "00000000-0000-0000-0000-000000000001",
  "address": "Rua da Tecnologia, 100"
}
```

A staff search request can combine multiple filters. The repository tests use the same pattern in [`StaffResourceTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/partner/presenter/StaffResourceTest.java):

```json
{
  "name": "Gabriel",
  "cpf": "123",
  "email": "gabr",
  "entityIds": ["11111111-1111-7111-8111-111111111111"]
}
```

The seeded Flyway data in [`V018__seed_test_data.sql`](../../../pug-service/src/main/resources/db/migration/V018__seed_test_data.sql) includes concrete partner examples such as `Inova Tech Labs Ltda`, `Instituto Crescer Social`, and staff emails under `@partner.pug.br`.

## Important classes and files

- Domain:
  - [`Entity`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/domain/Entity.java)
  - [`Staff`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/domain/Staff.java)
  - [`Cnpj`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/domain/vos/Cnpj.java)
  - [`PartnerErrorCodes`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/domain/enums/PartnerErrorCodes.java)
- Write services:
  - [`EntitiesServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/service/impl/EntitiesServiceImpl.java)
  - [`StaffServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/service/impl/StaffServiceImpl.java)
- Read services and queries:
  - [`EntitiesReadServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/service/impl/EntitiesReadServiceImpl.java)
  - [`StaffReadServiceImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/service/impl/StaffReadServiceImpl.java)
  - [`EntitiesQueriesImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/infra/read/impl/EntitiesQueriesImpl.java)
  - [`StaffQueriesImpl`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/infra/read/impl/StaffQueriesImpl.java)
- Presentation:
  - [`EntitiesResource`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/presenter/EntitiesResource.java)
  - [`StaffResource`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/presenter/StaffResource.java)
  - [`EntityPresenter`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/presenter/mappers/EntityPresenter.java)
  - [`StaffPresenter`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/presenter/mappers/StaffPresenter.java)
- Persistence and schema:
  - [`EntityEntity`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/infra/persistence/EntityEntity.java)
  - [`StaffEntity`](../../../pug-service/src/main/java/br/org/catolicasc/pug/partner/infra/persistence/StaffEntity.java)
  - [`V005__create_entities_table.sql`](../../../pug-service/src/main/resources/db/migration/V005__create_entities_table.sql)
  - [`V006__create_staff_table.sql`](../../../pug-service/src/main/resources/db/migration/V006__create_staff_table.sql)

## Dependencies on other modules

- [`shared`](../../../pug-service/src/main/java/br/org/catolicasc/pug/shared): response envelopes, pagination, locale resolution, UUIDv7 validation, search helpers, and audit publishing.
- [`geo`](../../../pug-service/src/main/java/br/org/catolicasc/pug/geo): `EntitiesServiceImpl` validates `cityId` through `CitiesReadService`, and the read side joins `CityEntity` for complex search responses.
- [`identity`](../../../pug-service/src/main/java/br/org/catolicasc/pug/identity): `StaffServiceImpl` provisions, updates, deactivates, and deletes accounts through `AccountsService`; `StaffResource` uses `AuthService` for `/me`.
- [`project`](../../../pug-service/src/main/java/br/org/catolicasc/pug/project): delete guards depend on `ProjectService` and `AttendancesService`; the `project` module also imports partner DTOs and `EntitiesService`.

## How to test the module

- Tests live under [`src/test/java/br/org/catolicasc/pug/partner`](../../../pug-service/src/test/java/br/org/catolicasc/pug/partner).
- The module has three useful test layers:
  - domain and mapper tests such as [`CnpjTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/partner/domain/vos/CnpjTest.java), [`EntityMapperTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/partner/infra/EntityMapperTest.java), and [`StaffMapperTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/partner/infra/StaffMapperTest.java)
  - query/read tests such as [`EntitiesQueriesImplTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/partner/infra/read/impl/EntitiesQueriesImplTest.java) and [`StaffQueriesImplTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/partner/infra/read/impl/StaffQueriesImplTest.java)
  - resource/service integration tests such as [`EntitiesResourceTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/partner/presenter/EntitiesResourceTest.java), [`StaffResourceTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/partner/presenter/StaffResourceTest.java), [`EntitiesServiceImplTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/partner/service/impl/EntitiesServiceImplTest.java), and [`StaffServiceImplTest`](../../../pug-service/src/test/java/br/org/catolicasc/pug/partner/service/impl/StaffServiceImplTest.java)
- Commands:
  - full suite: `./mvnw test`
  - module-focused examples: `./mvnw -Dtest=EntityTest,StaffTest,CnpjTest,EntitiesQueriesImplTest,StaffQueriesImplTest,EntitiesResourceTest,StaffResourceTest,EntitiesServiceImplTest,StaffServiceImplTest test`

## Links

- [Partner architecture](./ARCHITECTURE.md)
- [Back to pug-service docs](../README.md)
