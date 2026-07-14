---
name: codebase-onboarder
description: >
  Use this skill when a user needs to understand, onboard onto, or analyze an existing codebase or project.
  Trigger on requests like: "pahami project ini", "analyze this codebase", "onboard me to this repo",
  "visualisasikan flow sistem ini", "gambarkan alur kode ini", "how does this project work",
  "diagram the architecture", "show me the layers", "buatkan diagram arsitektur project ini",
  "I just joined this project, help me understand it", "explain this codebase structure",
  or any request to map out, understand, or document how an existing codebase is structured and behaves.
  Also trigger proactively when a user points at a repo/project and asks for an explanation of its
  architecture, layer structure, data flow, or integration points — even if they don't explicitly
  say "visualize" or "diagram". Do NOT use for generic UI mockups unrelated to explaining existing
  logic (use frontend-design for that instead).
---

# Codebase Onboarder

**Project onboarding and codebase understanding tool.** Turn any existing project into a structured,
layered analysis with diagrams — using a progressive scanning strategy that minimizes token consumption.

The skill produces persistent analysis files stored in the project's `.agents/skills/codebase-onboarder/`
directory, so agents (and developers) can reference them in future sessions without re-scanning the
entire codebase.

---

## When to Use This Skill

- Developer joining an existing project (backup team, developer transition)
- Understanding unfamiliar codebase architecture
- Documenting an undocumented project
- Visualizing how layers/modules connect
- Pre-work analysis before making changes to legacy code
- Identifying architectural issues, code smells, or bugs in a project

---

## Phase 0 — Pre-Flight Check

Before doing any scanning work, check if previous analysis already exists:

```
<project-root>/.agents/skills/codebase-onboarder/
```

**If the folder exists and contains analysis files:**
1. Read `project-overview.md` first — it has the summary and the timestamp of last analysis.
2. Ask the user: "I found existing analysis from [date]. Do you want me to update it, or start fresh?"
3. If updating, only re-scan files that changed (use `git diff` or file modification dates if git is available).

**If the folder does NOT exist:**
- Proceed to Phase 1. Create the folder structure during Phase 7 (Output Generation).

**If `.agents/` directory doesn't exist at the project root:**
- Create it. This is the project-local agents folder, NOT the global `~/.agents/` or `~/.gemini/config/` folder.

---

## Phase 1 — Structure Scan (Token Budget: ~200-500 tokens)

Goal: Build a mental map of the project without reading any file contents.

1. **Run `list_dir` on project root** — get top-level folders and files.
2. **Identify entry points** — look for: `main.*`, `index.*`, `app.*`, `manage.py`, `server.*`, `Program.cs`, `Application.*`.
3. **Identify config files** — look for: `package.json`, `requirements.txt`, `Pipfile`, `pyproject.toml`, `pom.xml`, `build.gradle`, `Cargo.toml`, `go.mod`, `composer.json`, `Gemfile`, `pubspec.yaml`, `.csproj`.
4. **Skip irrelevant directories entirely** — never scan into:
   - `node_modules/`, `vendor/`, `venv/`, `.venv/`, `__pycache__/`, `.git/`, `dist/`, `build/`, `.next/`, `.nuxt/`
   - Test fixtures, static assets (images/fonts), IDE configs
5. **Run `list_dir` on key subdirectories** (max 3-4 levels deep) to build the folder tree.

**Output of this phase:** A mental folder tree with annotations about what each top-level folder likely contains.

---

## Phase 2 — Manifest Read (Token Budget: ~300-800 tokens)

Goal: Determine the tech stack, framework, and dependencies.

1. **Read the primary config file** identified in Phase 1 (e.g., `package.json`, `requirements.txt`).
   - Read only the relevant sections: `dependencies`, `scripts`, `[project]` — not the entire file if it's large.
2. **Read README.md** if it exists — but only the first 50-80 lines (usually contains project description and setup).
3. **Detect the framework and language** from dependencies:

| Indicator | Framework |
|---|---|
| `django` in requirements | Django (Python) |
| `flask` in requirements | Flask (Python) |
| `fastapi` in requirements | FastAPI (Python) |
| `next` in package.json | Next.js (JavaScript/TypeScript) |
| `express` in package.json | Express (Node.js) |
| `react` in package.json | React (JavaScript/TypeScript) |
| `@angular/core` in package.json | Angular (TypeScript) |
| `vue` in package.json | Vue.js (JavaScript/TypeScript) |
| `laravel/framework` in composer.json | Laravel (PHP) |
| `spring-boot` in pom.xml/build.gradle | Spring Boot (Java/Kotlin) |
| `rails` in Gemfile | Ruby on Rails |
| `flutter` in pubspec.yaml | Flutter (Dart) |
| `.csproj` with `Microsoft.AspNetCore` | ASP.NET Core (C#) |

4. **Record the detected stack** — this determines which layer-detection rules to use in Phase 3.

**Output of this phase:** Stack identification (language, framework, major libraries).

---

## Phase 3 — Layer Classification (Token Budget: ~500-1000 tokens)

Goal: Map every significant folder/file to one of three architectural layers.

Use the framework-specific detection rules from `references/layer-detection.md`. For each folder in the project, classify it as:

| Layer | What it contains | Common folder names |
|---|---|---|
| **Presentation** | UI views, templates, components, controllers that handle HTTP requests, routing definitions, static assets, frontend state management | `views/`, `templates/`, `pages/`, `components/`, `screens/`, `controllers/`, `routes/`, `public/` |
| **Logic** | Business rules, services, use cases, domain logic, middleware, validators, event handlers, utilities | `services/`, `usecases/`, `domain/`, `middleware/`, `utils/`, `helpers/`, `lib/`, `core/`, `business/` |
| **Data** | Database models, schemas, repositories, migrations, ORM configs, data access objects, API clients for external data | `models/`, `entities/`, `repositories/`, `schemas/`, `migrations/`, `database/`, `dao/`, `prisma/`, `api/` |

**Rules for classification:**
- A file/folder can belong to multiple layers (e.g., a Django `views.py` is both Presentation and Logic) — note the overlap.
- If a folder doesn't clearly fit any layer, classify it as **Infrastructure** (configs, CI/CD, Docker, deployment) and note it separately.
- Use `grep_search` to check for import patterns if folder names are ambiguous — e.g., search for `import.*models` or `from.*services` to trace dependencies.

**Output of this phase:** A classification map — every significant folder tagged with its layer.

---

## Phase 4 — Deep Scan per Layer (Token Budget: ~2000-4000 tokens)

Goal: Extract meaningful summaries from key files in each layer.

**CRITICAL TOKEN-SAVING RULES:**
- **Do NOT read entire files.** Use `view_file` with `StartLine`/`EndLine` to read only:
  - Import statements (first 20-30 lines)
  - Class/function definitions (signatures only, not implementations)
  - Route/URL definitions
  - Model field definitions
- **Max 5-8 files per layer.** Prioritize in this order:
  1. Entry points (main app file, root router)
  2. Route/URL configuration files
  3. Service/use-case files (the core business logic)
  4. Model/entity definitions
  5. Key middleware or configuration
- **Use `grep_search` to find patterns** instead of reading files:
  - `class\s+\w+` to find all class definitions
  - `def\s+\w+|function\s+\w+|const\s+\w+\s*=` to find function signatures
  - `@(route|app\.|router\.|Get|Post|Put|Delete|RequestMapping)` to find API endpoints
  - `(import|from|require|use)\s+` to trace dependencies

**For each layer, extract:**

### Presentation Layer
- List of pages/views/components with their routes/URLs
- Navigation structure (what links to what)
- State management approach (Redux, Vuex, Context, BLoC, etc.)
- Template engine or rendering approach

### Logic Layer
- List of services/use-cases with their public methods
- Business rules and validation logic (summarize, don't copy)
- Event handlers and middleware chain
- External API integrations

### Data Layer
- Database type (PostgreSQL, MySQL, MongoDB, SQLite, etc.)
- Model/entity list with key fields and relationships
- Migration history summary (how many, latest migration name)
- Data access patterns (ORM, raw SQL, repository pattern)

---

## Phase 5 — Integration Mapping (Token Budget: ~1000-2000 tokens)

Goal: Trace how the three layers connect and communicate.

1. **Trace the request lifecycle** — pick 2-3 representative endpoints/features and trace:
   - HTTP request → Controller/View (Presentation)
   - → Service call (Logic)
   - → Database query (Data)
   - → Response transformation → HTTP response

2. **Map import dependencies** between layers using `grep_search`:
   - Which Presentation files import from Logic?
   - Which Logic files import from Data?
   - Are there any layer violations? (e.g., Presentation importing directly from Data, bypassing Logic)

3. **Identify integration patterns:**
   - Direct function calls
   - Event/message-based communication
   - API calls (REST/GraphQL) between services
   - Shared state or global objects

---

## Phase 6 — Bug & Issue Detection (Token Budget: ~500-1000 tokens)

While scanning in Phases 3-5, note any issues found. Categorize them as:

| Category | What to look for |
|---|---|
| **Architecture Violations** | Presentation layer directly accessing Data layer; circular dependencies; God classes/functions |
| **Code Smells** | Functions > 100 lines; deeply nested conditionals; duplicated code patterns; hardcoded values |
| **Security Concerns** | Hardcoded credentials/API keys; SQL injection patterns; missing input validation; exposed debug endpoints |
| **Performance Risks** | N+1 query patterns; missing database indexes (if detectable); synchronous calls that should be async |
| **Missing Patterns** | No error handling; no logging; no tests; no environment-based configuration |

**Important:** Only note issues you actually see in the code. Do NOT hallucinate problems. If you're unsure, prefix with "Potential: " and explain why it might be an issue.

---

## Phase 7 — Output Generation

Generate all analysis files into the project-local directory:

```
<project-root>/.agents/skills/codebase-onboarder/
```

### Files to Generate

Use the templates from `references/output-templates.md` for each file:

1. **`project-overview.md`** — High-level summary: name, stack, framework, entry points, folder structure, analysis timestamp
2. **`presentation-layer.md`** — All Presentation layer findings from Phase 4
3. **`logic-layer.md`** — All Logic layer findings from Phase 4
4. **`data-layer.md`** — All Data layer findings from Phase 4
5. **`layer-integration.md`** — Integration map from Phase 5 + Mermaid diagrams
6. **`issues-and-bugs.md`** — All issues from Phase 6 (only create if issues were found)

### Diagrams to Generate

Create Mermaid diagram files in a `diagrams/` subdirectory:

7. **`diagrams/architecture-overview.mmd`** — High-level component/box diagram showing all major modules
8. **`diagrams/layer-integration.mmd`** — Flow diagram showing how the 3 layers connect
9. **`diagrams/data-flow.mmd`** — Sequence diagram for 1-2 representative request flows

Also embed these diagrams inline (as fenced mermaid blocks) inside the relevant `.md` files for easy reading.

### Diagram Quality Rules

Follow the same rules as the original skill:
- **Name real things** — use actual file/function/service names, not "Module A"
- **Show decision points** — if/else, try/catch, routing conditions
- **Collapse noise** — no trivial getters/setters/loggers as separate nodes
- **Label edges** — protocol, data type, condition on each arrow
- **Max ~15-20 nodes per diagram** — split into sub-diagrams if bigger

---

## After Analysis — What the Agent Should Do

Once all files are generated:

1. **Present a summary** to the user highlighting:
   - The detected stack and architecture pattern
   - How many components were found in each layer
   - Key integration points
   - Critical issues (if any)
2. **Offer to drill down** — "Would you like me to explain any specific layer or component in more detail?"
3. **In future sessions**, when the user asks about this project, read from `.agents/skills/codebase-onboarder/` FIRST before re-scanning the codebase. This is the core token-saving mechanism for ongoing work.

---

## Quick Command Reference

| User says | Agent does |
|---|---|
| "pahami project ini" / "understand this project" | Run full Phase 0-7 |
| "update analisis project" / "refresh the analysis" | Phase 0 (detect changes) → re-run only changed phases |
| "jelaskan layer X" / "explain the data layer" | Read from existing `data-layer.md`, offer to deep-dive |
| "gambarkan arsitektur" / "show architecture" | Read/generate `diagrams/architecture-overview.mmd` |
| "ada bug apa?" / "what issues did you find?" | Read from `issues-and-bugs.md` |
| "trace flow endpoint X" | Run Phase 5 for specific endpoint, append to `layer-integration.md` |

---

## References

- `references/token-strategy.md` — Detailed token-saving scanning techniques
- `references/layer-detection.md` — Framework-specific layer detection rules for 8+ frameworks
- `references/output-templates.md` — Standard templates for all output files
- `references/embedding.md` — How to embed diagrams into `.docx`/`.pdf`/`.pptx` deliverables
