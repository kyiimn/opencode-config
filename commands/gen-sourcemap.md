---
description: "Generates SOURCEMAP.md — a condensed source map for AI coding agents. Reduces context-loading time and token consumption by providing a pre-scanned project overview. Delegates codebase exploration to parallel background agents."
agent: atlas
subtask: false
---

# /gen-sourcemap: $ARGUMENTS

> **Purpose**: Generate `SOURCEMAP.md` so that AI coding agents can understand
> the project structure, module responsibilities, key types, and dependency graph
> **without reading every source file individually**.
> This drastically reduces the tokens and time spent on initial codebase orientation.

Usage:

```
/gen-sourcemap                         # run from project root or monorepo package directory
/gen-sourcemap apps/my-api             # pass target path explicitly (monorepo)
/gen-sourcemap packages/core           # works for any subdirectory
```

---

## Execution Rules

| Rule | Detail |
|------|--------|
| **Output location** | `SOURCEMAP.md` is written to `TARGET_ROOT/` only — never to the monorepo root |
| **Overwrite is safe** | An existing `SOURCEMAP.md` is overwritten without warning |
| **Source is authoritative** | Actual source files are the single source of truth |
| **Parallel exploration** | Use `@explore` and `@librarian` in background for speed |
| **Abort conditions** | No `src/` or equivalent source directory found |
| **Language** | All output in English |
| **Progress tracking** | Use `TodoWrite` to track every phase. Mark `in_progress` on start, `completed` on finish. Only one todo `in_progress` at a time. |

---

## Initialization: Create Todo List

Before any work begins, call `TodoWrite` to register the full task plan:

```
TodoWrite([
  { id: "resolve-root",       content: "Resolve TARGET_ROOT",                    status: "in_progress", priority: "high" },
  { id: "validate-project",   content: "Validate project and collect metadata",  status: "pending",     priority: "high" },
  { id: "parallel-explore",   content: "Parallel codebase exploration (agents)", status: "pending",     priority: "high" },
  { id: "synthesize",         content: "Synthesize and classify modules",        status: "pending",     priority: "high" },
  { id: "generate-sourcemap", content: "Generate SOURCEMAP.md",                  status: "pending",     priority: "high" },
  { id: "update-agents-md",   content: "Update AGENTS.md references",            status: "pending",     priority: "medium" },
  { id: "completion-report",  content: "Completion report",                       status: "pending",     priority: "low" }
])
```

---

## PHASE 0: Resolve TARGET_ROOT

> `TodoWrite`: `resolve-root` → `in_progress`

### 0-1. Check for an explicit path argument

If `$ARGUMENTS` is non-empty, treat it as a relative or absolute path and set:

```
TARGET_ROOT = $ARGUMENTS
```

Verify the path exists:

```bash
!ls $ARGUMENTS/package.json $ARGUMENTS/tsconfig.json 2>/dev/null || echo "NOT_FOUND"
```

If the path does not exist or contains no recognizable project files, abort:

```
⛔ Path not found or not a project: $ARGUMENTS
   Provide a valid relative path from the repository root (e.g. apps/my-api).
```

Skip PHASE 0-2 and 0-3 — proceed directly to PHASE 1.

---

### 0-2. No argument — detect execution context

```bash
!pwd
!ls package.json tsconfig.json src/ apps/ packages/ pnpm-workspace.yaml 2>/dev/null
```

| Condition | Classification |
|-----------|----------------|
| `pnpm-workspace.yaml` or (`apps/` / `packages/`) exists AND no `src/` | Monorepo root |
| `src/` and `package.json` exist, no `apps/` / `packages/` | Project root |
| Inside `apps/*` or `packages/*` | Package directory |

- **Project root or package directory** — set `TARGET_ROOT = current directory`, skip 0-3.
- **Monorepo root** — proceed to 0-3.

---

### 0-3. Select target package (monorepo root, no argument)

```bash
!ls apps/ packages/ 2>/dev/null
```

Prompt via `ask_user_input_v0`:

```
Which package should SOURCEMAP.md be generated for?
  - apps/xxx
  - apps/yyy
  - packages/zzz
  - ...
```

Set `TARGET_ROOT` to the selected path and continue.

> `TodoWrite`: `resolve-root` → `completed`

---

## PHASE 1: Project Validation and Context Collection

> `TodoWrite`: `validate-project` → `in_progress`

### 1-1. Verify source directory exists

```bash
!ls $TARGET_ROOT/src/ 2>/dev/null || ls $TARGET_ROOT/lib/ 2>/dev/null || echo "NO_SOURCE"
```

If no source directory is found, abort:

```
⛔ No src/ or lib/ directory found in $TARGET_ROOT.
   SOURCEMAP.md requires a recognizable source directory.
```

### 1-2. Collect project metadata

