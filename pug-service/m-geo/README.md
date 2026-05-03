# 🌍 Geo Module

## Overview

The **Geo** module is a lightweight, read-only bounded context responsible for managing geographic reference data — specifically Brazilian cities and their IBGE codes. It serves as a shared dictionary consumed by other modules (Partner, Academic) to resolve location references.

## Domain Model

```mermaid
classDiagram
    class City {
        +UUID id
        +String name
        +IbgeCode ibgeCode
    }

    class IbgeCode {
        +String value
        «exactly 7 numeric digits»
    }

    City --> IbgeCode
```

## Architecture

```
presenter/              ← REST controllers (read-only)
  CityReadOnlyResource  ← GET endpoints for cities
  mappers/              ← Presenter → Response mapping
    CityPresenter
domain/                 ← Pure domain model
  City                  ← Aggregate root
  CityRepository        ← Read-only repository interface
  vos/IbgeCode          ← Value Object
service/                ← Application services (CQRS Query side)
  CityReadService       ← Read service
infra/                  ← Infrastructure layer
  CityMapper            ← Domain ↔ JPA mapping
  persistence/          ← JPA entities
    CityEntity          ← Hibernate Search indexed
  read/                 ← CQRS Query implementations
    CityQueries         ← Query interface
    impl/               ← JPQL + Elasticsearch implementations
```

## Endpoints

```mermaid
graph LR
    subgraph Cities["🌍 /v1/geo/cities"]
        direction TB
        GET_LIST["GET / — List all or search by name ?q=<br/>🔒 Authenticated"]
        GET_ID["GET /{id} — Get city by UUID<br/>🔒 Authenticated"]
        GET_IBGE["GET /by-ibge/{ibgeCode} — Get by IBGE code<br/>🔒 Authenticated"]
    end
```

## Use Case Diagram

```mermaid
graph TB
    subgraph Actors
        AUTH["👤 Authenticated User"]
    end

    subgraph GeoModule["🌍 Geo Module"]
        UC1["List / Search Cities"]
        UC2["Get City by ID"]
        UC3["Get City by IBGE Code"]
    end

    AUTH --> UC1
    AUTH --> UC2
    AUTH --> UC3
```

## ERM (Entity-Relationship Model)

```mermaid
erDiagram
    cities {
        UUID id PK
        VARCHAR name
        CHAR_7 ibge_code UK
    }

    entities {
        UUID city_id FK
    }

    cities ||--o{ entities : "referenced by"
```

## Key Design Decisions

- **Read-only**: City data is seeded via Flyway migrations, not through REST endpoints.
- **Full-text search**: City names are indexed in Elasticsearch via Hibernate Search with accent-insensitive, fuzzy, and autocomplete analyzers.
- **Cross-module dependency**: Partner entities reference `city_id` as a foreign key.

