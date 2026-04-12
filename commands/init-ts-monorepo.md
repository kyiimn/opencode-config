---
description: Initialize a TypeScript pnpm monorepo. generate AGENTS.md / RULES.md → scaffold workspace → git init → verify.
---

Initialize a TypeScript pnpm monorepo: generate AGENTS.md / RULES.md → scaffold workspace → git init → verify.

---

## TODO MANAGEMENT — Required

커멘드 시작 직후 `todowrite`를 호출하여 전체 페이즈를 등록하라. 각 페이즈 진입 시 `in_progress`, 완료 시 `completed`, 조건부 스킵 시 `cancelled`로 업데이트하라.

**`todowrite`는 전체 목록을 교체하므로, 호출 시 항상 전체 항목을 포함해야 한다.**

### 초기 `todowrite` 호출 (커멘드 시작 즉시)

```json
[
  { "id": "p0", "content": "PHASE 0: Check environment (abort condition)",        "priority": "high",   "status": "pending" },
  { "id": "p1", "content": "PHASE 1: Collect conventions (naming/style)",         "priority": "high",   "status": "pending" },
  { "id": "p2", "content": "PHASE 2: Write AGENTS.md and RULES.md",               "priority": "high",   "status": "pending" },
  { "id": "p3", "content": "PHASE 3: Scaffold workspace files (24 files)",        "priority": "high",   "status": "pending" },
  { "id": "p4", "content": "PHASE 4: git init",                                   "priority": "medium", "status": "pending" },
  { "id": "p5", "content": "PHASE 5: Verify (install/typecheck/lint/test)",       "priority": "high",   "status": "pending" },
  { "id": "p6", "content": "PHASE 6: Record troubleshooting",                     "priority": "medium", "status": "pending" },
  { "id": "p7", "content": "PHASE 7: Initial Commit (user approval required)",    "priority": "medium", "status": "pending" },
  { "id": "p8", "content": "PHASE 8: Report",                                     "priority": "low",    "status": "pending" }
]
```

---

## PHASE 0: Check environment

> ▶ `todowrite`: `p0` → `in_progress`

Check whether `.git/` exists (used in PHASE 4 to decide if `git init` is needed).

**Abort condition:** If `apps/` or `packages/` already contain subdirectories (other than `packages/core`), stop immediately and report:

```
ERROR: Non-empty workspace detected.
  Found existing packages in apps/ or packages/.
  init-ts-monorepo only initializes greenfield monorepos.
  Use init-ts-api / init-ts-cli / init-ts-react to add packages to an existing monorepo.
```

---

> ✅ `todowrite`: `p0` → `completed`

## PHASE 1: Collect conventions

> ▶ `todowrite`: `p1` → `in_progress`

Use `ask_user_input_v0` to collect the following. Project name = current directory name (e.g. `my-app` → scope `@my-app`).

```
[Naming]
Q-N1. File naming:   kebab-case | camelCase | PascalCase | snake_case
Q-N2. Class naming:  PascalCase | camelCase
Q-N3. Interface:     PascalCase (no prefix) | I-prefix + PascalCase
Q-N4. Functions:     camelCase + verb-prefix enforced | camelCase (verb recommended)

[TypeScript style]
Q-T1. Type definitions:  interface-first | type-first | mixed
Q-T2. Function style:    arrow-first | function-first | mixed (function at top-level, arrow in class/callbacks)
```

---

> ✅ `todowrite`: `p1` → `completed`

## PHASE 2: Write AGENTS.md and RULES.md

> ▶ `todowrite`: `p2` → `in_progress`

Fill `{…}` placeholders with PHASE 1 values. Use `write_file` — no shell redirects (`echo`, heredoc).

### 2-1. `./AGENTS.md`

````markdown
# {PROJECT_NAME} — AGENTS.md

**Stack:** TypeScript (strict) · pnpm workspaces
**Package Manager:** pnpm
**Rules:** [`RULES.md`](./RULES.md)

---

## PROJECT STRUCTURE