```bash
!cat $TARGET_ROOT/package.json
!cat $TARGET_ROOT/tsconfig.json 2>/dev/null
```

Extract and note:
- Project name and version
- Framework/runtime (Express, React, CLI, library, etc.)
- Key dependencies (top 10 by relevance)
- TypeScript configuration (paths, aliases, baseUrl)
- Entry point(s)

### 1-3. Collect existing documentation (if any)

Read the following files if they exist — these provide supplementary context but **source files are always authoritative**:

```bash
!ls $TARGET_ROOT/ARCHITECTURE.md $TARGET_ROOT/AGENTS.md $TARGET_ROOT/README.md $TARGET_ROOT/ENDPOINT.md 2>/dev/null
```

If found, read them to inform module descriptions but do not rely on them exclusively.

> `TodoWrite`: `validate-project` → `completed`

---

## PHASE 2: Parallel Codebase Exploration

> `TodoWrite`: `parallel-explore` → `in_progress`

Delegate exploration to background agents for maximum speed.
Launch all tasks in this phase **simultaneously** using `run_in_background: true`.

### 2-1. Directory structure scan — `@explore`

```
task(subagent_type="explore", run_in_background=true, prompt="""
Scan $TARGET_ROOT and produce a complete directory tree (3 levels deep from src/).
For each directory, note:
  - Number of files
  - Primary file types (.ts, .tsx, .test.ts, etc.)
  - Apparent purpose (inferred from directory name and file names)

Exclude: node_modules/, dist/, build/, .git/, coverage/, .next/
Output as a structured text listing.
""")
```

### 2-2. Export/type analysis — `@explore`

```
task(subagent_type="explore", run_in_background=true, prompt="""
Scan all .ts and .tsx files under $TARGET_ROOT/src/.
For each file, extract:
  1. All named exports (functions, classes, types, interfaces, constants, enums)
  2. Default export (if any) — name and kind
  3. Re-exports from other modules

Focus on PUBLIC API surface. Skip test files (*.test.ts, *.spec.ts) and mock files.
Output as: filepath → list of exports with their kind (function/class/type/interface/const/enum).
""")
```

### 2-3. Dependency and import graph — `@explore`

```
task(subagent_type="explore", run_in_background=true, prompt="""
Analyze import statements across all .ts/.tsx files under $TARGET_ROOT/src/.
Build a module dependency graph showing:
  1. Which internal modules import from which other internal modules
  2. Key external package imports per module (group by directory/layer)
  3. Circular dependency candidates (if any)

Output as a directed adjacency list: module → [imports from].
""")
```

### 2-4. Documentation and pattern extraction — `@librarian`

```
task(subagent_type="librarian", run_in_background=true, prompt="""
Analyze $TARGET_ROOT/src/ to identify:
  1. Architectural patterns in use (DDD layers, MVC, Clean Architecture, etc.)
  2. Key abstractions and base classes/interfaces that other modules extend
  3. Configuration and environment variable patterns
  4. Error handling patterns (custom error classes, error middleware)
  5. Notable JSDoc/TSDoc comments that describe module-level purpose

Output a concise pattern summary (not full code).
""")
```

Wait for all background tasks to complete. Collect their outputs.

> `TodoWrite`: `parallel-explore` → `completed`

---

## PHASE 3: Synthesize and Classify

> `TodoWrite`: `synthesize` → `in_progress`

Merge the outputs from all background agents.

### 3-1. Classify modules by layer/domain

Group the discovered modules into logical layers or domains based on directory structure and import patterns:

| Layer/Domain | Description |
|-------------|-------------|
| Entry points | Main application bootstrap, server startup, CLI entry |
| Routes/Controllers | HTTP route handlers, CLI command handlers |
| Services/Use cases | Business logic, orchestration |
| Domain/Models | Entities, value objects, domain types |
| Repositories/Data | Database access, external API clients |
| Shared/Utils | Cross-cutting utilities, helpers, constants |
| Config | Configuration loading, environment parsing |
| Types/DTOs | Shared type definitions, request/response schemas |
| Tests | Test structure overview (do NOT list individual tests) |

Adapt the layer names to match the actual project structure. Do not force a classification scheme that does not fit.

### 3-2. Identify key types and interfaces

From the export analysis, select the **most important** types:
- Types that appear in 3+ import sites
- Types used in public API boundaries (request/response, function signatures)
- Base classes and interfaces that others extend/implement
- Enum types that define domain constants

For each, note: name, kind, file location, brief purpose (1 line).

> `TodoWrite`: `synthesize` → `completed`

---

## PHASE 4: Generate SOURCEMAP.md

> `TodoWrite`: `generate-sourcemap` → `in_progress`

Write `$TARGET_ROOT/SOURCEMAP.md` with the following structure.

