# 🎓 Academic Module

## Overview

The **Academic** module manages the university's educational structure: **Schools** (departments), **Courses**, and **Students**. Students are enrolled into courses, tracked by academic registration, campus, and counterpart hour requirements. This module integrates deeply with the Identity module to provision student accounts automatically.

## Domain Model

```mermaid
classDiagram
    class School {
        +UUID id
        +String name
        +AuditInfo auditInfo
    }

    class Course {
        +UUID id
        +String name
        +UUID schoolId
        +AuditInfo auditInfo
    }

    class Student {
        +UUID accountId
        +AcademicRegistration academicRegistration
        +Campi campus
        +UUID courseId
        +CounterpartHours counterpartHours
        +Period period
        +AuditInfo auditInfo
    }

    class AcademicRegistration {
        +String value
        «max 15 chars»
    }

    class CounterpartHours {
        +BigDecimal requiredHours
        +BigDecimal completedHours
        +boolean concluded
    }

    class Period {
        +LocalDate startDate
        +LocalDate dueDate
    }

    class Campi {
        <<enumeration>>
        JARAGUA_DO_SUL
        JOINVILLE
    }

    School "1" --> "*" Course : contains
    Course "1" --> "*" Student : enrolled in
    Student --> AcademicRegistration
    Student --> CounterpartHours
    Student --> Period
    Student --> Campi
```

## Architecture

```
presenter/                      ← REST controllers
  SchoolResource                ← CRUD for schools
  CourseResource                ← CRUD for courses
  StudentResource               ← CRUD for students (incl. bulk create)
  dtos/                         ← Request/Response DTOs
  mappers/                      ← Presenter layer transformers
domain/                         ← Pure domain model
  School, Course, Student       ← Aggregate roots
  vos/                          ← Value Objects
  *Repository                   ← Repository interfaces
service/                        ← Application services (CQRS)
  SchoolService, CourseService  ← Write commands
  StudentService                ← Student enrollment commands
  *ReadService                  ← Query-side services
infra/                          ← Infrastructure layer
  persistence/                  ← JPA entities (Hibernate Search indexed)
  read/                         ← CQRS query implementations
  *Mapper                       ← Domain ↔ JPA anti-corruption layers
```

## Endpoints

### Schools — `/academic/schools`

```mermaid
graph LR
    subgraph Schools["🏫 /academic/schools"]
        direction TB
        GET_LIST["GET / — List all or search ?q=<br/>🔒 Authenticated"]
        GET_ID["GET /{id} — Get by UUID<br/>🔒 Authenticated"]
        POST["POST / — Create school<br/>🔒 ADMIN"]
        PUT["PUT /{id} — Update school<br/>🔒 ADMIN"]
        DELETE["DELETE /{id} — Delete school<br/>🔒 ADMIN"]
    end
```

### Courses — `/academic/courses`

```mermaid
graph LR
    subgraph Courses["📚 /academic/courses"]
        direction TB
        GET_LIST["GET / — List/search ?q= ?schoolId=<br/>🔒 Authenticated"]
        GET_ID["GET /{id} — Get by UUID<br/>🔒 Authenticated"]
        POST["POST / — Create course<br/>🔒 ADMIN"]
        PUT["PUT /{id} — Update course<br/>🔒 ADMIN"]
        DELETE["DELETE /{id} — Delete course<br/>🔒 ADMIN"]
    end
```

### Students — `/academic/students`

```mermaid
graph LR
    subgraph Students["🎓 /academic/students"]
        direction TB
        GET_LIST["GET / — List/search ?q= ?courseId=<br/>🔒 Authenticated"]
        GET_ID["GET /{id} — Get by account ID<br/>🔒 Authenticated"]
        GET_CPF["GET /by-cpf/{cpf}<br/>🔒 ADMIN"]
        GET_EMAIL["GET /by-email/{email}<br/>🔒 ADMIN"]
        GET_REG["GET /by-registration/{reg}<br/>🔒 ADMIN"]
        GET_ME["GET /me — Current student profile<br/>🔒 Authenticated"]
        POST["POST / — Enroll student<br/>🔒 ADMIN"]
        POST_BULK["POST /bulk — Bulk enroll<br/>🔒 ADMIN"]
        PUT["PUT /{id} — Update student<br/>🔒 ADMIN"]
        DELETE["DELETE /{id} — Remove student<br/>🔒 ADMIN"]
    end
```

## Use Case Diagram

```mermaid
graph TB
    subgraph Actors
        AUTH["👤 Authenticated User"]
        ADMIN["🛡️ Admin"]
    end

    subgraph AcademicModule["🎓 Academic Module"]
        UC1["List / Search Schools"]
        UC2["List / Search Courses"]
        UC3["List / Search Students"]
        UC4["View own Student profile /me"]
        UC5["CRUD Schools"]
        UC6["CRUD Courses"]
        UC7["Enroll / Update / Remove Students"]
        UC8["Bulk Enroll Students"]
        UC9["Search by CPF / Email / Registration"]
    end

    AUTH --> UC1
    AUTH --> UC2
    AUTH --> UC3
    AUTH --> UC4

    ADMIN --> UC5
    ADMIN --> UC6
    ADMIN --> UC7
    ADMIN --> UC8
    ADMIN --> UC9
```

## ERM (Entity-Relationship Model)

```mermaid
erDiagram
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

    schools ||--o{ courses : "contains"
    courses ||--o{ students : "enrolled in"
    students }o--|| accounts : "PK/FK account_id"
```

## Business Rules

- School names must be unique.
- Course names must be unique.
- A school cannot be deleted while it has courses.
- A course cannot be deleted while it has students.
- A student cannot be deleted while they have project enrollments.
- Academic registration must be unique across all students.
- Student creation automatically provisions a User + Account (with STUDENT type) in the Identity module.
