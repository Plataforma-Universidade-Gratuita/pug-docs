# 🧩 Shared Module

## 📌 Overview

The shared module contains the cross-cutting contracts and infrastructure used by every bounded context.

It is not a business module. It provides the platform rules that keep the other modules consistent.

Main responsibilities:

- standard API envelope and error payloads
- shared exceptions and exception mappers
- i18n
- UUID v7 validation
- pagination support
- JPQL search helpers
- audit event publication and MongoDB audit persistence
- small utility helpers used across modules

## 🧱 Main building blocks

```mermaid
graph TB
    ENV["ApiEnvelope / ApiError"]
    EX["Exceptions / Exception mappers"]
    I18N["I18n bundles"]
    PAG["PageQuery / PageExecution / PageResult / PageResponse"]
    AUD["AuditPublisher / AuditListener / AuditLog"]
    JPA["JpaSearchUtils"]
    UTL["StringUtils / CollectionUtils / PresenterUtils / DiffUtils"]
    VAL["UuidV7 validators"]
```

## 🌐 API response model

```mermaid
classDiagram
    class ApiEnvelope~T~ {
      +boolean success
      +T data
      +ApiError error
      +Instant timestamp
      +String correlationId
    }

    class ApiError {
      +String code
      +String message
      +Object details
    }

    ApiEnvelope --> ApiError
```

Every content-returning endpoint uses `ApiEnvelope`.

Common response patterns:

- `ApiEnvelope.ok(data)`
- `ApiEnvelope.created(data)`
- `204 No Content` for void contracts

## ❗ Exception mapping

```mermaid
graph LR
    EX["Domain / validation / infra exceptions"]
    MAP["REST exception mappers"]
    HTTP["Standard HTTP error envelope"]

    EX --> MAP --> HTTP
```

Important mapped families:

- validation errors
- business rule errors
- unauthorized errors
- duplicate resource errors
- not found errors
- persistence conflicts
- uncaught internal errors

## 🔍 Shared pagination and search

```mermaid
classDiagram
    class PageQuery {
      +int page
      +int size
      +isFetchAll()
    }

    class PageExecution {
      +from(PageQuery, totalElements)
      +apply(query)
    }

    class JpaSearchUtils {
      +folded(field)
      +containsClause(field, parameter)
      +containsPattern(raw)
      +bindContains(query, parameter, raw)
    }
```

Important convention:

- `PageQuery.size == 1` is the shared fetch-all sentinel

## 🧾 Audit architecture

```mermaid
sequenceDiagram
    participant Service as Write Service
    participant Publisher as AuditPublisher
    participant EventBus as CDI async event
    participant Listener as AuditListener
    participant Mongo as MongoDB

    Service->>Publisher: fireCreate / fireUpdate / fireDelete
    Publisher->>EventBus: DomainAuditEvent
    EventBus->>Listener: async delivery
    Listener->>Mongo: persist AuditLog
```

Notes:

- audit publication is asynchronous
- `DiffUtils` is used to compute field changes on updates
- the current account id and correlation id are attached to events

## 🌍 i18n

Shared i18n is the contract behind localized messages and enum formatting.

Current bundle layout includes:

- `messages_en_US.properties`
- `messages_pt_BR.properties`
- `ValidationMessages_en_US.properties`
- `ValidationMessages_pt_BR.properties`

## ✅ Notes

- search is JPQL-based with shared folded matching helpers, not Elasticsearch-based anymore
- this module is where cross-cutting behavior should be added before duplicating small infrastructure patterns across bounded contexts
