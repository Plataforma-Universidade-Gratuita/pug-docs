# 🎓 Academic Module

## 📌 Overview

The academic module owns the university-side catalog and former-student lifecycle.

Main responsibilities:

- manage areas of expertise
- manage courses
- create and update former students
- track counterpart hours and period information
- expose complex-search for frontend filtering

## 🧠 Domain model

```mermaid
classDiagram
    class AreaOfExpertise {
      +UUID id
      +String name
      +AuditInfo auditInfo
    }

    class Course {
      +UUID id
      +String name
      +UUID areaOfExpertiseId
      +AuditInfo auditInfo
    }

    class FormerStudent {
      +UUID accountId
      +AcademicRegistration academicRegistration
      +Campi campus
      +UUID courseId
      +CounterpartHours counterpartHours
      +Period period
      +AuditInfo auditInfo
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

    AreaOfExpertise "1" --> "*" Course
    Course "1" --> "*" FormerStudent
    FormerStudent --> CounterpartHours
    FormerStudent --> Period
```

## 🏗️ Internal structure

```mermaid
graph TB
    PRES["presenter<br/>AreasOfExpertiseResource<br/>CoursesResource<br/>FormerStudentsResource"]
    SERV["service<br/>write services<br/>read services<br/>processors"]
    DOM["domain<br/>AreaOfExpertise<br/>Course<br/>FormerStudent"]
    INF["infra<br/>persistence<br/>read queries<br/>mappers"]

    PRES --> SERV
    SERV --> DOM
    SERV --> INF
    INF --> DOM
```

## 🌐 Public endpoints

### Areas of expertise

```text
GET    /v1/academic/areas-of-expertise/{id}
GET    /v1/academic/areas-of-expertise?ids=
POST   /v1/academic/areas-of-expertise/search
POST   /v1/academic/areas-of-expertise
PUT    /v1/academic/areas-of-expertise/{id}
DELETE /v1/academic/areas-of-expertise/{id}
```

### Courses

```text
GET    /v1/academic/courses/{id}
GET    /v1/academic/courses?ids=
POST   /v1/academic/courses/search
POST   /v1/academic/courses
PUT    /v1/academic/courses/{id}
DELETE /v1/academic/courses/{id}
```

### Former students

```text
GET    /v1/academic/former-students/{id}
GET    /v1/academic/former-students/me
GET    /v1/academic/former-students?ids=
POST   /v1/academic/former-students/search
POST   /v1/academic/former-students
POST   /v1/academic/former-students/bulk
PUT    /v1/academic/former-students/{id}
PATCH  /v1/academic/former-students/{id}/status
DELETE /v1/academic/former-students/{id}
```

## 🔍 Complex-search contracts

```mermaid
graph LR
    AOE["AreaOfExpertiseComplexSearchRequest<br/>name"]
    C["CourseComplexSearchRequest<br/>name, areaOfExpertiseIds"]
    FS["FormerStudentComplexSearchRequest<br/>name, cpf, email, academicRegistration,<br/>campi, periodFrom, periodTo,<br/>includeConcluded, dateFrom, dateTo,<br/>activeOnly, courseIds, areaOfExpertiseIds"]
```

Important search behavior:

- all provided filters combine with `AND`
- `dateFrom` / `dateTo` apply to relevant timestamp fields
- `periodFrom` / `periodTo` apply to `startDate` and `dueDate`
- `includeConcluded=false` keeps concluded former students out unless explicitly requested

## 🔗 Identity coupling

Former-student creation is an aggregate workflow across modules:

```mermaid
sequenceDiagram
    participant API as FormerStudentsResource
    participant SVC as FormerStudentsService
    participant ID as Identity
    participant AC as Academic

    API->>SVC: create/update request
    SVC->>ID: provision user + account
    SVC->>AC: persist former-student aggregate
    AC-->>API: read model projection
```

Current account type used for former students:

- `FORMER_STUDENT`

## 📦 Response composition

Former-student read responses now expose grouped structures instead of a flatter model:

- account information
- campus information
- counterpart hours information
- period information
- course information
- area-of-expertise information through course nesting

## ✅ Notes

- The public API is fully renamed to `former-students` and `areas-of-expertise`.
- Some deep persistence/internal compatibility details may still reference earlier schema evolution, but new documentation and public contracts should not.
