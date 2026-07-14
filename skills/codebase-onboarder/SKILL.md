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
architectural analysis with diagrams — using a progressive scanning strategy that minimizes token consumption.

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

## Phase 0 — Pre-Flight Check & Tool Mapping

### Tool Mapping Guide
Depending on the IDE or agent platform you are running in, map the generic tool terms in this guide to the actual tools available in your context:
- **Directory listing tool** (e.g., `list_dir`, `glob`, `find`, `dir`)
- **File viewing tool** (e.g., `view_file`, `viewFile`, `cat`)
- **Text search tool** (e.g., `grep_search`, `grep`, `ripgrep`)
- **Command execution tool** (e.g., `run_command`, `execute_bash`, `shell`)

### Cache Verification
Before doing any scanning work, check if previous analysis already exists in the project-local directory:

```
<project-root>/.agents/skills/codebase-onboarder/
```

**If the folder exists and contains analysis files:**
1. Read `project-overview.md` first to check the timestamp and scope of the last analysis.
2. Ask the user: "I found existing analysis from [date]. Do you want me to update it, or start fresh?"
3. If updating, only re-scan files that changed (use git command checks or file modification dates if available).

**If the folder does NOT exist:**
- Proceed to Phase 1. Create the folder structure during Phase 7 (Output Generation).

**If the `.agents/` directory does not exist at the project root:**
- Create it. Ensure you write to the project-local `.agents/` folder, NOT the global agent config folder.

---

## Phase 1 — Structure Scan (Token Budget: ~200-500 tokens)

Goal: Build a mental map of the project without reading file contents.

1. **Use your directory listing tool** on the project root to inspect top-level folders and files.
2. **Identify entry points**: Look for filenames starting with `main`, `index`, `app`, `manage.py`, `server`, `Program.cs`, `Application`.
3. **Identify config files**: Look for files like `package.json`, `requirements.txt`, `Pipfile`, `pyproject.toml`, `pom.xml`, `build.gradle`, `Cargo.toml`, `go.mod`, `composer.json`, `Gemfile`, `pubspec.yaml`, `.csproj`.
4. **Exclude irrelevant folders** from all listing operations:
   - Dependency folders: `node_modules/`, `vendor/`, `venv/`, `.venv/`
   - Build directories: `dist/`, `build/`, `out/`, `.next/`, `.nuxt/`
   - Metadata/Cache: `.git/`, `__pycache__/`, `.antigravity/`
5. **Inspect key subdirectories** (max 3-4 levels deep) to build the core folder tree.

**Output:** A mental folder tree with annotations about what each top-level folder likely contains.

---

## Phase 2 — Manifest Read (Token Budget: ~300-800 tokens)

Goal: Determine the tech stack, framework, and dependencies.

1. **Read the primary config file** identified in Phase 1 (e.g., `package.json`, `requirements.txt`).
   - Read only the relevant sections (like `dependencies`, `scripts`) using your file viewing tool. Do not load the entire file if it is very large.
2. **Read the first 50-80 lines of README.md** if present to understand the project setup.
3. **Detect the framework and language** from dependencies (e.g. Django, Laravel, Next.js, Spring Boot, etc.).
4. **Record the detected stack** to determine which classification rules to use.

---

## Phase 3 — Architecture & Layer Classification (Token Budget: ~500-1000 tokens)

Goal: Map files and folders to an architectural model. Do not force a 3-layer model onto systems where it does not fit.

### Step 3.1: Choose the Architectural Pattern
Identify which pattern the codebase uses and apply the corresponding classification from `references/layer-detection.md`:

- **Layered / MVC Architecture** (Default):
  - **Presentation**: UI views, controllers, templates, routes.
  - **Logic**: Services, use-cases, business rules, handlers.
  - **Data**: Database models, repositories, schemas, migrations.
- **Clean / Hexagonal Architecture**:
  - **Core/Domain**: Pure business entities and use cases (no external dependencies).
  - **Ports/Adapters**: Interfaces and code connecting to external elements (DB repositories, controller HTTP inputs, external APIs).
- **Event-Driven Architecture**:
  - **Publishers/Producers**: Components emitting events or messages.
  - **Channels/Queues**: Topics, message brokers, routes.
  - **Handlers/Consumers**: Code processing messages and updating state.
- **Monorepos & Microservices**:
  - Treat each app/package as a separate domain. Scan them independently and document how they connect.

