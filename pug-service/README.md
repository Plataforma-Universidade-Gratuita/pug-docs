# 🐾 PUG Service

> **P**lataforma **U**niversitária de **G**estão — A comprehensive backend service for managing university community service (counterpart hours) programs.

## 📖 Project Overview

**PUG Service** is a modular, monolithic REST API built with **Quarkus** and **Java 21** that supports the full lifecycle of university community service programs in Brazil. It enables administrators to manage partner organizations, academic structures, student enrollments, community service projects, and QR-based attendance tracking — all through a unified, secure API.

### Key Features

- 🔐 **JWT Authentication** with role-based access control (ADMIN, PARTNER, STUDENT)
- 🏢 **Partner Management** — Organizations identified by CNPJ with staff assignments
- 🎓 **Academic Structure** — Schools → Courses → Students with counterpart hour tracking
- 📋 **Project Lifecycle** — Full state machine (Planned → In Progress → Completed/Canceled)
- 📝 **Enrollment Management** — Student-to-project enrollment with lifecycle states
- 📱 **QR Attendance** — QR-code-based attendance registration and staff validation
- 🔍 **Full-text Search** — Elasticsearch-powered fuzzy, autocomplete, accent-insensitive search
- 🌍 **Internationalization** — Full i18n support (pt_BR, en_US)
- 📊 **CQRS Architecture** — Separated read (Query) and write (Command) paths

## 🛠️ Tech Stack

```mermaid
mindmap
  root((PUG Service))
    Runtime
      Java 21
    Framework
      Quarkus 3.14.4
    REST
      RESTEasy Reactive
      Jackson
    ORM
      Hibernate ORM
      Panache
    Database
      PostgreSQL 16
    Search Engine
      Elasticsearch 9.x
      Hibernate Search
    Migrations
      Flyway
    Security
      SmallRye JWT HS256
      Bcrypt with pepper
    Validation
      Jakarta Bean Validation
      Hibernate Validator
    Build
      Maven
    Code Quality
      SpotBugs
      Checkstyle
      Spotless
    UUID Strategy
      UUIDv7 time-ordered
```

## 🏗️ Architecture

The project follows a **modular monolith** pattern using **Domain-Driven Design (DDD)** with clearly separated bounded contexts. Each module follows a clean layered architecture with strict anti-corruption boundaries.

### Architecture Diagram

```mermaid
graph TB
    subgraph PUG["🐾 PUG Service"]
        direction TB
        subgraph Modules["Bounded Contexts"]
            direction LR
            GEO["🌍 Geo<br/><i>Cities</i>"]
            IDENTITY["🔐 Identity<br/><i>Auth / JWT / Users<br/>Accounts / Admins</i>"]
            ACADEMIC["🎓 Academic<br/><i>Schools / Courses<br/>Students</i>"]
            PARTNER["🏢 Partner<br/><i>Entities / Staff</i>"]
            PROJECT["📋 Project<br/><i>Projects / Enrollments<br/>Attendances</i>"]
        end

        SHARED["🧩 Shared Module<br/><i>DomainError · Exceptions · i18n<br/>API Envelope · Search · Utils</i>"]

        GEO --> SHARED
        IDENTITY --> SHARED
        ACADEMIC --> SHARED
        PARTNER --> SHARED
        PROJECT --> SHARED
    end

    subgraph Infra["Infrastructure Layer"]
        direction LR
        PG[("PostgreSQL 16")]
        ES[("Elasticsearch 9")]
        FW["Flyway Migrations"]
    end

    SHARED --> Infra
```

### Module Layer Pattern

Each bounded context follows this consistent internal structure:

```
module/
├── domain/          ← Pure business logic (entities, value objects, repository interfaces)
├── service/         ← Application services (CQRS: commands + queries)
├── infra/           ← Infrastructure (JPA entities, mappers, repository implementations)
│   ├── persistence/ ← Hibernate/JPA entities
│   └── read/        ← CQRS Query implementations (JPQL + Elasticsearch)
└── presenter/       ← REST controllers, DTOs, request/response mapping
    ├── dtos/        ← Request/Response records
    └── mappers/     ← View → Response transformers
```

## 📦 Modules

```mermaid
graph LR
    GEO["🌍 <b>Geo</b><br/>com.pug.geo<br/><i>Geographic reference data<br/>Brazilian cities + IBGE codes</i>"]
    IDENTITY["🔐 <b>Identity</b><br/>com.pug.identity<br/><i>Authentication, users,<br/>accounts, admins, JWT</i>"]
    ACADEMIC["🎓 <b>Academic</b><br/>com.pug.academic<br/><i>Schools, courses, students,<br/>counterpart hours</i>"]
    PARTNER["🏢 <b>Partner</b><br/>com.pug.partner<br/><i>Partner organizations CNPJ,<br/>staff management</i>"]
    PROJECT["📋 <b>Project</b><br/>com.pug.project<br/><i>Community service projects,<br/>enrollments, QR attendance</i>"]
    SHARED["🧩 <b>Shared</b><br/>com.pug.shared<br/><i>Cross-cutting concerns:<br/>exceptions, i18n, search, API envelope</i>"]
```

## 🗃️ Full Entity-Relationship Model (ERM)

