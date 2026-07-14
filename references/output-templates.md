# Output Templates Reference

Use these Markdown templates to structure the analysis reports in `<project-root>/.agents/skills/codebase-onboarder/`. All files should be written in English.

---

## 1. project-overview.md

```markdown
# Project Overview: [Project Name]

*   **Analysis Date:** [YYYY-MM-DD HH:MM]
*   **Primary Language:** [e.g. TypeScript, Python]
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

## 2. presentation-layer.md

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

---

## 3. logic-layer.md

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

---

## 4. data-layer.md

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

## 5. layer-integration.md

```markdown
# Layer Integration Map

This document describes how the presentation, business logic, and data layers connect and communicate within the project.

## Request Lifecycle Flow

```mermaid
[Insert Mermaid sequence diagram representing a standard request path, e.g. UI -> Controller -> Service -> Repo -> DB]
```

## Layer Architecture Diagram

```mermaid
[Insert Mermaid class/graph diagram showing the layers and their dependencies]
```

## Communication Patterns

[Describe how layers communicate (direct imports, DI/IoC container, event-driven/pubsub, REST/gRPC API calls).]

## Architectural Violations

[Highlight any observed code patterns that bypass layers, such as a controller file directly executing raw SQL or importing database models and modifying records directly without going through a service class.]
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