```
{project-name}/
├── apps/                    # Deployable apps (each has its own AGENTS.md)
├── packages/
│   ├── core/                # Shared configs and libraries
│   │   ├── tsconfig/        # Base tsconfig variants (base, app, react-lib)
│   │   ├── eslint/          # ESLint flat config variants (base, react, node)
│   │   └── src/             # Shared types/utils entry point
│   └── dto/                 # Shared DTOs, Zod schemas, and API type definitions (single source of truth)
│       └── src/             # DTO interfaces, Zod schemas, request/response types, enums
├── AGENTS.md
├── RULES.md
├── pnpm-workspace.yaml
├── package.json
└── tsconfig.json
```

Import via package name. Never use relative paths across package boundaries.

```jsonc
// Each package's tsconfig.json
{ "extends": "@{scope}/core/tsconfig/app.json" }
```

```javascript
// Each package's eslint.config.js
import base from '@{scope}/core/eslint/node.js';

export default [
  ...base,
  {
    // projectService must point to THIS package's tsconfig.json.
    files: ["**/*.ts"],
    languageOptions: {
      parserOptions: {
        projectService: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
  },
  {
    // Config files at package root (vitest.config.ts, vite.config.ts, etc.)
    // are outside tsconfig include — exclude them to prevent projectService errors.
    ignores: ["dist/", "eslint.config.js", "vitest.config.ts", "vite.config.ts"],
  },
];
```

---

## COMMANDS

| Task          | Command                    | Where                 |
| ------------- | -------------------------- | --------------------- |
| Type check    | `pnpm tsc --noEmit`        | package dir           |
| Test (single) | `pnpm vitest run {path}`   | package dir           |
| Test (all)    | `pnpm vitest run`          | root                  |
| Lint          | `pnpm eslint --fix {path}` | root                  |
| Build         | `pnpm build`               | explicit request only |

Type-check and test: run autonomously after implementation. Build/deploy: require user approval.

---

## SAFETY & PERMISSIONS

**Autonomous:** read files, `tsc --noEmit`, `eslint`, `prettier`, single test file.

**Require approval:** `pnpm add/remove`, `git push`, branch/PR creation, file/directory deletion, full build/deploy.

---

## GIT CONVENTIONS

- Branch: `feature/{task}` · `fix/{issue}` · `chore/{task}`
- Commits: Conventional Commits (`feat:` `fix:` `chore:` `docs:` `refactor:` `test:`)
- Pass `tsc --noEmit` and lint before PR.

---

## FILE REFERENCES

Load `RULES.md` before running `/start-work`. Never start without it.

```
@AGENTS.md   # this file
@RULES.md    # coding rules (required)
```

Load only the relevant section anchor:

| When                 | Section                                  |
| -------------------- | ---------------------------------------- |
| Writing/editing code | `RULES.md#code-standards`                |
| Using the logger     | `RULES.md#logger`                        |
| DTO / API types      | `RULES.md#dto`                           |
| Implementing API     | `RULES.md#contract-compliance`           |
| API structure/layers | package `ARCHITECTURE.md`                |
| Frontend structure   | package `ARCHITECTURE.md`                |
| CLI structure        | package `ARCHITECTURE.md`                |
| DB schema            | `prisma/schema.prisma`                   |
| Env vars             | `.env.example`                           |
````

### 2-2. `./RULES.md`

Fill `{FILE_NAMING}`, `{CLASS_NAMING}`, `{INTERFACE_NAMING}`, `{METHOD_NAMING}`, `{TYPE_DEFINITION_STYLE}`, `{FUNCTION_STYLE}` with PHASE 1 values.

