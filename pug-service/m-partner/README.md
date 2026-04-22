# 🏢 Partner Module

## Overview

The **Partner** module manages external partner organizations (entities) and their staff members. A partner entity represents a company or organization identified by a Brazilian CNPJ that offers community service projects. Staff members are accounts with privileges to manage that organization's projects.

## Domain Model

```mermaid
classDiagram
    class Entity {
        +UUID id
        +Cnpj cnpj
        +String name
        +UUID cityId
        +String address
        +AuditInfo auditInfo
    }

    class Staff {
        +UUID accountId
        +UUID entityId
    }

    class Cnpj {
        +String value
        «14-digit with modulo-11 checksum»
    }

    Entity "1" --> "*" Staff : employs
    Entity --> Cnpj
```

## Architecture

```
presenter/                    ← REST controllers
  EntityResource              ← CRUD for partner entities
  StaffResource               ← CRUD for staff members
  dtos/                       ← Request/Response DTOs
  mappers/                    ← Presenter layer transformers
domain/                       ← Pure domain model
  Entity                      ← Partner organization aggregate
  Staff                       ← Staff assignment aggregate
  vos/Cnpj                    ← Value Object
  *Repository                 ← Repository interfaces
service/                      ← Application services (CQRS)
  EntityService               ← Entity CRUD commands
  StaffService                ← Staff assignment commands
  *ReadService                ← Query-side services
infra/                        ← Infrastructure layer
  persistence/                ← JPA entities (Hibernate Search indexed)
  read/                       ← CQRS query implementations
  *Mapper                     ← Domain ↔ JPA anti-corruption layers
```

## Endpoints

### Partner Entities — `/partner/entities`

```mermaid
graph LR
    subgraph Entities["🏢 /partner/entities"]
        direction TB
        GET_LIST["GET / — List/search ?q= ?cityId=<br/>🔒 Authenticated"]
        GET_ID["GET /{id} — Get by UUID<br/>🔒 Authenticated"]
        GET_CNPJ["GET /by-cnpj/{cnpj} — Get by CNPJ<br/>🔒 Authenticated"]
        GET_CITIES["GET /cities — List city IDs used<br/>🔒 Authenticated"]
        POST["POST / — Create entity<br/>🔒 ADMIN"]
        PUT["PUT /{id} — Update entity<br/>🔒 ADMIN"]
        DELETE["DELETE /{id} — Delete entity<br/>🔒 ADMIN"]
    end
```

### Staff Members — `/partners/staff`

```mermaid
graph LR
    subgraph Staff["👥 /partners/staff"]
        direction TB
        GET_LIST["GET / — List/search ?q= ?entityId= ?cpf=<br/>🔒 Authenticated"]
        GET_ID["GET /{id} — Get by account ID<br/>🔒 Authenticated"]
        GET_EMAIL["GET /by-email/{email}<br/>🔒 Authenticated"]
        GET_CPF["GET /by-cpf/{cpf}<br/>🔒 Authenticated"]
        GET_ENTITY["GET /by-entity/{entityId}<br/>🔒 Authenticated"]
        GET_ME["GET /me — Current staff profile<br/>🔒 Authenticated"]
        POST["POST / — Assign staff to entity<br/>🔒 ADMIN"]
        PUT["PUT /{id} — Update staff<br/>🔒 ADMIN"]
        PATCH["PATCH /{id}/deactivate — Soft-deactivate<br/>🔒 ADMIN"]
        DELETE["DELETE /{id} — Revoke staff<br/>🔒 ADMIN"]
    end
```

## Use Case Diagram

```mermaid
graph TB
    subgraph Actors
        AUTH["👤 Authenticated User"]
        ADMIN["🛡️ Admin"]
    end

    subgraph PartnerModule["🏢 Partner Module"]
        UC1["List / Search Partner Entities"]
        UC2["Get Entity by ID or CNPJ"]
        UC3["List / Search Staff Members"]
        UC4["View own Staff profile /me"]
        UC5["Create / Update / Delete Partner Entity"]
        UC6["Assign / Update / Revoke Staff"]
    end

    AUTH --> UC1
    AUTH --> UC2
    AUTH --> UC3
    AUTH --> UC4

    ADMIN --> UC5
    ADMIN --> UC6
```

## ERM (Entity-Relationship Model)

```mermaid
erDiagram
    entities {
        UUID id PK
        CHAR_14 cnpj UK
        VARCHAR name
        UUID city_id FK
        VARCHAR address
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
    }

    staff {
        UUID account_id PK
        UUID entity_id FK
    }

    cities ||--o{ entities : "located in"
    entities ||--o{ staff : "employs"
    staff }o--|| accounts : "PK/FK account_id"
```

## Business Rules

- A CNPJ must be unique across all partner entities.
- An account can only be assigned as staff to **one** partner entity at a time.
- A partner entity cannot be deleted if it has associated projects.
- A staff member cannot be deleted if they have validated attendances or created projects.

