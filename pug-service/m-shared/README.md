# 🧩 Shared Module

## Overview

The **Shared** module provides cross-cutting infrastructure, utilities, and base domain abstractions used by all other bounded contexts. It is **not** a standalone business module — instead, it defines the foundational building blocks of the platform's architecture, including the domain audit trail persisted to MongoDB.

## Components

### Domain Layer

```mermaid
classDiagram
    class DomainError {
        <<abstract>>
        +List~FieldError~ errors
        +addError(field, code, message)
        +hasErrors() boolean
        +validate()
    }

    class AuditInfo {
        <<value object>>
        +Instant createdAt
        +Instant updatedAt
    }

    class GenericCodes {
        <<interface>>
        +getBundleKey() String
        +getCode() String
    }

    class GenericFieldErrorCodes {
        <<interface>>
        +getFieldName() String
    }

    class SharedErrorCodes {
        <<enumeration>>
        BUSINESS_RULE_ERROR
        DATA_INTEGRITY_ERROR
        DUPLICATED_RESOURCE_ERROR
        INTERNAL_ERROR
        RESOURCE_NOT_FOUND_ERROR
        UNAUTHORIZED_ERROR
        VALIDATION_ERROR
    }

    class SharedFieldErrorCodes {
        <<enumeration>>
        INVALID_AUDIT_INFO_BLANK
        INVALID_CAMPUS_BLANK
        INVALID_CREATED_AT_BLANK
        INVALID_NAME_BLANK
        INVALID_NAME_TOO_LONG
        INVALID_ID_BLANK
        INVALID_UPDATED_AT_BLANK
        INVALID_UPDATED_AT_BEFORE_CREATED_AT
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

    GenericFieldErrorCodes --|> GenericCodes
    SharedErrorCodes ..|> GenericCodes
    SharedFieldErrorCodes ..|> GenericFieldErrorCodes
    DomainError --> AuditInfo
    DomainError ..> SharedFieldErrorCodes : uses
```

### Exception Handling

```mermaid
classDiagram
    class AppValidationException {
        «HTTP 400»
        +List~FieldError~ fieldErrors
    }

    class BusinessRuleException {
        «HTTP 422»
        +String code
        +String message
    }

    class DuplicateResourceException {
        «HTTP 409»
        +String resourceName
    }

    class ResourceNotFoundException {
        «HTTP 404»
        +String resourceName
        +String identifier
    }

    RuntimeException <|-- AppValidationException
    RuntimeException <|-- BusinessRuleException
    RuntimeException <|-- DuplicateResourceException
    RuntimeException <|-- ResourceNotFoundException
```

Exception mappers translate these (and infrastructure-level exceptions) into standardized `ApiEnvelope` error responses:

| Mapper | Handles | HTTP Status |
|---|---|---|
| `AppValidationExceptionMapper` | `AppValidationException` | 400 |
| `ConstraintViolationExceptionMapper` | Jakarta Bean Validation | 400 |
| `NotAuthorizedExceptionMapper` | `NotAuthorizedException` | 401 |
| `ResourceNotFoundExceptionMapper` | `ResourceNotFoundException` | 404 |
| `DuplicateResourceExceptionMapper` | `DuplicateResourceException` | 409 |
| `PersistenceExceptionMapper` | JPA/Hibernate `PersistenceException` | 409 / 500 |
| `BusinessRuleExceptionMapper` | `BusinessRuleException` | 422 |
| `UncaughtExceptionMapper` | Any unhandled `Throwable` | 500 |

### REST API Infrastructure

```mermaid
classDiagram
    class ApiEnvelope~T~ {
        +boolean success
        +T data
        +ApiError error
        +Instant timestamp
        +String correlationId
        +ok(data) ApiEnvelope
        +created(data) ApiEnvelope
        +error(err) ApiEnvelope
    }

    class ApiError {
        +String code
        +String message
        +Details details
    }

    class Details {
        +Object payload
        «@JsonValue — serialized inline»
    }

    class FieldErrorsResponse {
        +String field
        +List~CodeMessage~ errors
    }

    class CorrelationFilter {
        «JAX-RS Filter»
        +X-Correlation-Id management
    }

    ApiEnvelope --> ApiError
    ApiError --> Details
    Details --> FieldErrorsResponse : may contain
```

### Internationalization (i18n)

```mermaid
classDiagram
    class I18n {
        <<ApplicationScoped>>
        +getMessage(code, locale) String
        +getMessage(code, locale, args) String
        -ResourceBundle messages_en_US
        -ResourceBundle messages_pt_BR
    }
```

### Audit Infrastructure (MongoDB)

```mermaid
classDiagram
    class AuditLog {
        «MongoDB Document»
        +String id
        +String entityName
        +UUID entityId
        +String action
        +Map changes
        +UUID performedBy
        +OffsetDateTime timestamp
        +String correlationId
    }

    class AuditPublisher {
        <<ApplicationScoped>>
        +fireCreate(entityName, entityId)
        +fireDelete(entityName, entityId)
        +fireUpdate(entityName, entityId, oldObj, newObj)
    }

    class AuditListener {
        <<ApplicationScoped>>
        «@ObservesAsync»
        +onAuditEvent(DomainAuditEvent)
    }

    class DomainAuditEvent {
        <<record>>
        +String entityName
        +UUID entityId
        +String action
        +Map changes
        +UUID performedBy
        +String correlationId
    }

    class FieldChange {
        <<record>>
        +Object oldValue
        +Object newValue
    }

    AuditPublisher ..> DomainAuditEvent : fires async
    AuditListener ..> DomainAuditEvent : observes
    AuditListener --> AuditLog : persists
    AuditPublisher ..> FieldChange : via DiffUtils
```

