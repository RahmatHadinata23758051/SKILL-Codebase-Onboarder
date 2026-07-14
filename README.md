# Codebase Onboarder

An agent skill designed to analyze, map, and document existing codebases during developer transitions or project onboarding. 

Instead of reading the entire codebase, this skill uses a progressive scanning strategy to classify project files into architectural layers (such as Presentation/Logic/Data, Ports/Adapters, or Event-Driven Pub-Sub channels) and generate persistent local documentation.

---

## Agent Compatibility

This skill uses tool-agnostic descriptions for file system access, text search, and code viewing. It is compatible with any AI coding assistant that supports the Open Agent Skills specification, including:

*   **Google Antigravity IDE**
*   **Claude Code** (Anthropic CLI)
*   **Windsurf** (Cascade by Codeium)
*   **Cursor** (via custom instruction injection)

---

## Installation & Distribution

You can distribute and install this skill directly using the Skills CLI or by cloning the repository.

### Method 1: Using the Skills CLI (Recommended)
Install this skill directly from GitHub into your local or global agent environment:

```bash
# Global installation (recommended for use across all codebases)
npx skills add RahmatHadinata23758051/SKILL-Codebase-Onboarder -g -y

# Local installation (scoped to current workspace root)
npx skills add RahmatHadinata23758051/SKILL-Codebase-Onboarder -y
```

### Method 2: Manual Installation
Clone this folder directly into your agent's customization directory:
*   **Global Path**: `~/.agents/skills/codebase-onboarder/`
*   **Project-specific Path**: `.agents/skills/codebase-onboarder/` at the root of your workspace.

---

## Problem Solved

When taking over or backing up an existing project, understanding the system structure usually requires either manual code reading or feeding the entire repository into an AI context window. The latter approach consumes excessive API tokens and often leads to context overflow.

This skill automates codebase mapping by scanning only structure and structural code files (imports, class/function signatures, models, and routes). It saves token costs while producing a reliable overview of the system architecture.

---

## Folder Structure

Once executed, the skill creates the following local directory structure at the project root to store the generated analysis:

```text
.agents/
└── skills/
    └── codebase-onboarder/
        ├── project-overview.md          # Tech stack, framework, entry points, and directory map
        ├── presentation-layer.md        # Routes, views, controllers, and UI state management (Layered/MVC)
        ├── logic-layer.md               # Core business services, rules, and external integrations (Layered/MVC)
        ├── data-layer.md                # Database schemas, ORM models, and migration logs (Layered/MVC)
        ├── core-domain.md               # Entities, aggregates, and domain services (Clean/Hexagonal)
        ├── ports-and-adapters.md        # Ports interfaces and their implementations (Clean/Hexagonal)
        ├── event-publishers.md          # Event schemas and trigger conditions (Event-Driven)
        ├── event-channels.md            # Message queues and broker configs (Event-Driven)
        ├── event-handlers.md            # Consumers, subscribers, and background tasks (Event-Driven)
        ├── layer-integration.md         # Data flow description and interaction diagrams
        ├── issues-and-bugs.md           # Log of detected anti-patterns, code smells, or bugs
        └── diagrams/
            ├── architecture-overview.mmd    # General component layout
            ├── layer-integration.mmd        # Inter-layer communication diagram
            └── data-flow.mmd                # Sequence diagram of main request lifecycles
```

---

## Workflow Phases

The skill directs the agent through seven execution phases:

1.  **Pre-Flight Check**: Checks if a previous analysis exists in the `.agents/` directory to avoid redundant scanning.
2.  **Structure Scan**: Lists directories and excludes dependency folders (`node_modules`, `venv`, etc.) to map out folder paths.
3.  **Manifest Read**: Parses package configuration files (e.g. `package.json`, `requirements.txt`) to identify languages and frameworks.
4.  **Layer Classification**: Categorizes files into Presentation, Logic, and Data folders (or Clean/Event-driven boundaries) using framework-specific patterns.
5.  **Deep Scan**: Inspects only routing setups, interface declarations, database models, and service headers to build summaries without loading function bodies.
6.  **Integration Mapping**: Traces how modules communicate and checks for architectural layer violations.
7.  **Output Generation**: Saves the markdown files and compiles the Mermaid diagrams.

---

## Usage

This skill activates when you prompt the agent with phrases such as:
- "pahami project ini" / "analyze this codebase"
- "buatkan diagram arsitektur project ini" / "diagram the architecture"
- "bagaimana flow sistem di repo ini" / "explain this codebase structure"

The generated `.md` and `.mmd` files can be read by both developers and agents in subsequent sessions, preventing the need to re-scan the codebase in the future.