```markdown
# {PROJECT_NAME} — RULES.md

> Load on demand. Use section anchors — do not read the entire file at once.

- [`#code-standards`](#code-standards) — TypeScript · naming · bundling · Do/Don't
- [`#logger`](#logger) — Logger import paths · level rules · Do/Don't
- [`#bigint-json`](#bigint-json) — BigInt-safe JSON · Express middleware · fetch wrapper
- [`#dto`](#dto) — DTO & API type definitions · centralized in packages/dto
- [`#contract-compliance`](#contract-compliance) — API contract verification during implementation

---

## CODE STANDARDS {#code-standards}

### TypeScript

- `strict: true` required
- No `any` — use `unknown` + type guard
- Explicit return types (`noImplicitReturns: true`)
- Minimize non-null assertion (`!`) — prefer guards or optional chaining

### Naming

| Target          | Rule                                        | Example                      |
| --------------- | ------------------------------------------- | ---------------------------- |
| File            | {FILE_NAMING}                               | `{FILE_NAMING_EXAMPLE}`      |
| Class           | {CLASS_NAMING}                              | `{CLASS_NAMING_EXAMPLE}`     |
| Interface       | {INTERFACE_NAMING}                          | `{INTERFACE_NAMING_EXAMPLE}` |
| Type alias      | PascalCase                                  | `UserDto`, `ApiResponse<T>`  |
| Function/method | {METHOD_NAMING}                             | `{METHOD_NAMING_EXAMPLE}`    |
| Constant        | SCREAMING_SNAKE_CASE                        | `MAX_RETRY_COUNT`            |
| Enum            | PascalCase (name) / SCREAMING_SNAKE (value) | `Status.ACTIVE`              |

### Type definition style

{TYPE_DEFINITION_STYLE}

### Function declaration style

{FUNCTION_STYLE}

### Bundling / execution

- Bundle with Vite (`vite build`)
- Run TS directly with `vite-node`
- `tsc` for type-check only (`--noEmit`) — never use as build tool
- Forbidden: esbuild, webpack, ts-node, tsx

### Do

- Functional, single responsibility
- `async/await` — no callbacks
- Always throw errors to upper layers
- Narrow types (union, literal types)
- Use conditional spread when building objects with optional properties:
  `{ ...value !== undefined && { key: value } }` — required by `exactOptionalPropertyTypes`

### Don't

- No `console.log` in production — use a logger
- No magic numbers/strings — extract as constants
- No `// TODO` without a commit
- No new packages without user approval
- No `any`
- Do not assign `undefined` explicitly to optional properties — use conditional spread instead

---

## LOGGER {#logger}

All packages in this monorepo share the logger from `@{scope}/core`.
No source scan is needed — import paths and level rules are fixed.

### Import

```typescript
// Named logger — recommended for any class or module
import { createLogger } from '@{scope}/core';
const logger = createLogger('FeatureName');

// Default logger — for one-off scripts or simple entry points
import { logger } from '@{scope}/core';
```

### Levels

| Level | When to use |
|-------|-------------|
| `debug` | Internal state, query params, tracing — **suppressed in production** |
| `log` | Normal lifecycle events (server start, job complete, request handled) |
| `warn` | Unexpected but recoverable situation (retry, fallback, deprecation) |
| `error` | Exception, external service failure, unrecoverable error |

### Usage

```typescript
logger.debug('Query params', { filter, page });   // dev only
logger.log('User created', { id: user.id });
logger.warn('Retry attempt', { attempt, maxRetries });
logger.error('Payment failed', error);
```

### Do / Don't

- **Do** call `createLogger('ContextName')` once at the top of each file
- **Do** pass structured data as the second argument instead of string interpolation
- **Don't** use `console.log`, `console.warn`, or `console.error` anywhere in application code
- **Don't** log sensitive data (passwords, tokens, PII) at any level
---

## DTO & API TYPES {#dto}

> **Single source of truth.** All DTOs, Zod schemas, and API type definitions live exclusively in `packages/dto`.
> No package may define its own DTO/request/response type for cross-package communication.

### Ownership rule

| Scenario | Where to define | Where to import from |
|----------|-----------------|----------------------|
| API request body (schema + type) | `packages/dto/src` | `@{scope}/dto` |
| API response shape (schema + type) | `packages/dto/src` | `@{scope}/dto` |
| Shared enum (status, role, …) | `packages/dto/src` | `@{scope}/dto` |
| Validation function | `packages/dto/src` | `@{scope}/dto` |
| Internal domain type (not exposed via API) | inside the package | same package only |

### Directory layout

```
packages/dto/
└── src/
    ├── index.ts                       # Re-exports everything — the only public surface
    └── {domain}/
        ├── {domain}.schema.ts         # Zod schemas → infer TypeScript types from here
        ├── {domain}.response.ts       # Response types (no Zod needed — output only)
        └── {domain}.enum.ts           # Shared enums
```

### Type definition style

**Types are always inferred from Zod schemas** — never declare a type independently alongside a schema.

```typescript
// packages/dto/src/user/user.schema.ts
import { z } from 'zod';

export const CreateUserSchema = z.object({
  email: z.string().email(),
  name:  z.string().min(1).max(100),
  role:  z.enum(['ADMIN', 'MEMBER']),
});

export const UpdateUserSchema = CreateUserSchema.partial().extend({
  id: z.string().uuid(),
});

// Types are inferred — never written by hand
export type CreateUserRequest = z.infer<typeof CreateUserSchema>;
export type UpdateUserRequest = z.infer<typeof UpdateUserSchema>;
```

```typescript
// packages/dto/src/user/user.response.ts
// Response types have no Zod schema — they are pure output shapes.
export interface UserResponse {
  id:        string;
  email:     string;
  name:      string;
  role:      string;
  createdAt: string; // ISO 8601
}

export interface UserListResponse {
  items: UserResponse[];
  total: number;
}
```

### Validation usage

**API server — router / middleware**

```typescript
import { CreateUserSchema } from '@{scope}/dto';

router.post('/users', (req, res, next) => {
  const result = CreateUserSchema.safeParse(req.body);
  if (!result.success) {
    res.status(400).json({ errors: result.error.flatten() });
    return;
  }
  next(); // result.data is typed as CreateUserRequest
});
```

**Client — Frontend / CLI (pre-flight validation)**

```typescript
import { CreateUserSchema, type CreateUserRequest } from '@{scope}/dto';

function validateBeforeSend(payload: unknown): CreateUserRequest {
  return CreateUserSchema.parse(payload); // throws ZodError on failure
}

// Or non-throwing:
const result = CreateUserSchema.safeParse(payload);
if (!result.success) {
  console.error(result.error.flatten());
}
```

### Do / Don't

- **Do** define types via `z.infer<typeof Schema>` — never write a duplicate `interface` alongside a schema
- **Do** add new schemas in `packages/dto/src/{domain}/{domain}.schema.ts` and re-export from `src/index.ts`
- **Do** keep `packages/dto` framework-agnostic — no Express, Prisma, or React imports
- **Do** use `.response.ts` (plain `interface`) for response shapes — response validation is the server's responsibility
- **Don't** duplicate a DTO or schema locally inside `apps/*` or other `packages/*`
- **Don't** import Prisma model types in responses — map to a response interface first
- **Don't** use `class` for DTOs
- **Don't** use `z.parse()` in routers where a 400 response is expected — use `.safeParse()` and handle the error explicitly

---

## BIGINT JSON {#bigint-json}

> Apply when any Prisma column or API field uses the `BigInt` type.
> Run the `ts-bigint-json` skill to generate the implementation files.

`JSON.stringify` throws `TypeError` on `bigint` values; `JSON.parse` never produces `bigint`.
All BigInt ↔ JSON serialization must go through `bigintJson` — never use native `JSON.*` directly.

### Wire format

`{ "__bigint__": "9007199254740993" }` — tagged object, lossless, portable to any JSON client.

### Import

```typescript
import { bigintJson } from '@{scope}/core';
```

### Stringify / Parse

```typescript
const body = bigintJson.stringify({ id: 9007199254740993n, name: 'Alice' });
// → '{"id":{"__bigint__":"9007199254740993"},"name":"Alice"}'

const data = bigintJson.parse<{ id: bigint; name: string }>(body);
// → { id: 9007199254740993n, name: 'Alice' }
```

### API Server — Express

Replace `express.json()` with `bigintJsonMiddleware()` in `app.ts`:

```typescript
import { bigintJsonMiddleware } from '@/common/infrastructure/http/bigint-json.middleware';

// replaces: app.use(express.json())
app.use(bigintJsonMiddleware());
```

### Client — Frontend / CLI

```typescript
import { bigintFetch } from '@/lib/bigint-fetch';
import { bigintJson } from '@{scope}/core';

// GET — response body deserialized with bigintJson.parse
const user = await bigintFetch<User>('/api/users/1');

// POST — body serialized with bigintJson.stringify
const order = await bigintFetch<Order>('/api/orders', {
  method: 'POST',
  body: bigintJson.stringify({ productId: 1n, quantity: 2n }),
});
```

### Do / Don't

- **Do** use `bigintJson.stringify` / `.parse` for any payload that may contain `bigint`
- **Do** use `bigintJsonMiddleware()` in the Express app instead of `express.json()`
- **Do** use `bigintFetch()` on the client side for endpoints that return BigInt fields
- **Don't** use native `JSON.stringify` / `JSON.parse` for API payloads containing BigInt
- **Don't** cast `bigint` to `Number` — values > 2^53 − 1 lose precision silently
- **Don't** use `.toString()` inline as a workaround — it breaks downstream type contracts

---

## CONTRACT COMPLIANCE {#contract-compliance}

> Apply when `.sisyphus/contract/{keyword}.md` exists for the current task.
> This section is enforced during `/start-work` execution.

Before marking any endpoint implementation as complete, the implementing agent MUST:

1. Read `$REPO_ROOT/.sisyphus/contract/{keyword}.md`
2. Compare the actual response shape (success body, status code, error codes) against the contract
3. If any mismatch exists, fix the code before proceeding to the next task

### Checklist per endpoint

| Check | What to verify |
|-------|---------------|
| Success status code | Matches contract exactly (e.g. `201` not `200`) |
| Success body shape | All field names, types, and nesting match |
| Error status codes | Every error case in the contract is handled |
| Error codes | String error codes match verbatim (e.g. `ALREADY_REFUNDED`) |
| Envelope format | `{ ok, data }` / `{ ok, message, code }` matches project convention |

### Do / Don't

- **Do** read the contract file at the start of each endpoint implementation
- **Do** fix mismatches immediately — do not defer to test phase
- **Don't** invent response fields or error codes not in the contract
- **Don't** skip this check even if the plan's `### Expected Contract` subsection is present — always verify against the contract file as the single source of truth
```

### 2-3. Save paths

```
./AGENTS.md
./RULES.md
```

Report saved paths to the user.

> ✅ `todowrite`: `p2` → `completed`

---

## PHASE 3: Scaffold workspace files

> ▶ `todowrite`: `p3` → `in_progress`

Use current directory name as project name and scope. Create files in order using `write_file`.

**1. `package.json`**

```jsonc
{
  "name": "{project-name}",
  "private": true,
  "type": "module",
  "scripts": {
    "build": "pnpm -r build",
    "dev": "pnpm -r dev",
    "test": "pnpm -r test",
    "lint": "pnpm -r lint",
    "typecheck": "pnpm -r typecheck",
  },
  "devDependencies": {
    "typescript": "^5",
    "vite": "^6",
    "vitest": "^3",
    "@vitest/coverage-v8": "^3",
    "eslint": "^9",
    "typescript-eslint": "^8",
    "@types/node": "^22",
  },
}
```

**2. `pnpm-workspace.yaml`**

```yaml
packages:
  - "apps/*"
  - "packages/*"
```

**3. `tsconfig.json`**

```jsonc
{
  "extends": "./packages/core/tsconfig/base.json",
  "include": [],
  "exclude": ["node_modules"],
}
```

**4. `.npmrc`**

```
shamefully-hoist=false
strict-peer-dependencies=false
auto-install-peers=true
```

**5. `packages/core/package.json`**

```jsonc
{
  "name": "@{scope}/core",
  "version": "0.0.1",
  "private": true,
  "type": "module",
  "main": "./src/index.ts",
  "scripts": {
    "typecheck": "tsc --noEmit",
    "lint": "eslint --fix .",
    "test": "vitest run",
  },
}
```

**6. `packages/core/tsconfig/base.json`**

```jsonc
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,
    "noEmit": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
  },
}
```

**7. `packages/core/tsconfig/app.json`**

```jsonc
{
  "extends": "./base.json",
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "Bundler",
  },
}
```

**8. `packages/core/tsconfig/react-lib.json`**

```jsonc
{
  "extends": "./base.json",
  "compilerOptions": {
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "jsx": "react-jsx",
  },
}
```

**9. `packages/core/eslint/index.js`**

```javascript
import tseslint from "typescript-eslint";

// Type-checked rules applied only to TypeScript source files.
// Each consuming package must add parserOptions with projectService
// in its own eslint.config.js (see packages/core/eslint.config.js for reference).
const tsFiles = tseslint.config({
  files: ["**/*.ts", "**/*.tsx"],
  extends: [...tseslint.configs.strictTypeChecked],
  rules: {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/explicit-function-return-type": "warn",
  },
});

// Non-type-checked rules for JS config files (eslint.config.js, vite.config.js, etc.).
// strictTypeChecked must NOT apply here — those rules require type information.
const jsFiles = tseslint.config({
  files: ["**/*.{js,mjs,cjs}"],
  extends: [...tseslint.configs.strict],
  rules: {
    "@typescript-eslint/no-explicit-any": "error",
  },
});

export default [...tsFiles, ...jsFiles];
```

**10. `packages/core/eslint/react.js`**

```javascript
import base from "./index.js";

// Extend with React-specific rules here.
export default [...base];
```

**11. `packages/core/eslint/node.js`**

```javascript
import base from "./index.js";

// Extend with Node.js-specific rules here.
export default [...base];
```

**12. `packages/core/tsconfig.json`**

> Required for `tsc --noEmit` and ESLint `projectService` in the core package itself.

```jsonc
{
  "extends": "./tsconfig/base.json",
  "include": ["src/**/*.ts"]
}
```

**13. `packages/core/eslint.config.js`**

> The core package cannot self-reference `@{scope}/core`, so relative paths must be used here.

```javascript
import nodeConfig from "./eslint/node.js";

export default [
  ...nodeConfig,
  {
    // parserOptions must be set per-package so projectService resolves
    // the correct tsconfig.json relative to THIS package root.
    files: ["**/*.ts"],
    languageOptions: {
      parserOptions: {
        projectService: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
  },
  {
    // Exclude root-level TS config files from projectService parsing.
    // These files are outside tsconfig.json's `include` and will cause
    // "parserOptions.project" errors if linted with type-checked rules.
    ignores: ["dist/", "eslint.config.js", "vitest.config.ts", "vite.config.ts"],
  },
];
```

**14. `packages/core/vitest.config.ts`**

```typescript
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    globals: false,
    coverage: {
      provider: "v8",
      reporter: ["text", "lcov"],
      thresholds: { lines: 80, functions: 80, branches: 80 },
    },
  },
});
```

**15. `packages/core/src/index.ts`**

```typescript
// Export shared types and utilities from this entry point only.
// Never import across package boundaries via relative paths.
export * from './lib/logger.js';
```

> `packages/core/src/lib/logger.ts` is generated by the `ts-logger` skill.
> Run the `ts-logger` skill in monorepo mode immediately after PHASE 3 file creation.

**16. `packages/core/src/index.test.ts`**

```typescript
import { describe, it, expect } from "vitest";