### Step 3.2: Classify Code Paths
Group folders and files according to the chosen pattern. Use your text search tool to check import patterns if folder boundaries are unclear (e.g., searching for database client imports vs view component imports).

---

## Phase 4 — Deep Scan (Token Budget: ~2000-4000 tokens)

Goal: Extract summaries from key structural files.

**CRITICAL TOKEN-SAVING RULES:**
- **Do NOT read entire source files.** Use your file viewing tool with line range parameters to retrieve only imports, class definitions, interface declarations, and method/function signatures. Ignore function bodies.
- **Limit file inspections**: Read at most 5-8 files per architectural domain.
- **Use your text search tool** to inspect signatures (e.g., grep for `class `, `def `, `function `, `@route`, or database mapping decorators).

**For each architectural type, extract:**
- **Layered/MVC**: Page routes, service public methods, database models, and migration states.
- **Hexagonal**: Domain entities, ports (interface definitions), and adapters (implementations).
- **Event-Driven**: Events emitted, broker configs, consumers, and background queues.
- **Monorepo/Microservices**: Inter-service APIs, environment variables, and Docker/k8s configurations.

---

## Phase 5 — Integration Mapping (Token Budget: ~1000-2000 tokens)

Goal: Trace communication and data flows between components.

1. **Trace the Request/Message Lifecycle**: Trace 1-2 representative flows (e.g., UI → Router → Controller → Service → Repository → DB, or Producer → Broker → Consumer).
2. **Map Dependency Imports**: Scan which packages/modules import from other modules using your text search tool.
3. **Identify Architectural Violations**: Flag files that bypass designated boundaries (e.g., UI components making direct database queries, or domain logic importing database libraries directly).

---

## Phase 6 — Bug & Issue Detection (Token Budget: ~500-1000 tokens)

Note any structural issues found during Phase 3-5. Categorize them as:
- **Architecture Violations**: Cross-layer bypasses, circular dependencies, domain model leaks.
- **Code Smells**: Very long functions (>100 lines), nested conditionals, duplicate logic, hardcoded configurations.
- **Security Concerns**: Hardcoded credentials/API keys, raw SQL input bindings (injection risk), missing input sanitization.
- **Performance Risks**: N+1 query patterns, lack of indexing on query parameters.

**Anti-Hallucination Guardrail**: Only log issues you actually observe in the codebase. If an issue is suspected but not confirmed, prefix it with "Potential: " and explain why.

---

## Phase 7 — Output Generation

Write all documentation files to the local folder: `<project-root>/.agents/skills/codebase-onboarder/`. Use the templates defined in `references/output-templates.md`.

1.  **`project-overview.md`**: Tech stack, entry points, high-level folder tree, and overview diagram.
2.  **Architectural Reports**:
    - For Layered/MVC: Create `presentation-layer.md`, `logic-layer.md`, and `data-layer.md`.
    - For Clean/Hexagonal: Create `core-domain.md` and `ports-and-adapters.md`.
    - For Event-Driven: Create `event-publishers.md`, `event-channels.md`, and `event-handlers.md`.
3.  **`layer-integration.md`**: Integration map and request lifecycle flows with embedded Mermaid diagrams.
4.  **`issues-and-bugs.md`**: Log of detected code smells, bugs, and violations (only create if issues were found).

### Diagrams to Generate (stored in `diagrams/` folder)
- `diagrams/architecture-overview.mmd`
- `diagrams/layer-integration.mmd`
- `diagrams/data-flow.mmd`

Follow standard diagram guidelines: use actual names, show decision points explicitly, collapse noise, and keep each diagram under 15-20 nodes.

---

## Quick Reference

| User Query | Action |
|---|---|
| "pahami project ini" / "analyze this codebase" | Run full Phase 0-7 |
| "update analisis project" | Run Phase 0 (check git diff / file changes) → re-scan only updated components |
| "gambarkan arsitektur" / "show architecture" | Display or generate `diagrams/architecture-overview.mmd` |
| "ada bug apa?" | Display `issues-and-bugs.md` |

---

## References

- `references/token-strategy.md` — Selective code reading and token saving guidelines
- `references/layer-detection.md` — Identification tables for various architectures
- `references/output-templates.md` — Templates for layered, clean, and event-driven reports
- `references/embedding.md` — Embedding diagrams in exports
