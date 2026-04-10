# 📋 Project Module

## Overview

The **Project** module is the operational core of the PUG platform. It manages community service **Projects** offered by partner entities, **Enrollments** of students into those projects, and **Attendance** tracking via QR code validation. Projects follow a rich lifecycle state machine (PLANNED → IN_PROGRESS → COMPLETED/CANCELED) and track counterpart hours.

## Domain Model

```mermaid
classDiagram
    class Project {
        +UUID id
        +String name
        +UUID entityId
        +String description
        +ProjectInfo projectInfo
        +ProjectStatus projectStatus
    }

    class ProjectBySchool {
        +UUID projectId
        +UUID schoolId
    }

    class Enrollment {
        +EnrollmentIdentifier id
        +EnrollmentStatus status
        +EnrollmentInfo enrollmentInfo
    }

    class Attendance {
        +UUID id
        +EnrollmentIdentifier enrollmentIdentifier
        +QrValidationInfo qrValidationInfo
        +AttendanceInfo attendanceInfo
        +AttendanceStatus status
    }

    class ProjectInfo {
        +UUID createdBy
        +int maxParticipants
        +BigDecimal offeredHours
        +BigDecimal completedHours
        +Instant closedAt
        +AuditInfo auditInfo
    }

    class EnrollmentIdentifier {
        +UUID studentId
        +UUID projectId
    }

    class EnrollmentInfo {
        +Instant acceptedAt
        +Instant closingStatusAt
        +AuditInfo auditInfo
    }

    class AttendanceInfo {
        +UUID validatedBy
        +Instant validatedAt
        +AuditInfo auditInfo
    }

    class QrValidationInfo {
        +BigDecimal duration
        +String qrValidationHash
    }

    class ProjectStatus {
        <<enumeration>>
        PLANNED
        IN_PROGRESS
        ON_HOLD
        COMPLETED
        CANCELED
    }

    class EnrollmentStatus {
        <<enumeration>>
        PENDING
        APPROVED
        COMPLETED
        CANCELED
        EXITED
        REJECTED
        REMOVED
    }

    class AttendanceStatus {
        <<enumeration>>
        WAITING
        PRESENT
        ABSENT
    }

    Project --> ProjectInfo
    Project --> ProjectStatus
    Project "1" --> "*" ProjectBySchool
    Project "1" --> "*" Enrollment
    Project "1" --> "*" Attendance
    Enrollment --> EnrollmentIdentifier
    Enrollment --> EnrollmentStatus
    Enrollment --> EnrollmentInfo
    Attendance --> EnrollmentIdentifier
    Attendance --> QrValidationInfo
    Attendance --> AttendanceInfo
    Attendance --> AttendanceStatus
```

## Architecture

```
presenter/                        ← REST controllers
  ProjectResource                 ← CRUD + lifecycle for projects
  ProjectBySchoolResource         ← Project-School associations
  EnrollmentResource              ← CRUD + status transitions for enrollments
  AttendanceResource              ← CRUD + QR validation for attendances
  dtos/                           ← Request/Response DTOs
  mappers/                        ← Presenter layer transformers
domain/                           ← Pure domain model
  Project, Enrollment, Attendance ← Aggregate roots
  ProjectBySchool                 ← Association aggregate
  vos/                            ← Value Objects
  enums/                          ← Status enums
  *Repository                     ← Repository interfaces
service/                          ← Application services (CQRS)
  ProjectService                  ← Project commands + lifecycle
  EnrollmentService               ← Enrollment commands + transitions
  AttendanceService               ← Attendance commands + QR validation
  *ReadService                    ← Query-side services
infra/                            ← Infrastructure layer
  persistence/                    ← JPA entities (Hibernate Search indexed)
  read/                           ← CQRS query implementations
  *Mapper                         ← Domain ↔ JPA anti-corruption layers
```

## Endpoints

### Projects — `/projects/projects`

```mermaid
graph LR
    subgraph Projects["📋 /projects/projects"]
        direction TB
        GET_LIST["GET / — List/search ?q= ?entityId= ?createdBy=<br/>🔒 Authenticated"]
        GET_ID["GET /{id} — Get by UUID<br/>🔒 Authenticated"]
        POST["POST / — Create project<br/>🔒 Authenticated"]
        PUT["PUT /{id} — Update project<br/>🔒 Authenticated"]
        PATCH_START["PATCH /{id}/start — To IN_PROGRESS<br/>🔒 Authenticated"]
        PATCH_COMPLETE["PATCH /{id}/complete — To COMPLETED<br/>🔒 Authenticated"]
        PATCH_CANCEL["PATCH /{id}/cancel — To CANCELED<br/>🔒 Authenticated"]
        PATCH_HOLD["PATCH /{id}/hold — To ON_HOLD<br/>🔒 Authenticated"]
        PATCH_RETAKE["PATCH /{id}/retake — Resume from ON_HOLD<br/>🔒 Authenticated"]
        DELETE["DELETE /{id} — Delete project<br/>🔒 Authenticated"]
    end
```

