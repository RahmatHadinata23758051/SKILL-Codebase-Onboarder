# Token-Saving Scanning Strategy

This reference guide provides specific instructions for agents to perform codebase analysis while keeping token consumption to a minimum. Brute-force reading of every file in a project will exhaust context windows and cause high latency and API costs.

---

## 1. The Principle of Progressive Discovery

Instead of reading full files or folders upfront, use a **hierarchical discovery pattern**:
1. **Metadata first**: Get the directory structures and package manifests using the directory listing tool.
2. **Scoping**: Identify the core entry points and high-value directories.
3. **Targeted Reading**: Retrieve only class/function headers and signatures, never function bodies, unless specifically requested.

---

## 2. Directory & Structure Scanning Rules

### Exclusions list
Always ignore the following folders when using commands or directory listing tools:
*   **Dependency Folders**: `node_modules/`, `vendor/`, `venv/`, `.venv/`, `packages/`
*   **Build/Dist Folders**: `dist/`, `build/`, `out/`, `.next/`, `.nuxt/`, `.svelte-kit/`
*   **System/Metadata**: `.git/`, `.idea/`, `.vscode/`, `.antigravity/`
*   **Assets/Static**: `images/`, `assets/`, `public/static/`, `fonts/`
*   **Caches**: `__pycache__/`, `.pytest_cache/`, `.mypy_cache/`

### Scoping large directories
If a subdirectory has more than 50 files, do NOT list every file. Inspect the top-level files to identify the naming convention, then use patterns to filter (e.g. searching only for `*.ts` or `*.py` files).

---

## 3. Targeted Code Reading Techniques

### Reading imports and signatures
When inspecting a source file, do not load the whole file. Read only the top 30-50 lines to extract imports and the top-level declarations.
*   **For Python**: Look for `def` and `class` definitions.
*   **For JavaScript/TypeScript**: Look for `interface`, `class`, `function`, and `export` definitions.
*   **For Java/C#**: Look for `public class`, `interface`, and method declarations.

### Using line ranges
Use your file viewing tool's line range parameters (such as `StartLine` and `EndLine` constraints) to restrict code fetching to specific lines. 

Example workflow:
1. Run your text search tool with a query like `class ` or `def ` or `function ` in the target file.
2. The search results will return matching line numbers.
3. Call your file viewing tool passing only the line ranges containing the declarations and imports.

---

## 4. Smart Search Filters

Instead of checking files manually, let the search tool do the filtering. Here are optimized regular expression queries for framework analysis:

*   **Endpoint detection**:
    *   Express: `router\.(get|post|put|delete|use)\(`
    *   FastAPI: `@app\.(get|post|put|delete)\(`
    *   Django: `path\(` or `re_path\(`
*   **Database models**:
    *   SQLAlchemy: `class.*\(Base\)` or `Column\(`
    *   Mongoose: `new Schema\(`
    *   Prisma: `model ` in schema files
*   **Dependencies**:
    *   Look for `import ` or `require(` statements to build dependency lists.

---

## 5. Reusing Existing Analysis (The Cache Pattern)

The `.agents/skills/codebase-onboarder/` folder serves as a local cache.
*   Before scanning a file, check if its summary or description is already present in the corresponding domain report (e.g. `data-layer.md` or `ports-and-adapters.md`).
*   If the file's modification date has not changed since the last analysis, **do not re-read it**. Reuse the summary from the previous report.
