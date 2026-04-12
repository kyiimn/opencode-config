---
description: "Scans an Express-family API project and generates ENDPOINT.md. Source implementation takes absolute priority; .y/contract and .y/plans are supplementary references only."
agent: atlas
subtask: false
---

# /gen-endpoint-docs: $ARGUMENTS

> **Purpose**: Generate `ENDPOINT.md` so that external consumers of this API
> can call every endpoint without reading source code.

Usage:

```
/gen-endpoint-docs                       # run from project root or monorepo package directory
/gen-endpoint-docs apps/my-api           # pass target path explicitly (monorepo)
/gen-endpoint-docs packages/gateway      # works for any subdirectory
```

---

## PHASE 0: Initialize — Register Todo List

Before any other action, call `todowrite` to register all phases as a todo list.
This allows progress to be tracked visibly throughout execution.

```
todowrite([
  { id: "p0", content: "PHASE 0: Resolve TARGET_ROOT",                    status: "in_progress" },
  { id: "p1", content: "PHASE 1: Verify Express-family project",           status: "pending" },
  { id: "p2", content: "PHASE 2: Collect supplementary references",        status: "pending" },
  { id: "p3", content: "PHASE 3: Source scan (routes, schemas, errors)",   status: "pending" },
  { id: "p4", content: "PHASE 4: Organize document structure",             status: "pending" },
  { id: "p5", content: "PHASE 5: Write ENDPOINT.md",                       status: "pending" },
  { id: "p6", content: "PHASE 6: Update AGENTS.md",                        status: "pending" },
  { id: "p7", content: "PHASE 7: Final report",                            status: "pending" },
])
```

---

## PHASE 0: Resolve TARGET_ROOT

> **Agent**: Atlas (self)

### 0-1. Check for an explicit path argument

If `$ARGUMENTS` is non-empty, treat it as a relative or absolute path and set:

```
TARGET_ROOT = $ARGUMENTS
```

Verify the path exists:

```bash
!ls {TARGET_ROOT}/package.json 2>/dev/null || echo "NOT_FOUND"
```

If the path does not exist, abort immediately:

```
Path not found: {TARGET_ROOT}
Provide a valid relative path from the repository root (e.g. apps/my-api).
```

Skip PHASE 0-2 and 0-3 — proceed directly to PHASE 1.

---

### 0-2. No argument — detect execution context

```bash
!pwd
!ls package.json src/ apps/ packages/ 2>/dev/null
```

Determine whether the current directory is a monorepo root or a package directory.

| Condition | Classification |
|-----------|----------------|
| `apps/` or `packages/` exists, and no `src/` | Monorepo root |
| `src/` or `package.json` exists, no `apps/` / `packages/` | Project root |
| Inside `apps/*` or `packages/*` | Package directory |

- **Project root or package directory** — set `TARGET_ROOT = current directory`, skip 0-3.
- **Monorepo root** — proceed to 0-3.

---

### 0-3. Select target package (monorepo root, no argument)

```bash
!ls apps/ packages/ 2>/dev/null
```

Use the `question` tool to prompt:

```
Which package should ENDPOINT.md be generated for?
  - apps/foo
  - apps/bar
  - packages/gateway
  - ...
```

Set `TARGET_ROOT` to the selected path and continue.

Mark `p0` complete:
```
todowrite([{ id: "p0", status: "completed" }, { id: "p1", status: "in_progress" }])
```

---

## PHASE 1: Verify Express-family project

> **Agent**: `@explore` — fast glob/grep on a single file

Delegate to `@explore`:

```
task(
  subagent_type = "explore",
  prompt = """
    Read {TARGET_ROOT}/package.json.
    Check whether `dependencies` or `devDependencies` contains at least one of:
    express, hono, fastify, koa.
    Reply with exactly one line: FOUND:<framework> or NOT_FOUND.
  """
)
```

If NOT_FOUND, abort:

```
⛔ No Express / Hono / Fastify / Koa dependency found in {TARGET_ROOT}/package.json.
   This command only supports Express-family API server projects.
   Verify the target path and re-run.
```