### Project ↔ School Associations — `/projects/projects-by-schools`

```mermaid
graph LR
    subgraph PBS["🏫 /projects/projects-by-schools"]
        direction TB
        GET_LIST["GET / — List by ?projectId= or ?schoolId=<br/>🔒 Authenticated"]
        POST["POST / — Create association<br/>🔒 Authenticated"]
        DELETE["DELETE / — Remove association<br/>🔒 Authenticated"]
    end
```

### Enrollments — `/projects/enrollments`

```mermaid
graph LR
    subgraph Enrollments["📝 /projects/enrollments"]
        direction TB
        GET_LIST["GET / — List ?projectId= ?studentId=<br/>🔒 Authenticated"]
        GET_ID["GET /{projectId}/{studentId} — Get specific<br/>🔒 Authenticated"]
        POST["POST / — Enroll student<br/>🔒 Authenticated"]
        PATCH["PATCH /{projectId}/{studentId}/status — Change status<br/>🔒 Authenticated"]
        DELETE["DELETE /{projectId}/{studentId} — Delete<br/>🔒 Authenticated"]
    end
```

### Attendances — `/projects/attendances`

```mermaid
graph LR
    subgraph Attendances["📱 /projects/attendances"]
        direction TB
        GET_LIST["GET / — List ?projectId= ?studentId=<br/>🔒 Authenticated"]
        GET_ID["GET /{id} — Get by UUID<br/>🔒 Authenticated"]
        POST["POST / — Record attendance via QR<br/>🔒 Authenticated"]
        PATCH["PATCH /{id}/validate — Staff validates<br/>🔒 Authenticated"]
        DELETE["DELETE /{id} — Delete record<br/>🔒 Authenticated"]
    end
```

## Use Case Diagram

```mermaid
graph TB
    subgraph Actors
        STAFF["👥 Staff / Authenticated User"]
        STUDENT["🎓 Student"]
    end

    subgraph ProjectModule["📋 Project Module"]
        UC1["Create / Update / Delete Project"]
        UC2["Manage Project Lifecycle<br/>start · hold · retake · complete · cancel"]
        UC3["Associate Projects with Schools"]
        UC4["Enroll Students in Projects"]
        UC5["Approve / Reject / Remove Enrollments"]
        UC6["Record Student Attendance via QR"]
        UC7["Validate Attendance — mark PRESENT / ABSENT"]
        UC8["View own Enrollments"]
        UC9["View own Attendances"]
        UC10["Submit Attendance via QR Code"]
    end

    STAFF --> UC1
    STAFF --> UC2
    STAFF --> UC3
    STAFF --> UC4
    STAFF --> UC5
    STAFF --> UC6
    STAFF --> UC7

    STUDENT --> UC8
    STUDENT --> UC9
    STUDENT --> UC10
```

## Project Lifecycle State Machine

```mermaid
stateDiagram-v2
    [*] --> PLANNED
    PLANNED --> IN_PROGRESS : start()
    IN_PROGRESS --> ON_HOLD : putOnHold()
    ON_HOLD --> IN_PROGRESS : retake()
    IN_PROGRESS --> COMPLETED : complete()
    PLANNED --> CANCELED : cancel()
    IN_PROGRESS --> CANCELED : cancel()
    ON_HOLD --> CANCELED : cancel()

    COMPLETED --> [*]
    CANCELED --> [*]

    note right of COMPLETED
        Terminal state —
        cannot be canceled
    end note
```

## Enrollment Lifecycle

```mermaid
stateDiagram-v2
    [*] --> PENDING
    PENDING --> APPROVED : approve()
    PENDING --> REJECTED : reject()
    APPROVED --> COMPLETED : complete()
    APPROVED --> CANCELED : cancel()
    APPROVED --> EXITED : exit()
    APPROVED --> REMOVED : remove()

    COMPLETED --> [*]
    CANCELED --> [*]
    EXITED --> [*]
    REJECTED --> [*]
    REMOVED --> [*]
```

## ERM (Entity-Relationship Model)

```mermaid
erDiagram
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

    entities ||--o{ projects : "offers"
    staff ||--o{ projects : "created_by"
    projects ||--o{ projects_by_schools : "associated"
    schools ||--o{ projects_by_schools : "associated"
    projects ||--o{ enrollments : "has"
    students ||--o{ enrollments : "participates"
    projects ||--o{ attendances : "has"
    students ||--o{ attendances : "records"
    staff ||--o{ attendances : "validated_by"
```

## Business Rules

- Project names must be unique per partner entity.
- A project cannot be deleted if it has enrollments.
- Enrollment status transitions follow a strict state machine (e.g., cannot go from PENDING directly to COMPLETED).
- Attendance QR hashes must be globally unique to prevent duplicate submissions.
- When completed hours reach offered hours, the project auto-completes.
- Staff members validate attendances, creating an audit trail (who validated, when).