### Presenter DTOs

```mermaid
classDiagram
    class AuditInfoResponse {
        <<record>>
        +Instant createdAt
        +Instant updatedAt
    }

    class CampusResponse {
        <<record>>
        +Campi campus
        +String campusFormatted
    }
```

### Infrastructure

```mermaid
classDiagram
    class BaseUuidV7Entity {
        <<abstract MappedSuperclass>>
        +UUID id
    }

    class BaseAuditedEntity {
        <<abstract MappedSuperclass>>
        +Instant createdAt
        +Instant updatedAt
    }

    class EsAnalysis {
        «Elasticsearch Analyzers»
        +pt_folded
        +auto_ngram
        +folding_lowercase
    }

    class HibernateSearchUtils {
        +multiFieldSearch(query, fields, boosts)
    }

    class SearchIndexer {
        «@Observes StartupEvent»
        +rebuildIndexes()
    }

    BaseUuidV7Entity <|-- BaseAuditedEntity
    BaseAuditedEntity ..> EsAnalysis : indexed by
    HibernateSearchUtils ..> EsAnalysis : uses
    SearchIndexer ..> HibernateSearchUtils : triggers
```

### Utilities

```mermaid
classDiagram
    class StringUtils {
        +trim(value) String
        +isEmpty(value) boolean
        +fold(value) String
        +formatDate(date) String
    }

    class CollectionUtils {
        +isNullOrEmpty(collection) boolean
        +toStream(collection) Stream
    }

    class PresenterUtils {
        +resolveLocale(headers) Locale
    }

    class DiffUtils {
        +diff(oldObj, newObj) Map~String, FieldChange~
        -hasChanged(oldVal, newVal) boolean
        «Excludes sensitive fields: passwordHash, email, cpf, cnpj»
    }
```

### Validation

```mermaid
classDiagram
    class UuidV7 {
        <<Jakarta Bean Validation>>
        +Custom annotation
        +Validates UUIDv7 path parameters
    }

    class UuidV7ForString {
        +isValid(String, ctx) boolean
        «Validates string-based UUID v7»
    }

    class UuidV7ForUuid {
        +isValid(UUID, ctx) boolean
        «Validates UUID object v7»
    }

    UuidV7 ..> UuidV7ForString : validated by
    UuidV7 ..> UuidV7ForUuid : validated by
```

## Architecture Diagram

```
shared/
├── domain/
│   ├── DomainError              ← Self-validating base
│   ├── enums/
│   │   ├── AccountType, Campi   ← Shared domain enums
│   │   ├── GenericCodes         ← Base interface for all error/code enums
│   │   ├── GenericFieldErrorCodes ← Base interface for field-specific error codes
│   │   ├── SharedErrorCodes     ← System-wide error codes
│   │   └── SharedFieldErrorCodes ← Shared field validation codes
│   └── vos/AuditInfo            ← Shared value object
├── exceptions/                  ← Custom exception hierarchy
├── http/CorrelationFilter       ← Request tracing
├── i18n/I18n                    ← Internationalization
├── infra/
│   ├── audit/                   ← MongoDB-backed audit trail
│   │   ├── AuditLog             ← MongoDB document entity
│   │   ├── AuditListener        ← Async CDI event consumer
│   │   ├── AuditPublisher       ← Event-firing service
│   │   ├── DomainAuditEvent     ← CDI event payload record
│   │   └── FieldChange          ← Old/new value pair record
│   ├── persistence/             ← Base JPA entities
│   └── search/                  ← Elasticsearch config + utilities
├── presenter/
│   ├── dtos/
│   │   ├── AuditInfoResponse    ← Audit timestamps DTO
│   │   └── CampusResponse       ← Campus enum + localized label DTO
│   ├── mappers/                 ← SharedDataPresenter
│   └── rest/
│       ├── ApiEnvelope          ← Standardized response wrapper
│       ├── ApiError             ← Error payload structure
│       ├── Details              ← Polymorphic error details wrapper
│       ├── FieldErrorsResponse  ← Field-level error details
│       └── mappers/             ← Exception → HTTP response mappers
│           ├── AppValidationExceptionMapper
│           ├── BusinessRuleExceptionMapper
│           ├── ConstraintViolationExceptionMapper
│           ├── DuplicateResourceExceptionMapper
│           ├── NotAuthorizedExceptionMapper
│           ├── PersistenceExceptionMapper
│           ├── ResourceNotFoundExceptionMapper
│           └── UncaughtExceptionMapper
├── utils/
│   ├── CollectionUtils          ← Collection helpers
│   ├── DiffUtils                ← Reflection-based object differ for audit
│   ├── PresenterUtils           ← Locale resolution
│   └── StringUtils              ← String manipulation
└── validation/
    ├── UuidV7                   ← Custom annotation
    ├── UuidV7ForString          ← String-based validator
    └── UuidV7ForUuid            ← UUID-based validator
```