describe("packages/core smoke test", () => {
  it("workspace is initialized", () => {
    expect(true).toBe(true);
  });
});
```

**17. `packages/dto/package.json`**

```jsonc
{
  "name": "@{scope}/dto",
  "version": "0.0.1",
  "private": true,
  "type": "module",
  "main": "./src/index.ts",
  "scripts": {
    "typecheck": "tsc --noEmit",
    "lint": "eslint --fix .",
    "test": "vitest run",
  },
  "dependencies": {
    "zod": "^3",
  },
}
```

**18. `packages/dto/tsconfig.json`**

```jsonc
{
  "extends": "../core/tsconfig/base.json",
  "include": ["src/**/*.ts"]
}
```

**19. `packages/dto/eslint.config.js`**

```javascript
import nodeConfig from "../core/eslint/node.js";

export default [
  ...nodeConfig,
  {
    files: ["**/*.ts"],
    languageOptions: {
      parserOptions: {
        projectService: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
  },
  {
    ignores: ["dist/", "eslint.config.js"],
  },
];
```

**20. `packages/dto/src/index.ts`**

```typescript
// Central re-export for all DTOs, Zod schemas, and API type definitions.
// Add new domain exports here as packages/dto grows.
// Each domain module re-exports: schema, inferred request types, response interfaces, enums.
//
// Example:
//   export * from './user/user.schema.js';   // CreateUserSchema, UpdateUserSchema, CreateUserRequest, …
//   export * from './user/user.response.js'; // UserResponse, UserListResponse
//   export * from './user/user.enum.js';     // UserRole, UserStatus
```

**21. `packages/dto/src/index.test.ts`**

```typescript
import { describe, it, expect } from "vitest";

describe("packages/dto smoke test", () => {
  it("workspace is initialized", () => {
    expect(true).toBe(true);
  });
});
```

**22. `packages/dto/vitest.config.ts`**

```typescript
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    globals: false,
    coverage: {
      provider: "v8",
      reporter: ["text", "lcov"],
      thresholds: { lines: 80, functions: 80, branches: 80 },
    },
  },
});
```

**23. `apps/.gitkeep`** — empty file to track directory

**24. `.gitignore`** — skip if already exists

```
node_modules/
dist/
.env
.env.local
*.tsbuildinfo
coverage/
```

> ✅ `todowrite`: `p3` → `completed`

---

## PHASE 4: git init

> ▶ `todowrite`: `p4` → `in_progress` (no .git) or `cancelled` (already exists)

Skip if `.git/` already exists. Otherwise:

```bash
git init
git branch -M main
```

If `git user.name` / `user.email` is not set, notify user and record the warning.
The initial commit will be requested in **PHASE 7: Initial Commit** after verification and troubleshooting.

> ✅ `todowrite`: `p4` → `completed`

---

## PHASE 5: Verify

> ▶ `todowrite`: `p5` → `in_progress`

Run in order. Do not report completion without running this.

```bash
pnpm install
pnpm --filter @{scope}/core tsc --noEmit
pnpm --filter @{scope}/core lint
pnpm --filter @{scope}/core test
pnpm --filter @{scope}/dto tsc --noEmit
pnpm --filter @{scope}/dto lint
pnpm --filter @{scope}/dto test
```

Report immediately if any command is missing or fails.

> ✅ `todowrite`: `p5` → `completed`

---

## PHASE 6: Record troubleshooting

> ▶ `todowrite`: `p6` → `in_progress`

Run `/gen-trouble-shoot --source init --label init-ts-monorepo`

> `/gen-trouble-shoot` will scan this session, extract AI mistakes and incorrect
> implementations that occurred during scaffolding, and write them to `TROUBLE_SHOOT.md`.
> No action is needed here beyond invoking the command.

> ✅ `todowrite`: `p6` → `completed`

---

## PHASE 7: Initial Commit ⚠️

> ▶ `todowrite`: `p7` → `in_progress`

If `.git/` does not exist in the project root, skip this phase entirely.

If `git user.name` / `user.email` was not configured, remind the user and skip.

Present the following via `ask_user_input_v0`:

```
Verification and troubleshooting recording are complete.
Proceed with the initial git commit?

  Commit message:
  "chore: initialize pnpm monorepo workspace

  - Add pnpm-workspace.yaml with apps/* and packages/* globs
  - Add packages/core with tsconfig, eslint, vitest shared configs
  - Add packages/dto as centralized DTO and API type definitions package
  - Add AGENTS.md, RULES.md"

  - Yes — run git commit now
  - No  — skip commit (leave files untracked)
```

If the user selects **No**, record "commit skipped (user declined)" in the PHASE 8 report.

If the user selects **Yes**:

```bash
git add .
git commit -m "chore: initialize pnpm monorepo workspace

- Add pnpm-workspace.yaml with apps/* and packages/* globs
- Add packages/core with tsconfig, eslint, vitest shared configs
- Add packages/dto as centralized DTO and API type definitions package
- Add AGENTS.md, RULES.md"
```

> ✅ `todowrite`: `p7` → `completed`

---

## PHASE 8: Report

> ▶ `todowrite`: `p8` → `in_progress`

```
AGENTS.md / RULES.md  : ./AGENTS.md, ./RULES.md

Workspace files
  root          : package.json, pnpm-workspace.yaml, tsconfig.json, .npmrc, .gitignore
  packages/core : tsconfig/{base,app,react-lib}.json
                  eslint/{index,react,node}.js
                  vitest.config.ts, src/index.ts, src/index.test.ts
  packages/dto  : package.json, tsconfig.json, eslint.config.js, vitest.config.ts
                  src/index.ts, src/index.test.ts
  apps/         : .gitkeep

git
  git init      : {done | skipped (already exists)}
  initial commit: done

Verification
  pnpm install       : {PASS | FAIL}
  core tsc --noEmit  : {PASS | FAIL}
  core lint          : {PASS | FAIL}
  core vitest run    : {PASS | FAIL}
  dto  tsc --noEmit  : {PASS | FAIL}
  dto  lint          : {PASS | FAIL}
  dto  vitest run    : {PASS | FAIL}
```

> ✅ `todowrite`: `p8` → `completed`

---

## Constraints

- Write files with `write_file` only — no `echo` or heredoc.
- Only commit if `git user.name` and `user.email` are configured.
- Stop immediately and report if `pnpm install` fails.