> **Formatting rules:**
> - Keep descriptions concise — 1 line per file/module where possible
> - Use tables for structured data
> - Use fenced code blocks only for directory trees
> - Do not include full source code — only signatures and descriptions
> - Mark auto-generated timestamp so agents know how fresh the map is

````markdown
# SOURCEMAP — {project-name}

> Auto-generated by `/gen-sourcemap` on {YYYY-MM-DD HH:mm}.
> This file helps AI coding agents orient quickly without scanning every source file.
> Re-run `/gen-sourcemap` after significant structural changes.

---

## Project Overview

| Field | Value |
|-------|-------|
| Name | {name} |
| Version | {version} |
| Framework | {framework} |
| Language | TypeScript |
| Entry Point | {entry file(s)} |
| Package Manager | {pnpm/yarn/npm} |

**Key Dependencies**: {dep1}, {dep2}, {dep3}, ...

---

## Directory Structure

```
src/
├── {dir1}/          # {purpose} ({N} files)
│   ├── {subdir}/    # {purpose}
│   └── ...
├── {dir2}/          # {purpose} ({N} files)
└── ...
```

---

## Module Map

### {Layer/Domain Name}

| File | Exports | Purpose |
|------|---------|---------|
| `src/{path}/{file}.ts` | `FunctionA`, `ClassB`, `TypeC` | {1-line description} |
| `src/{path}/{file2}.ts` | `default: Router` | {1-line description} |

_(repeat for each layer/domain)_

---

## Key Types

| Type | Kind | Location | Purpose |
|------|------|----------|---------|
| `{TypeName}` | interface | `src/{path}` | {1-line description} |
| `{EnumName}` | enum | `src/{path}` | {1-line description} |
| `{ClassName}` | class | `src/{path}` | {1-line description} |

---

## Dependency Graph

```
{module-a} → {module-b}, {module-c}
{module-b} → {module-d}
{module-c} → {module-d}, {module-e}
```

{Note any circular dependencies here if found.}

---

## Architectural Patterns

- **{Pattern}**: {1-line description of how it is applied}
- **{Pattern}**: {1-line description}

---

## Configuration

| Variable / Config | Source | Purpose |
|-------------------|--------|---------|
| `{ENV_VAR}` | `.env` | {purpose} |
| `{config.key}` | `{config file}` | {purpose} |

---

## Cross-References

| Document | Purpose |
|----------|---------|
| `ARCHITECTURE.md` | Stack and project structure details |
| `AGENTS.md` | Agent rules and commands |
| `ENDPOINT.md` | API endpoint documentation |
| `RULES.md` | Coding standards |

_(only list files that actually exist)_
````

> `TodoWrite`: `generate-sourcemap` → `completed`

---

## PHASE 5: Update AGENTS.md

> `TodoWrite`: `update-agents-md` → `in_progress`

After `SOURCEMAP.md` is written, register it in existing `AGENTS.md` files.

### 5-1. Project-level AGENTS.md

```bash
!ls $TARGET_ROOT/AGENTS.md 2>/dev/null || echo "NOT_FOUND"
```

If found, read the file and locate the most appropriate insertion point — a section that lists project files, references, or documentation artifacts.

Insert the following block. Do not duplicate if a `SOURCEMAP.md` entry already exists — update the existing entry instead.

```markdown
### SOURCEMAP.md

Condensed source map for AI coding agents — covers directory structure, module exports,
key types, and dependency graph. Read this file FIRST before scanning source files.
Generated by `/gen-sourcemap` — re-run after significant structural changes.
```

Edit with a minimal, targeted insertion — do not rewrite or reformat existing content.

### 5-2. Monorepo root AGENTS.md (if applicable)

This step runs only when `TARGET_ROOT` is a subdirectory of a monorepo.

```bash
!git rev-parse --show-toplevel 2>/dev/null || echo "NOT_GIT"
```

If the repository root differs from `TARGET_ROOT` and a root `AGENTS.md` exists:

```bash
!ls {REPO_ROOT}/AGENTS.md 2>/dev/null || echo "NOT_FOUND"
```

If found, add a reference entry under the appropriate package listing section:

```markdown
- `{relative-path-to-package}/SOURCEMAP.md` — Source map for {package-name}
```

Do not duplicate. Do not modify any other content.

> `TodoWrite`: `update-agents-md` → `completed`

---

## PHASE 6: Completion Report

> `TodoWrite`: `completion-report` → `in_progress`

```
✅ SOURCEMAP.md generated
   Path      : $TARGET_ROOT/SOURCEMAP.md
   Modules   : {N} files mapped across {N} layers/domains
   Key Types : {N} types documented
   AGENTS.md : {updated | not found | already registered}
```

> `TodoWrite`: `completion-report` → `completed`

Call `TodoRead` to confirm all items are `completed`.
If any item remains `pending` or `in_progress`, investigate and resolve before finishing.