If FOUND, record the detected framework for use in PHASE 5 (Overview table).

Mark `p1` complete:
```
todowrite([{ id: "p1", status: "completed" }, { id: "p2", status: "in_progress" }])
```

---

## PHASE 2: Collect supplementary references (optional)

> **Agent**: `@librarian` — documentation lookup and evidence-based context gathering

> ⚠️ These files are for understanding intent and context only.
> When they conflict with source code, **source always wins**.

Delegate to `@librarian`:

```
task(
  subagent_type = "librarian",
  prompt = """
    Read all files under {TARGET_ROOT}/.y/contract/ (if the directory exists).
    Read all files under {TARGET_ROOT}/.y/plans/ (if the directory exists).
    Return the full content of each file found, prefixed with its path.
    If neither directory exists or both are empty, reply: NO_SUPPLEMENTARY_REFS.
  """
)
```

Keep the returned content as supplementary context in memory.
If NO_SUPPLEMENTARY_REFS, skip silently.

Mark `p2` complete:
```
todowrite([{ id: "p2", status: "completed" }, { id: "p3", status: "in_progress" }])
```

---

## PHASE 3: Source scan

> ⛔ Scan results from this phase are the sole authoritative source for the generated document.
> Supplementary references from PHASE 2 may only enrich descriptions.
> Endpoint list, parameters, and types must be extracted directly from source.

### 3-1. Directory structure and router entry points

> **Agent**: `@explore` — fast codebase grep and file listing

Delegate to `@explore`:

```
task(
  subagent_type = "explore",
  prompt = """
    In {TARGET_ROOT}/src, perform the following and return all results:

    1. List all .ts files (find {TARGET_ROOT}/src -type f -name "*.ts" | sort)
    2. List all directories (find {TARGET_ROOT}/src -type d | sort)
    3. Grep for Express router entry points:
         app.use | router.use | Router() | createRouter | registerRoutes
       across all .ts files (-l flag, filenames only)
    4. Grep for Hono entry points:
         new Hono | app.route | .basePath
       across all .ts files (-l flag, filenames only)

    Return file paths grouped by step number.
  """
)
```

### 3-2. Deep route handler analysis

> **Agent**: `@librarian` — follows import chains, reads handler files, extracts schemas

Using the router entry point files discovered in 3-1, delegate to `@librarian`:

```
task(
  subagent_type = "librarian",
  prompt = """
    Starting from these router entry points in {TARGET_ROOT}/src:
    {ROUTER_FILES_FROM_3_1}

    Recursively follow all imports to their handler files and read each one.

    From every file, extract the following:

    | Item               | Detection method                                                            |
    |--------------------|-----------------------------------------------------------------------------|
    | HTTP method        | .get( .post( .put( .patch( .delete( .head( .options(                        |
    | Path pattern       | String literals '/...' "/..." or path constants                             |
    | Path parameters    | :param or {param} notation                                                  |
    | Query parameters   | req.query.xxx, c.req.query(...), Zod / Joi schema keys                      |
    | Request body       | req.body, Zod z.object, class-validator DTO, Joi schema                     |
    | Response type      | res.json(...), res.status(N).json(...), type annotations, JSDoc @returns    |
    | Error responses    | throw new HttpException, res.status(4xx/5xx), next(err), custom error class |
    | Auth / authz       | Middleware names: authenticate, authorize, guard, jwtMiddleware, etc.       |
    | Description        | JSDoc @description, @summary, inline comments                               |

    Additionally, for each endpoint collect `sourceFiles`: the set of files that define
    the types actually used by that endpoint. Follow every import that resolves to a
    type-bearing file and record its path if it falls into one of these categories:

    | Category          | Detection heuristic                                                              |
    |-------------------|----------------------------------------------------------------------------------|
    | Request body DTO  | Imported class/interface/Zod schema passed to req.body, @Body(), z.parse(), etc. |
    | Response DTO      | Return type annotation, JSDoc @returns type, interface/class in res.json(...)    |
    | Query param schema| Zod/Joi/class-validator schema used to validate req.query or c.req.query(...)    |
    | Path param schema | Zod/Joi/class-validator schema used to validate req.params or route-level types  |
    | Shared type       | Re-exported type that the above types extend or compose                          |

    Rules for path recording:
    - Record the path relative to TARGET_ROOT (e.g. src/dto/create-user.dto.ts).
    - If the type is defined inline in the handler file itself, record the handler file path.
    - If the type originates from a node_modules package, skip it — external types are not linked.
    - Deduplicate: if request body and response share the same file, list it once.
    - If no qualifying file can be found for a given category, omit that category entry.

    Also:
    - Grep for common error format: errorHandler | AppError | HttpException | ApiError | createError
      Read discovered files and identify the common error response shape.
    - Grep for common response wrapper: ApiResponse | ResponseWrapper | successResponse | wrapResponse
      If found, read the file and note the wrapper structure.

    Return a structured JSON array where each element represents one endpoint with all fields above,
    plus a `sourceFiles` array structured as:
    [
      { "category": "requestBody", "label": "CreateUserDto", "path": "src/dto/create-user.dto.ts" },
      { "category": "response",    "label": "UserResponse",  "path": "src/dto/user.response.ts"   },
      { "category": "querySchema", "label": "UserQueryDto",  "path": "src/dto/user-query.dto.ts"  }
    ]
    For any field that cannot be determined, use the string "undetermined".
    For endpoints where type information is entirely insufficient, add "undocumented": true.
  """
)
```

