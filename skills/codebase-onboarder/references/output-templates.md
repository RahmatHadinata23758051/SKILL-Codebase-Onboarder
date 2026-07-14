# Output Templates Reference

Use these Markdown templates to structure the analysis reports in `<project-root>/.agents/skills/codebase-onboarder/`. All files should be written in English.

---

## 1. project-overview.md

```markdown
# Project Overview: [Project Name]

*   **Analysis Date:** [YYYY-MM-DD HH:MM]
*   **Primary Language:** [e.g. TypeScript, Python]
*   **Architecture Pattern:** [Layered MVC / Hexagonal / Event-Driven / Monorepo]
*   **Framework/Stack:** [e.g. Next.js, Django]
*   **Entry Points:**
    *   `[path/to/entry/file]`: [brief role]
*   **Database:** [e.g. PostgreSQL (via Prisma)]

## Core Folder Structure

```text
[Insert clean tree diagram showing major folders only]
```

## Architectural Summary

[Provide a 1-2 paragraph description of the general architecture of the project. Mention whether it is a standard clean architecture, a MVC pattern, a monolith, or a microservices-based setup.]

## Main Dependency Summary

[Highlight the main external libraries and packages the project relies on, gathered from package manifest analysis.]
```

---

## 2. Layered / MVC Templates

If the project is a standard layered/MVC project, generate these three files:

### presentation-layer.md
```markdown
# Presentation Layer Analysis

This report lists the routing definitions, pages, views, and controllers that handle user interaction and external API exposure.

## Routing Map

| Route / Path | Handling Component/File | Method | Description |
|---|---|---|---|
| `[e.g. /api/users]` | `[e.g. src/controllers/user.controller.ts]` | `[GET/POST]` | [Brief description of route functionality] |

## Pages / Screens / UI Components

[Delineate the structure of the UI components, folders containing views, and template configurations.]

## State Management

[Describe how the user interface keeps state — e.g. React hooks, Redux, local storage, context APIs, BLoC pattern.]

## Request Validation

[Detail how input from requests is validated before being passed down to the business logic layer.]
```

### logic-layer.md
```markdown
# Logic Layer (Business Rules)

This report details the core business logic, services, event handlers, and rules driving the application.

## Core Services & Use Cases

*   **[Service Name] (`[path/to/service/file]`)**
    *   **Method Summary:**
        *   `[methodName(params)]`: [What it does]
    *   **Logic summary:** [Summary of business rules implemented inside this class/module]

## Middleware & Interceptors

[Detail authentication, logging, validation, or correlation middleware chained in the business logic flow.]

## External API Integrations

[Summarize external integrations like payment processors, mailing services, or third-party OAuth providers.]
```

### data-layer.md
```markdown
# Data Layer Analysis

This report details the database schemas, ORM models, data access repositories, and external data sources.

## Database & ORM Config

*   **Database Engine:** [e.g. PostgreSQL]
*   **ORM / Data Mapper:** [e.g. SQLAlchemy, TypeORM, Prisma]
*   **Connection Config file:** `[path/to/config/file]`

## Database Models & Entities

*   **[Model Name]**
    *   **Source File:** `[path/to/model/file]`
    *   **Key Fields:** `[e.g. id (Int, PK), name (String)]`
    *   **Relationships:** `[e.g. belongsTo User, hasMany Invoices]`

## Database Migrations

*   **Tool:** [e.g. Alembic, Prisma Migrate]
*   **Latest Migration:** `[Migration Name]`
*   **Summary:** [Brief note on migration status, tables created]
```

---

## 3. Clean / Hexagonal Architecture Templates

If the project uses Domain-Driven Design or Hexagonal architecture, generate these two files instead of layered ones:

### core-domain.md
```markdown
# Core Domain Analysis

This report documents the central business models, aggregates, and domain services which have zero framework dependencies.

## Domain Entities & Aggregates

*   **[Entity Name] (`[path/to/entity/file]`)**
    *   **Attributes:** `[e.g. id, status, balance]`
    *   **Validation Rules:** [Business rules enforced inside the entity (e.g. status can't transition from Pending to Active without payment)]

## Domain Services

*   **[Domain Service Name] (`[path/to/service/file]`)**
    *   **Purpose**: [Brief explanation of domain service responsibility]
    *   **Methods**:
        *   `[methodName(params)]`: [Brief description]
```

