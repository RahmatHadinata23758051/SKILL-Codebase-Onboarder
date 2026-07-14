# Component & Layer Integration Map: BookStore API

This document describes how the presentation, business logic, and data layers connect and communicate within the BookStore API project.

## Interaction Flow

```mermaid
sequenceDiagram
    autonumber
    actor User as Client App
    participant C as UserController (Presentation)
    participant S as UserService (Logic)
    participant R as PrismaDB (Data)

    User->>C: POST /api/users/register (payload)
    Note over C: Validates schema via Zod
    C->>S: registerUser(userData)
    Note over S: Enforces business logic (e.g. email uniqueness)
    S->>R: prisma.user.create(data)
    R-->>S: userRecord
    S-->>C: registrationResult
    C-->>User: HTTP 201 Created (JSON Response)
```

## Architecture Diagram

```mermaid
graph TD
    subgraph Presentation
        app[src/app/api] --> ctrl[src/controllers]
    end
    subgraph Logic
        ctrl --> svc[src/services]
    end
    subgraph Data
        svc --> model[src/models]
        model --> db[(PostgreSQL)]
    end
```

## Communication Patterns

All interaction is synchronous. The HTTP layer (routing) communicates exclusively with controllers. Controllers invoke domain services within the logic layer. Services write to and read from the database using the Prisma ORM client instance. No raw SQL queries were detected in the service or controller layers.