```mermaid
erDiagram
    cities {
        UUID id PK
        VARCHAR name
        CHAR_7 ibge_code UK
    }

    users {
        UUID id PK
        CHAR_11 cpf UK
        VARCHAR name
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
    }

    accounts {
        UUID id PK
        UUID user_id FK
        VARCHAR email UK
        VARCHAR password_hash
        VARCHAR account_type
        BOOLEAN active
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
    }

    admins {
        UUID account_id PK
        VARCHAR campus
        TIMESTAMPTZ granted_at
    }

    schools {
        UUID id PK
        VARCHAR name UK
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
    }

    courses {
        UUID id PK
        UUID school_id FK
        VARCHAR name UK
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
    }

    students {
        UUID account_id PK
        VARCHAR academic_registration UK
        VARCHAR campus
        UUID course_id FK
        DECIMAL required_hours
        DECIMAL completed_hours
        BOOLEAN concluded
        DATE start_date
        DATE due_date
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
    }

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

    projects {
        UUID id PK
        VARCHAR name
        UUID entity_id FK
        TEXT description
        UUID created_by FK
        INT max_participants
        DECIMAL offered_hours
        DECIMAL completed_hours
        VARCHAR status
        TIMESTAMPTZ closed_at
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
    }

    projects_by_schools {
        UUID project_id PK
        UUID school_id PK
    }

    enrollments {
        UUID project_id PK
        UUID student_id PK
        VARCHAR status
        TIMESTAMPTZ accepted_at
        TIMESTAMPTZ closing_status_at
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
    }

    attendances {
        UUID id PK
        UUID project_id FK
        UUID student_id FK
        DECIMAL duration
        VARCHAR qr_validation_hash UK
        VARCHAR status
        UUID validated_by FK
        TIMESTAMPTZ validated_at
        TIMESTAMPTZ created_at
        TIMESTAMPTZ updated_at
    }

    users ||--o{ accounts : "has"
    accounts ||--o| admins : "may be"
    accounts ||--o| students : "may be"
    accounts ||--o| staff : "may be"
    cities ||--o{ entities : "located in"
    entities ||--o{ staff : "employs"
    entities ||--o{ projects : "offers"
    schools ||--o{ courses : "contains"
    courses ||--o{ students : "enrolled in"
    staff ||--o{ projects : "created_by"
    projects ||--o{ projects_by_schools : "associated"
    schools ||--o{ projects_by_schools : "associated"
    projects ||--o{ enrollments : "has"
    students ||--o{ enrollments : "participates"
    projects ||--o{ attendances : "has"
    students ||--o{ attendances : "records"
    staff ||--o{ attendances : "validated_by"
```

## 🔗 Module Dependencies

```mermaid
graph BT
    SHARED["🧩 Shared"]

    GEO["🌍 Geo"] --> SHARED
    IDENTITY["🔐 Identity"] --> SHARED
    ACADEMIC["🎓 Academic"] --> SHARED

    PARTNER["🏢 Partner"] --> GEO
    PARTNER --> IDENTITY
    PARTNER --> SHARED

    PROJECT["📋 Project"] --> IDENTITY
    PROJECT --> ACADEMIC
    PROJECT --> PARTNER
    PROJECT --> SHARED

    style SHARED fill:#f9f,stroke:#333,stroke-width:2px
    style PROJECT fill:#bbf,stroke:#333,stroke-width:2px
```

## 🗄️ Database Migrations

Managed by Flyway in `src/main/resources/db/migration/`:

```mermaid
timeline
    title Database Migration History
    V000 : UUID v7 generation function
    V001 : Users table
    V002 : Accounts table
    V003 : Cities table
    V004 : Admins table
    V005 : Entities - Partner table
    V006 : Staff table
    V007 : Schools table
    V008 : Courses table
    V009 : Students table
    V010 : Projects table
    V011 : Projects by Schools M-N table
    V012 : Enrollments table
    V013 : Attendances table
    V014 : System user seed data
```

## 🚀 Getting Started

### Prerequisites

- Java 21+
- Docker (for PostgreSQL and Elasticsearch dev services)
- Maven 3.9+

### Running in Development Mode

```bash
./mvnw quarkus:dev
```

Quarkus Dev Services will automatically provision PostgreSQL and Elasticsearch containers.

> **_NOTE:_** Quarkus ships with a Dev UI, available in dev mode at <http://localhost:8080/q/dev/>.

### Packaging and Running

```bash
./mvnw package
java -jar target/quarkus-app/quarkus-run.jar
```

### Building an Über-jar

```bash
./mvnw package -Dquarkus.package.jar.type=uber-jar
java -jar target/*-runner.jar
```

### Creating a Native Executable

```bash
./mvnw package -Dnative
```

Or using a container build:

```bash
./mvnw package -Dnative -Dquarkus.native.container-build=true
```

## 📄 API Response Format

All API responses follow a standardized envelope:

```json
{
  "success": true,
  "data": { ... },
  "error": null,
  "timestamp": "2026-04-10T12:00:00Z",
  "correlationId": "abc-123"
}
```

Error responses:

```json
{
  "success": false,
  "data": null,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed.",
    "details": [
      {
        "field": "name",
        "errors": [
          { "code": "INVALID_NAME_BLANK", "message": "Name cannot be blank." }
        ]
      }
    ]
  },
  "timestamp": "2026-04-10T12:00:00Z",
  "correlationId": "abc-123"
}
```

## 📜 License

This project is part of an academic portfolio.
