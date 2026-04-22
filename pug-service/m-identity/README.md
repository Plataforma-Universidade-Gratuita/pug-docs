# 🔐 Identity Module

## Overview

The **Identity** module is the core authentication and authorization bounded context. It manages **Users** (personal identity — CPF and name), **Accounts** (authentication credentials — email, password, account type), and **Admins** (elevated administrative privileges tied to a campus). It provides JWT-based authentication and role-based access control (RBAC) for the entire platform.

## Domain Model

```mermaid
classDiagram
    class User {
        +UUID id
        +Cpf cpf
        +String name
        +AuditInfo auditInfo
    }

    class Account {
        +UUID id
        +UUID userId
        +Email email
        +AccountType accountType
        +String passwordHash
        +boolean active
        +AuditInfo auditInfo
    }

    class Admin {
        +UUID accountId
        +Campi campus
        +Instant grantedAt
    }

    class Cpf {
        +String value
        «11-digit with modulo-11 checksum»
    }

    class Email {
        +String value
        «normalized, validated»
    }

    class AccountType {
        <<enumeration>>
        ADMIN
        PARTNER
        STUDENT
    }

    class Campi {
        <<enumeration>>
        JARAGUA_DO_SUL
        JOINVILLE
    }

    User "1" --> "*" Account : has
    Account "1" --> "0..1" Admin : may be
    User --> Cpf
    Account --> Email
    Account --> AccountType
    Admin --> Campi
```

## Architecture

```
presenter/                     ← REST controllers
  AuthResource                 ← POST /auth/login (public)
  AdminResource                ← CRUD for admins (ADMIN role)
  AccountReadOnlyResource      ← Read-only account queries (ADMIN role)
  UserReadOnlyResource         ← Read-only user queries (ADMIN role)
  dtos/                        ← Request/Response DTOs
  mappers/                     ← Presenter layer transformers
domain/                        ← Pure domain model
  User, Account, Admin         ← Aggregate roots
  vos/Cpf, Email               ← Value Objects
  *Repository                  ← Repository interfaces
service/                       ← Application services (CQRS)
  AuthService                  ← Login + JWT generation
  PasswordService              ← Bcrypt hashing with pepper
  AccountService               ← Account CRUD commands
  AdminService                 ← Admin CRUD commands
  *ReadService                 ← Query-side services
infra/                         ← Infrastructure layer
  persistence/                 ← JPA entities (UserEntity, AccountEntity, AdminEntity)
  read/                        ← CQRS query implementations
  *Mapper                      ← Domain ↔ JPA anti-corruption layers
```

## Endpoints

### Authentication — `/auth`

```mermaid
graph LR
    subgraph Auth["🔑 /auth"]
        direction TB
        POST_LOGIN["POST /login — Authenticate and receive JWT<br/>🔓 Public"]
    end
```

### Users — `/identity/users`

```mermaid
graph LR
    subgraph Users["👤 /identity/users"]
        direction TB
        GET_LIST["GET / — List all or search ?q=<br/>🔒 ADMIN"]
        GET_ID["GET /{id} — Get by UUID<br/>🔒 ADMIN"]
        GET_CPF["GET /by-cpf/{cpf} — Get by CPF<br/>🔒 ADMIN"]
        GET_ME["GET /me — Current user profile<br/>🔒 Authenticated"]
    end
```

### Accounts — `/identity/accounts`

```mermaid
graph LR
    subgraph Accounts["📧 /identity/accounts"]
        direction TB
        GET_LIST["GET / — List all or search ?q=<br/>🔒 ADMIN"]
        GET_ID["GET /{id} — Get by UUID<br/>🔒 ADMIN"]
        GET_EMAIL["GET /by-email/{email} — Get by email<br/>🔒 ADMIN"]
        GET_CPF["GET /by-cpf/{cpf} — List by CPF<br/>🔒 ADMIN"]
        GET_ME["GET /me — Current account<br/>🔒 Authenticated"]
    end
```

### Admins — `/identity/admins`

```mermaid
graph LR
    subgraph Admins["🛡️ /identity/admins"]
        direction TB
        GET_LIST["GET / — List all or search ?q=<br/>🔒 ADMIN"]
        GET_ID["GET /{id} — Get by account ID<br/>🔒 ADMIN"]
        GET_EMAIL["GET /by-email/{email}<br/>🔒 ADMIN"]
        GET_CPF["GET /by-cpf/{cpf}<br/>🔒 ADMIN"]
        GET_ME["GET /me — Current admin profile<br/>🔒 ADMIN"]
        POST["POST / — Create admin<br/>🔒 ADMIN"]
        PUT["PUT /{id} — Update admin<br/>🔒 ADMIN"]
        PATCH["PATCH /{id}/deactivate — Soft-deactivate<br/>🔒 ADMIN"]
        DELETE["DELETE /{id} — Revoke admin<br/>🔒 ADMIN"]
    end
```

## Use Case Diagram

```mermaid
graph TB
    subgraph Actors
        PUBLIC["🌐 Public User"]
        AUTH["👤 Authenticated User"]
        ADMIN["🛡️ Admin"]
    end

    subgraph IdentityModule["🔐 Identity Module"]
        UC1["Login — email + password → JWT"]
        UC2["View own User profile /me"]
        UC3["View own Account details /me"]
        UC4["CRUD Admins"]
        UC5["Search / List Users"]
        UC6["Search / List Accounts"]
        UC7["Deactivate Accounts"]
    end

    PUBLIC --> UC1
    AUTH --> UC2
    AUTH --> UC3
    ADMIN --> UC4
    ADMIN --> UC5
    ADMIN --> UC6
    ADMIN --> UC7
```

## ERM (Entity-Relationship Model)

```mermaid
erDiagram
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

    users ||--o{ accounts : "has"
    accounts ||--o| admins : "may be"
```

## Security Model

- **Password hashing**: Bcrypt with application-level pepper.
- **JWT tokens**: SmallRye JWT with HS256 signing. Claims include `accountId`, `userId`, `accountType`, and role groups.
- **Role-based access**: Enforced via `@RolesAllowed("ADMIN")`, `@Authenticated`, and `@PermitAll`.