### ports-and-adapters.md
```markdown
# Ports and Adapters Analysis

This report lists the interfaces (Ports) and their implementations (Adapters) connecting the Core Domain to external services.

## Incoming Ports (Driving Adapters)
*How triggers/requests enter the core (e.g., Controllers, CLI).*

*   **Port Name:** `[Interface/UseCase Name]`
    *   **Driving Adapter:** `[e.g. UserController.ts]`
    *   **Entry Type:** [HTTP Endpoint / CLI Command]

## Outgoing Ports (Driven Adapters)
*How the core calls databases or external services.*

*   **Port Name:** `[e.g. UserRepository interface]`
    *   **Driven Adapter Implementation:** `[e.g. PostgresUserRepository.ts]`
    *   **Technology**: [e.g. TypeORM / raw SQL]
```

---

## 4. Event-Driven Architecture Templates

If the project is primarily structured around message processing, generate these three files:

### event-publishers.md
```markdown
# Event Publishers

This report maps the events published by the application and the triggers that cause them.

*   **[Event Name / Topic]**
    *   **Published From:** `[path/to/publisher/file]`
    *   **Payload Schema:** `[e.g. { userId: String, timestamp: Epoch }]`
    *   **Trigger Condition:** [What actions publish this event]
```

### event-channels.md
```markdown
# Event Channels (Queue Configurations)

This report details queue configurations, brokers, and routing topics.

*   **Broker Technology:** [e.g. RabbitMQ / Kafka / Redis]
*   **Config File:** `[path/to/queue/config/file]`
*   **Daftar Queue / Topics:**
    *   `[Queue/Topic Name]`: [Routing key / exchanges / settings]
```

### event-handlers.md
```markdown
# Event Handlers & Consumers

This report maps queue event consumers and the logic triggered by them.

*   **[Consumer Class / Handler Name]**
    *   **Listening to Topic/Queue:** `[Queue/Topic Name]`
    *   **Handler File:** `[path/to/handler/file]`
    *   **Actions Performed:** [What this consumer does when it receives the event]
```

---

## 5. layer-integration.md

```markdown
# Component & Layer Integration Map

This document describes how the presentation, business logic, data, and broker layers connect and communicate.

## Interaction Flow

```mermaid
[Insert Mermaid sequence or flow diagram representing a standard path]
```

## Architecture Diagram

```mermaid
[Insert Mermaid class/graph diagram showing the layers/packages and their dependencies]
```

## Communication Patterns

[Describe communication styles - e.g. direct imports, DI container, event-driven pub-sub, REST/gRPC API calls.]

## Architectural Violations

[Highlight code patterns that bypass designated layers/boundaries (e.g. presentation direct DB query).]
```

---

## 6. issues-and-bugs.md

*(Create this file only if issues are found)*

```markdown
# Project Issues & Code Smell Log

The following issues were identified during code-flow and progressive scanning analysis of the codebase.

## 1. High Priority (Bugs & Security Risks)

*   **[Issue Title]**
    *   **File:** [Filename](file:///absolute/path/to/file#Lline-number)
    *   **Category:** [Security / Logic Bug]
    *   **Description:** [Detailed description of the issue]
    *   **Recommendation:** [Suggested fix]

## 2. Medium Priority (Architectural Violations & Performance)

*   **[Issue Title]**
    *   **File:** [Filename](file:///absolute/path/to/file#Lline-number)
    *   **Category:** [Architectural Violation / Performance]
    *   **Description:** [Description]
    *   **Recommendation:** [Recommendation]

## 3. Low Priority (Code Smells & Formatting)

*   **[Issue Title]**
    *   **File:** [Filename](file:///absolute/path/to/file#Lline-number)
    *   **Category:** [Code Smell]
    *   **Description:** [Description]
    *   **Recommendation:** [Recommendation]
```
