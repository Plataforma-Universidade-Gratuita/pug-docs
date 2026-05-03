# 📋 Project Module

## Overview

The **Project** module is the operational core of the PUG platform. It manages community service **Projects** offered by partner entities, **Project-School associations**, **Enrollments** of students into projects, and **Attendance** tracking via QR code validation. Projects follow a lifecycle state machine and expose the workflow used by staff and students during execution.

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

    class ProjectSchool {
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
    Project "1" --> "*" ProjectSchool
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
  ProjectResource                 ← CRUD + lifecycle updates for projects
  ProjectSchoolResource           ← Project → School association endpoints
  SchoolProjectResource           ← School → Project listing/removal endpoints
  EnrollmentResource              ← Enrollment queries + status transitions
  AttendanceResource              ← CRUD + QR validation for attendances
  dtos/                           ← Request/Response DTOs
  mappers/                        ← Presenter layer transformers
domain/                           ← Pure domain model
  Project, Enrollment, Attendance ← Aggregate roots
  ProjectSchool                   ← Association aggregate
  vos/                            ← Value Objects
  enums/                          ← Status enums
  *Repository                     ← Repository interfaces
service/                          ← Application services (CQRS)
  ProjectService                  ← Project commands + lifecycle
  ProjectSchoolService            ← Project-school association commands
  EnrollmentService               ← Enrollment commands + transitions
  AttendanceService               ← Attendance commands + QR validation
  *ReadService                    ← Query-side services
infra/                            ← Infrastructure layer
  persistence/                    ← JPA entities (Hibernate Search indexed)
  read/                           ← CQRS query implementations
  *Mapper                         ← Domain ↔ JPA anti-corruption layers
```

## Endpoints

### Projects — `/v1/projects`

```mermaid
graph LR
    subgraph Projects["📋 /v1/projects"]
        direction TB
        GET_LIST["GET / — List/search ?q= ?entityId= ?createdBy=<br/>🔒 Authenticated"]
        GET_ID["GET /{id} — Get by UUID<br/>🔒 Authenticated"]
        POST["POST / — Create project<br/>🔒 ADMIN, STAFF"]
        PUT["PUT /{id} — Update project<br/>🔒 ADMIN, STAFF"]
        PATCH["PATCH /{id} — Partial update / lifecycle status change<br/>🔒 ADMIN, STAFF"]
        DELETE["DELETE /{id} — Delete project<br/>🔒 ADMIN, STAFF"]
    end
```

### Project ↔ School Associations

```mermaid
graph LR
    subgraph ProjectSide["🏫 /v1/projects/{projectId}/schools"]
        direction TB
        GET_SCHOOLS["GET / — List schools for project<br/>🔒 Authenticated"]
        POST_SCHOOLS["POST / — Create associations<br/>🔒 ADMIN, STAFF"]
        DELETE_ONE["DELETE /{schoolId} — Remove one association<br/>🔒 ADMIN, STAFF"]
        DELETE_ALL_PROJECT["DELETE / — Remove all by project<br/>🔒 ADMIN, STAFF"]
    end

    subgraph SchoolSide["🎓 /v1/academic/schools/{schoolId}/projects"]
        direction TB
        GET_PROJECTS["GET / — List projects for school<br/>🔒 Authenticated"]
        DELETE_ALL_SCHOOL["DELETE / — Remove all by school<br/>🔒 ADMIN, STAFF"]
    end
```

### Enrollments

```mermaid
graph LR
    subgraph EnrollmentCollection["📝 /v1/projects/enrollments"]
        direction TB
        GET_LIST["GET / — List ?projectId= ?studentId=<br/>🔒 ADMIN, STAFF"]
        GET_ME_LIST["GET /me — My enrollments<br/>🔒 STUDENT"]
    end

    subgraph EnrollmentProject["📝 /v1/projects/{projectId}/enrollments"]
        direction TB
        GET_ONE["GET /{studentId} — Specific enrollment<br/>🔒 ADMIN, STAFF"]
        GET_ME_ONE["GET /me — My enrollment in project<br/>🔒 STUDENT"]
        POST["POST / — Enroll in project (no body)<br/>🔒 STUDENT"]
        PATCH_ONE["PATCH /{studentId} — Update enrollment status<br/>🔒 ADMIN, STAFF"]
        PATCH_ME["PATCH /me — Student exits own enrollment<br/>🔒 STUDENT"]
        DELETE["DELETE /{studentId} — Delete enrollment<br/>🔒 ADMIN"]
    end
```

### Attendances — `/v1/projects/attendances`

```mermaid
graph LR
    subgraph Attendances["📱 /v1/projects/attendances"]
        direction TB
        GET_LIST["GET / — List ?projectId= ?studentId=<br/>🔒 Authenticated"]
        GET_ID["GET /{id} — Get by UUID<br/>🔒 Authenticated"]
        POST["POST / — Record attendance<br/>🔒 Authenticated"]
        PATCH["PATCH /{id}/validate — Staff validates<br/>🔒 Authenticated"]
        DELETE["DELETE /{id} — Delete record<br/>🔒 Authenticated"]
    end
```

## Use Case Diagram

```mermaid
graph TB
    subgraph Actors
        STAFF["👥 Staff / Admin"]
        STUDENT["🎓 Student"]
    end

    subgraph ProjectModule["📋 Project Module"]
        UC1["Create / Update / Delete Projects"]
        UC2["Change Project Status<br/>planned · in progress · on hold · completed · canceled"]
        UC3["Associate Projects with Schools"]
        UC4["List Projects by School"]
        UC5["Enroll in Project"]
        UC6["Approve / Reject / Complete / Cancel / Remove Enrollments"]
        UC7["Exit Own Enrollment"]
        UC8["Record Attendance via QR"]
        UC9["Validate Attendance"]
        UC10["View Own Enrollments"]
    end

    STAFF --> UC1
    STAFF --> UC2
    STAFF --> UC3
    STAFF --> UC4
    STAFF --> UC6
    STAFF --> UC8
    STAFF --> UC9

    STUDENT --> UC5
    STUDENT --> UC7
    STUDENT --> UC8
    STUDENT --> UC10
```

## Project Lifecycle State Machine

```mermaid
stateDiagram-v2
    [*] --> PLANNED
    PLANNED --> IN_PROGRESS : status=IN_PROGRESS
    PLANNED --> CANCELED : status=CANCELED
    IN_PROGRESS --> ON_HOLD : status=ON_HOLD
    IN_PROGRESS --> COMPLETED : status=COMPLETED
    IN_PROGRESS --> CANCELED : status=CANCELED
    ON_HOLD --> IN_PROGRESS : status=IN_PROGRESS
    ON_HOLD --> CANCELED : status=CANCELED

    COMPLETED --> [*]
    CANCELED --> [*]
```

## Enrollment Lifecycle

```mermaid
stateDiagram-v2
    [*] --> PENDING
    PENDING --> APPROVED : status=APPROVED
    PENDING --> REJECTED : status=REJECTED
    APPROVED --> COMPLETED : status=COMPLETED
    APPROVED --> CANCELED : status=CANCELED
    APPROVED --> EXITED : status=EXITED
    APPROVED --> REMOVED : status=REMOVED

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
- A project cannot be deleted while dependent data still blocks the operation.
- Project status transitions follow the lifecycle rules enforced by the domain.
- Project retake is represented by setting status back to `IN_PROGRESS`.
- Enrollment status transitions follow a strict state machine.
- Enrollment creation uses the project identifier from the route, not from a request body.
- Attendance QR hashes must be globally unique to prevent duplicate submissions.
- Attendance validation records who validated and when.


