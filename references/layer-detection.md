# Framework Layer & Architecture Detection Reference

Use this reference to identify how different frameworks and architectural patterns organize files and folders.

---

## 1. Detection Matrix (Layered / MVC)

| Framework / Tech Stack | Presentation Layer | Logic Layer | Data Layer |
|---|---|---|---|
| **Django (Python)** | `templates/` folders, `views.py`, `forms.py`, `urls.py`, template tags | `services/` folder, custom business utils, `signals.py`, middleware | `models.py`, `managers.py`, database `migrations/` |
| **Laravel (PHP)** | `resources/views/`, `Http/Controllers/`, `Http/Requests/`, `routes/` | `Services/`, `Actions/`, `Events/`, `Listeners/`, `Jobs/` | `Models/`, `Repositories/`, `database/migrations/` |
| **Express/Node (JS/TS)** | `routes/`, `views/`, controllers (if separate), public assets | `services/`, `middleware/`, `validators/`, utility files | `models/`, `schemas/`, `db/migrations/`, API clients |
| **Next.js (TS/React)** | `app/` (pages, layouts), `pages/`, `components/`, UI hooks, styles | Server Actions, API routes, `lib/services/`, server-side logic | Database clients (Prisma, Drizzle, Kysely), model schemas |
| **Spring Boot (Java)** | Controllers (`@RestController`, `@Controller`), DTOs, routing configs | Services (`@Service`), logic components, event listeners | Repositories (`@Repository`), entities (`@Entity`), migrations (`db/migration/`) |
| **Flask (Python)** | Templates, blueprints, views, routes, static resources | Business logic functions, helper modules, custom decorators | Database models (SQLAlchemy, Peewee), database config |
| **FastAPI (Python)** | Routers (`APIRouter`), endpoints, path/query dependencies, Pydantic schemas | Business services, middleware, complex background tasks | SQLAlchemy/SQLModel models, database sessions, migrations |
| **React Native (JS/TS)** | `screens/`, `components/`, custom styling, navigation configs | Hooks, state controllers, event-handling hooks | APIs, local DB (WatermelonDB, SQLite), stores (Redux, Zustand) |
| **Flutter (Dart)** | `widgets/`, `screens/`, `pages/`, presentation logic components | BLoC, Providers, Riverpod controllers, domain classes | DB sources (Hive, Isar, SQFlite), API clients, model serializers |
| **Ruby on Rails** | `app/views/`, `app/controllers/`, `config/routes.rb` | `app/services/`, `app/jobs/`, custom domain classes | `app/models/`, database `db/migrate/` files |
| **ASP.NET Core (C#)** | Controllers, Razor Pages, Views, routing endpoints, DTOs | Services, business handlers, filters/middleware | EF Core DbContext, Entities, DB Migrations, Repository layer |

---

## 2. Dynamic Detection Rules for Unknown Stacks

If the project uses an undocumented custom architecture, apply these semantic detection rules:

### A. Presentation Layer Indicators
*   Any code importing UI-related libraries (e.g. React, Vue, Flutter, HTML-rendering packages).
*   Files handling request payloads, checking routing paths, or preparing HTTP status codes/JSON responses.
*   Template files (`.html`, `.ejs`, `.blade.php`, `.twig`, `.jinja`).
*   Form validation schemas (e.g. Zod, Yup, Pydantic response models, WTForms).

### B. Logic Layer Indicators
*   Files implementing core application rules, calculators, PDF generation, external authentication integrations, or payment gateways.
*   Classes/functions with names ending in: `Service`, `Manager`, `Processor`, `Action`, `Interactor`, `UseCase`, `Handler`, `Helper`.
*   Code handling background queue publishing (e.g. Celery, BullMQ, Sidekiq, RabbitMQ).

### C. Data Layer Indicators
*   Files containing SQL queries, ORM mapping annotations (like `@Entity`, `@Column`, `db.Column`), or MongoDB schemas.
*   Connection setup and configuration files for databases (e.g. `prisma.schema`, `knexfile.js`, `database.config`).
*   Files implementing the repository pattern (names ending in `Repository`, `DAO`, `Schema`).
*   Code responsible for wrapping external third-party API payloads into internal objects (API wrappers/SDKs).

---

## 3. Clean / Hexagonal Architecture Detection

For projects utilizing Domain-Driven Design (DDD) or Ports-and-Adapters architectures:

### A. Core / Domain Layer
*   **What it is**: Pure business rules and entity models. It must have **zero dependencies** on databases, frameworks, or web APIs.
*   **Folder names**: `domain/`, `core/`, `entities/`, `model/domain/`.
*   **Indicators**: Classes containing only logic, getters/setters, and validations. No external HTTP or database framework imports.

### B. Ports Layer (Interfaces)
*   **What it is**: Interfaces defining how the core communicates with the outside world (incoming or outgoing).
*   **Folder names**: `ports/`, `usecases/`, `interfaces/`.
*   **Indicators**: Class or interface definitions containing method signatures but no implementations (e.g. `UserRepository`, `PaymentService` interface).

### C. Adapters Layer (Implementations)
*   **What it is**: Implementations that connect ports to actual databases, web frameworks, and APIs.
*   **Folder names**: `adapters/`, `infrastructure/`, `repositories/`, `controllers/`.
*   **Indicators**: Controllers implementing routing, and repository classes implementing DB queries (e.g. `SqlUserRepository` inheriting from `UserRepository` port).

---

## 4. Event-Driven Architecture Detection

For projects built around async message brokers (RabbitMQ, Kafka, SQS, Redis PubSub):

### A. Event Publishers / Producers
*   **What it is**: Components that construct events and publish them to a broker.
*   **Folder names**: `producers/`, `publishers/`, `emitters/`, `events/`.
*   **Indicators**: Methods calling `.publish()`, `.emit()`, `.send()`, or broker SDK client methods.

### B. Event Channels / Brokers
*   **What it is**: Message routing, queues, topic names, and broker setup configurations.
*   **Folder names**: `config/queue/`, `topics/`, `queues/`, `broker/`.
*   **Indicators**: YAML/JSON configuration mappings for queues, exchange configurations, or broker connection definitions.

### C. Event Handlers / Consumers
*   **What it is**: Code triggered by incoming events to execute tasks.
*   **Folder names**: `consumers/`, `handlers/`, `subscribers/`, `listeners/`.
*   **Indicators**: Functions marked with listener annotations (e.g., `@RabbitListener`, `@KafkaListener`, `.on('message')`) that process data payloads and trigger services.

---

## 5. Monorepos & Microservices Directory Management

When encountering project layouts containing multiple distinct applications:

*   **Detection**: Look for a root containing `packages/`, `apps/`, or `services/` alongside a root-level workspace configuration (e.g. `pnpm-workspace.yaml`, `lerna.json`, `nx.json`).
*   **Handling Strategy**:
    1.  **Do not force a single 3-layer model on the root directory**.
    2.  Identify individual service/app directories (e.g. `apps/api`, `apps/web`).
    3.  Classify each subdirectory independently using the rules above.
    4.  Document the relationship between packages in `project-overview.md` and use `layer-integration.md` to show how these apps interact (e.g., via HTTP REST, gRPC, or events).
