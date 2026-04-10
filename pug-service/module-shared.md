# 🧩 Shared Module

## Overview

The **Shared** module provides cross-cutting infrastructure, utilities, and base domain abstractions used by all other bounded contexts. It is **not** a standalone business module — instead, it defines the foundational building blocks of the platform's architecture.

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

    class SharedErrorCodes {
        <<interface>>
        INTERNAL_ERROR
        VALIDATION_ERROR
        UNAUTHORIZED_ERROR
    }

    class SharedFieldErrorCodes {
        <<interface>>
        INVALID_ID_BLANK
        INVALID_NAME_BLANK
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

### REST API Infrastructure

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
        +List~Details~ details
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
    ApiError --> FieldErrorsResponse
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
```

### Validation

```mermaid
classDiagram
    class UuidV7 {
        <<Jakarta Bean Validation>>
        +Custom annotation
        +Validates UUIDv7 path parameters
    }
```

## Architecture Diagram

```
shared/
├── domain/
│   ├── DomainError              ← Self-validating base
│   ├── enums/                   ← AccountType, Campi, error codes
│   └── vos/AuditInfo            ← Shared value object
├── exceptions/                  ← Custom exception hierarchy
├── http/CorrelationFilter       ← Request tracing
├── i18n/I18n                    ← Internationalization
├── infra/
│   ├── persistence/             ← Base JPA entities
│   └── search/                  ← Elasticsearch config + utilities
├── presenter/
│   ├── dtos/                    ← Shared response DTOs
│   ├── mappers/                 ← SharedDataPresenter
│   └── rest/                    ← ApiEnvelope, ApiError, exception mappers
├── utils/                       ← String, Collection, Presenter utilities
└── validation/                  ← Custom annotations
```