Store the returned JSON as `ENDPOINT_DATA`.

Mark `p3` complete:
```
todowrite([{ id: "p3", status: "completed" }, { id: "p4", status: "in_progress" }])
```

---

## PHASE 4: Organize document structure

> **Agent**: Atlas (self) — synthesis and grouping
> **Optional consultation**: `@oracle` for ambiguous structural decisions

Group endpoints into sections by domain / resource, derived from router file paths or directory structure.

- Example: `src/routes/users/` → `## Users`, `src/routes/auth/` → `## Auth`
- Within each section, order by: `GET` → `POST` → `PUT/PATCH` → `DELETE`.

If grouping logic is ambiguous (e.g. flat router structure with no clear domain separation),
consult `@oracle`:

```
task(
  subagent_type = "oracle",
  prompt = """
    Given these endpoints: {ENDPOINT_DATA_SUMMARY}
    Propose the most logical grouping into documentation sections.
    Consider route prefixes, handler file locations, and naming conventions.
    Return a JSON mapping: { sectionName: [endpointIds] }
  """
)
```

### Parameter notation rules

| Kind | Notation |
|------|----------|
| Required | `name: string` |
| Optional | `name?: string` |
| Union | `status: "active" \| "inactive"` |
| Array | `ids: string[]` |
| Type undetermined | `any (undetermined)` |

Mark `p4` complete:
```
todowrite([{ id: "p4", status: "completed" }, { id: "p5", status: "in_progress" }])
```

---

## PHASE 5: Write ENDPOINT.md

> **Agent**: Atlas (self) — direct file write

If no endpoints were found or scan results are insufficient, report to the user and abort — do not write any file.

Write `{TARGET_ROOT}/ENDPOINT.md` using the template below.
If a file already exists at that path, overwrite it silently.

---

````markdown
# ENDPOINT.md

> Auto-generated by `/gen-endpoint-docs`
> Source: `{TARGET_ROOT}/src`
> Generated: {YYYY-MM-DD}
>
> ⚠️ This file is derived from source code. If anything conflicts with the implementation, source takes priority. Re-run `/gen-endpoint-docs` to refresh.

---

## Overview

| Field | Value |
|-------|-------|
| Base URL | `{BASE_URL or "undetermined — check environment config"}` |
| Authentication | `{Bearer JWT \| API Key \| None \| undetermined}` |
| Content-Type | `application/json` |
| Response wrapper | `{present (see Common Response below) \| none}` |

---

## Common Response Structure

### Success

```json
{
  // Describe wrapper shape here if one exists.
  // If no wrapper: "See each endpoint for its individual response shape."
}
```

