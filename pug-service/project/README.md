# 📋 Project Module

## 📌 Overview

The project module is the operational core of the platform.

Responsibilities:

- manage projects and lifecycle transitions
- associate projects with academic areas of expertise
- manage enrollments
- manage attendances and validation
- propagate status effects between projects, enrollments, and former-student progress

## 🧠 Domain model

```mermaid
classDiagram
    class Project {
      +UUID id
      +String name
      +UUID entityId
      +String description
      +ProjectInfo projectInfo
      +ProjectStatus status
    }

    class ProjectAreaOfExpertise {
      +UUID projectId
      +UUID areaOfExpertiseId
    }

    class Enrollment {
      +EnrollmentIdentifier identifier
      +EnrollmentStatus status
      +EnrollmentInfo enrollmentInfo
    }

    class Attendance {
      +UUID id
      +UUID projectId
      +UUID formerStudentId
      +AttendanceStatus status
      +AttendanceInfo attendanceInfo
      +QrValidationInfo qrValidationInfo
    }

    Project "1" --> "*" ProjectAreaOfExpertise
    Project "1" --> "*" Enrollment
    Project "1" --> "*" Attendance
```

## 🌐 Public endpoints

### Projects

```text
GET    /v1/projects/{id}
GET    /v1/projects?ids=
GET    /v1/projects/entities/{entityId}
GET    /v1/projects/creators/{createdById}
POST   /v1/projects/search
POST   /v1/projects
PUT    /v1/projects/{id}
PATCH  /v1/projects/{id}/status
DELETE /v1/projects/{id}
```

### Project <-> area-of-expertise association

```text
GET    /v1/projects/{projectId}/areas-of-expertise
POST   /v1/projects/{projectId}/areas-of-expertise
DELETE /v1/projects/{projectId}/areas-of-expertise/{areaOfExpertiseId}
DELETE /v1/projects/{projectId}/areas-of-expertise

GET    /v1/academic/areas-of-expertise/{areaOfExpertiseId}/projects
DELETE /v1/academic/areas-of-expertise/{areaOfExpertiseId}/projects
```

### Enrollments

```text
GET    /v1/projects/{projectId}/enrollments/{formerStudentId}
GET    /v1/projects/{projectId}/enrollments/me
GET    /v1/projects/enrollments
GET    /v1/projects/enrollments/me
POST   /v1/projects/{projectId}/enrollments
POST   /v1/projects/enrollments/search
PATCH  /v1/projects/{projectId}/enrollments/{formerStudentId}
PATCH  /v1/projects/{projectId}/enrollments/me
DELETE /v1/projects/{projectId}/enrollments/{formerStudentId}
```

### Attendances

```text
GET    /v1/projects/attendances/{id}
GET    /v1/projects/attendances?ids=
POST   /v1/projects/attendances/search
POST   /v1/projects/attendances
PATCH  /v1/projects/attendances/{id}/validate
DELETE /v1/projects/attendances/{id}
```

## 🔍 Complex-search contracts

```mermaid
graph TB
    P["ProjectComplexSearchRequest<br/>name, entityIds, description,<br/>createdByIds, dateFrom, dateTo,<br/>statuses, minOfferedHours, maxOfferedHours"]
    E["EnrollmentComplexSearchRequest<br/>projectIds, formerStudentIds, statuses,<br/>dateFrom, dateTo, periodFrom, periodTo"]
    A["AttendanceComplexSearchRequest<br/>projectIds, formerStudentIds, statuses,<br/>validatedByIds, durationFrom, durationTo,<br/>dateFrom, dateTo"]
```

## 🔄 Lifecycle rules

### Project status

```mermaid
stateDiagram-v2
    [*] --> PLANNED
    PLANNED --> IN_PROGRESS
    PLANNED --> CANCELED
    IN_PROGRESS --> ON_HOLD
    IN_PROGRESS --> COMPLETED
    IN_PROGRESS --> CANCELED
    ON_HOLD --> PLANNED
    ON_HOLD --> CANCELED
```

### Enrollment status

```mermaid
stateDiagram-v2
    [*] --> PENDING
    PENDING --> APPROVED
    PENDING --> REJECTED
    APPROVED --> ON_HOLD
    APPROVED --> COMPLETED
    APPROVED --> REMOVED
    APPROVED --> EXITED
    ON_HOLD --> APPROVED
    ON_HOLD --> COMPLETED
    ON_HOLD --> REMOVED
```

### Attendance status

```mermaid
stateDiagram-v2
    [*] --> WAITING
    WAITING --> PRESENT
    WAITING --> ABSENT
```

## 🔁 Propagation behavior

The project module contains the most cross-aggregate propagation.

```mermaid
graph TD
    PS["Project status change"]
    EN["Enrollment state propagation"]
    FS["Former-student completed hours"]
    AT["Attendance validation"]

    PS --> EN
    AT --> FS
    FS --> EN
```

Current business effects:

- project `CANCELED` -> project enrollments become `CANCELED`
- project `COMPLETED` -> project enrollments become `COMPLETED`
- project `ON_HOLD` -> approved enrollments become `ON_HOLD`
- project retake to `PLANNED` -> `ON_HOLD` enrollments go back to `APPROVED`
- attendance validation to `PRESENT` adds completed hours
- when a former student concludes required hours, approved enrollments may be completed

## 📦 Response composition

Project-side responses now use grouped DTOs:

- `ProjectResponse`
  - `projectInfo`
  - `status`
- `EnrollmentResponse`
  - `enrollmentInfo`
  - `status`
- `AttendanceResponse`
  - `attendanceInfo`
  - `status`
  - `qrValidationInfo`

## ✅ Notes

- public association terminology is `area-of-expertise`, not `school`
- status transitions should stay in dedicated endpoints and domain/service logic, not drift back into broad update payloads