### Error

```json
{
  "status": 400,
  "code": "VALIDATION_ERROR",
  "message": "Human-readable description"
}
```

> If no common error format was found in source:
> "Common error format undetermined — see individual endpoint error tables."

---

## {Domain / Resource Name}

---

### `{METHOD}` `{/path/:param}`

> {One-line description — from JSDoc `@description`, `@summary`, or inferred from the handler name}

**Auth required**: `{yes (Bearer JWT) | no | undetermined}`
**Permission**: `{admin | owner | any | undetermined}`

#### Path Parameters

| Name | Type | Required | Description |
|------|------|----------|-------------|
| `param` | `string` | yes | {description} |

> Omit this section if the endpoint has no path parameters.

#### Query Parameters

| Name | Type | Required | Default | Description |
|------|------|----------|---------|-------------|
| `page` | `number` | no | `1` | Page number |

> Omit this section if the endpoint has no query parameters.

#### Request Body

```typescript
{
  field: string           // required
  optionalField?: number  // optional
}
```

> Omit this section for endpoints that have no request body (GET, DELETE, etc.).

#### Response `200`

```typescript
{
  id: string
  name: string
  createdAt: string  // ISO 8601
}
```

#### Errors

| Status | Code | Cause |
|--------|------|-------|
| `400` | `VALIDATION_ERROR` | Missing required field or invalid format |
| `401` | `UNAUTHORIZED` | Missing or expired token |
| `404` | `NOT_FOUND` | Resource does not exist |
| `500` | `INTERNAL_ERROR` | Unexpected server error |

#### Source References

| Category | File |
|----------|------|
| Request body | [`CreateUserDto`](src/dto/create-user.dto.ts) |
| Response | [`UserResponse`](src/dto/user.response.ts) |
| Query schema | [`UserQueryDto`](src/dto/user-query.dto.ts) |

> List only files that exist in this repository (no node_modules).
> If request body, response, and query schema all resolve to the same file, list it once with the label `Types`.
> Omit this section entirely if no qualifying type files were found.

---

### `{METHOD}` `{/path}`

{Repeat the structure above}

---

## {Domain / Resource Name 2}

{Repeat the structure above}

---

## Undocumented Endpoints

> Endpoints where type information was insufficient for full documentation.
> Inspect the source directly for these.

| Method | Path | Location | Reason |
|--------|------|----------|--------|
| `POST` | `/example` | `src/routes/example.ts:42` | Body type is `any`, cannot infer |

> Omit this section if all endpoints were fully documented.
````

Mark `p5` complete:
```
todowrite([{ id: "p5", status: "completed" }, { id: "p6", status: "in_progress" }])
```

---

## PHASE 6: Update AGENTS.md

> **Agent**: Atlas (self) — minimal targeted insertion

After `ENDPOINT.md` is written, register it in any `AGENTS.md` files that exist.
This allows every agent operating in that project (or monorepo) to know the file exists and consult it when working on API-related tasks.

### 6-1. Project-level AGENTS.md

```bash
!ls {TARGET_ROOT}/AGENTS.md 2>/dev/null || echo "NOT_FOUND"
```

If found, read the file and locate the most appropriate insertion point — preferably a section that lists project files, references, or documentation artifacts (e.g. `## Project Files`, `## References`, `## Documentation`, or similar). If no such section exists, append to the end of the file.

Insert the following block. Do not duplicate it if an `ENDPOINT.md` entry already exists.

```markdown
### ENDPOINT.md

Documents every HTTP endpoint exposed by this service.
Generated by `/gen-endpoint-docs` — re-run the command after route changes.
Consumers of this API should read `ENDPOINT.md` instead of scanning the source.
```

Edit the file with a minimal, targeted insertion — do not rewrite or reformat any existing content.

### 6-2. Monorepo root AGENTS.md

This step runs only when `TARGET_ROOT` is a subdirectory of a monorepo (i.e. `TARGET_ROOT` is not the repository root).

Locate the monorepo root:

```bash
!git rev-parse --show-toplevel
```

```bash
!ls {MONOREPO_ROOT}/AGENTS.md 2>/dev/null || echo "NOT_FOUND"
```

If found, read the file and locate a section that references individual packages or apps (e.g. `## Packages`, `## Apps`, `## Monorepo Structure`, or a table of packages). Insert or append a reference to the generated file under the entry for `TARGET_ROOT`. If no per-package section exists, append to the end of the file.

Insert the following block. Do not duplicate it if an entry for this package's `ENDPOINT.md` already exists.

```markdown
- `{TARGET_ROOT}/ENDPOINT.md` — HTTP endpoint reference for `{package name}`. Read before working on API integration tasks involving this package.
```

Edit with a minimal insertion — do not rewrite or reformat any existing content.

### 6-3. AGENTS.md not found

If neither `AGENTS.md` file exists, skip this phase silently.
Do not create an `AGENTS.md` file — that is outside the scope of this command.

Mark `p6` complete:
```
todowrite([{ id: "p6", status: "completed" }, { id: "p7", status: "in_progress" }])
```

---

## PHASE 7: Report

> **Agent**: Atlas (self) — final summary

```
ENDPOINT.md written : {TARGET_ROOT}/ENDPOINT.md
  Sections          : {N} domain(s)
  Endpoints         : {N} documented / {N} undocumented
  Aux refs          : {used (.y/contract/{filename}) | none}

AGENTS.md updated   : {TARGET_ROOT}/AGENTS.md          {updated | not found, skipped}
                      {MONOREPO_ROOT}/AGENTS.md         {updated | not found, skipped | n/a (not a monorepo)}

Agents used:
  @explore    — directory listing, router entry point discovery
  @librarian  — import chain traversal, schema/type extraction
  @oracle     — (if invoked) document section grouping advice

{List any endpoints or fields where types could not be determined}

Next steps:
  - Add JSDoc to undocumented handlers, then re-run /gen-endpoint-docs.
  - Re-run /gen-endpoint-docs after source changes to keep the document current.
```

Mark `p7` complete:
```
todowrite([{ id: "p7", status: "completed" }])
```

---

## Agent Responsibility Summary

| Phase | Task | Agent |
|-------|------|-------|
| 0 | Resolve TARGET_ROOT, prompt user (question tool) | Atlas |
| 1 | Verify Express-family dependency | `@explore` |
| 2 | Read `.y/contract` and `.y/plans` | `@librarian` |
| 3-1 | File listing, router entry point grep | `@explore` |
| 3-2 | Import chain traversal, full schema extraction | `@librarian` |
| 4 | Group endpoints into sections; ambiguous cases → consult | `@oracle` (optional) |
| 5 | Write ENDPOINT.md | Atlas |
| 6 | Update AGENTS.md (minimal insertion) | Atlas |
| 7 | Final report | Atlas |

---

## Execution Rules

| Rule | Detail |
|------|--------|
| **Todo-first** | Call `todowrite` at the start of PHASE 0 to register all phases. Mark each phase complete before proceeding to the next. |
| **Source is authoritative** | `.y/contract` and `.y/plans` are supplementary. Source wins on any conflict. |
| **Make gaps explicit** | Undetermined types or missing descriptions are marked `undetermined`, never silently omitted. |
| **Output location** | `ENDPOINT.md` is written to `TARGET_ROOT/` only — never to the monorepo root. |
| **Overwrite is safe** | An existing `ENDPOINT.md` is overwritten without warning. |
| **Abort conditions** | No Express-family dependency found, or zero route files discovered. Mark all remaining todo items `cancelled` before aborting. |
| **AGENTS.md: minimal touch** | Insert only the required block — do not rewrite or reformat any existing content. |
| **AGENTS.md: no creation** | Never create an `AGENTS.md` file. Only update files that already exist. |
| **AGENTS.md: no duplication** | Skip insertion if an `ENDPOINT.md` entry is already present in the file. |
| **Subagent results are inputs** | Always wait for each subagent task to complete before using its output in the next phase. |
