---
description: Initialize a TypeScript Express API (DDD + Prisma) at the given directory. Detects monorepo context automatically — inherits CODE STANDARDS from root if found, otherwise interviews first.
---

Initialize a TypeScript Express API (DDD + Prisma) at `$ARGUMENTS`.
Detects monorepo context automatically — inherits CODE STANDARDS from root if found, otherwise interviews first.

---

## PHASE 0: Detect context

**Step 1 — Resolve target path.**
`$ARGUMENTS` is the directory to create (e.g. `apps/order-api`). Abort if it already contains source files.

**Step 2 — Detect monorepo.**
Traverse parent directories up to filesystem root. Check for `pnpm-workspace.yaml`.

- Found → **monorepo mode**: read root `RULES.md#code-standards`, skip PHASE 1.
- Not found → **standalone mode**: run PHASE 1.

---

## PHASE 1: Collect conventions (standalone mode only)

Use `ask_user_input_v0`:

```
Q-N1. File naming:   kebab-case | camelCase | PascalCase | snake_case
Q-N2. Class naming:  PascalCase | camelCase
Q-N3. Interface:     PascalCase (no prefix) | I-prefix + PascalCase
Q-N4. Functions:     camelCase + verb-prefix enforced | camelCase (verb recommended)
Q-T1. Type style:    interface-first | type-first | mixed
Q-T2. Fn style:      arrow-first | function-first | mixed
```

Project name = `$ARGUMENTS` basename (e.g. `apps/order-api` → `order-api`).

---

## PHASE 2: Collect API options

Use `ask_user_input_v0`:

```
Q-A1. DB engine:          PostgreSQL | MySQL | SQLite
Q-A2. JWT auth module:    include | exclude
Q-A3. IP filter middleware: include | exclude
```

---

## PHASE 3: Write AGENTS.md (and RULES.md if standalone)

Use `write_file` only — no shell redirects.

### 3-1. Monorepo mode → `$ARGUMENTS/AGENTS.md`

````markdown
# {TARGET_DIR} — AGENTS.md

This file covers only `{TARGET_DIR}`-specific rules. Everything else inherits from root.

> **Required:** Load root `/RULES.md` before running `/implement`.
> Find root: `!git rev-parse --show-toplevel`

| Inherited                | Source                     |
| ------------------------ | -------------------------- |
| TypeScript strict config | `/RULES.md#code-standards` |
| Naming conventions       | `/RULES.md#code-standards` |
| Type / function style    | `/RULES.md#code-standards` |
| Bundling rules           | `/RULES.md#code-standards` |

**Package role:** Express REST API — DDD layered architecture + Prisma
**Package type:** API server app
**Architecture:** [`ARCHITECTURE.md`](./ARCHITECTURE.md)

---

## STRUCTURE

```
prisma/
├── schema.prisma              # outside src/ — do not move
└── migrations/
src/
├── app.ts
├── lib/
│   ├── index.ts
│   ├── db.ts
│   └── db-error.ts
├── common/infrastructure/http/
│   └── ip-filter.middleware.ts   # [optional]
├── modules/{feature}/
│   ├── domain/
│   │   ├── {feature}.repository.interface.ts
│   │   └── {feature}.validation.ts
│   ├── application/
│   │   └── {feature}.service.ts
│   └── infrastructure/
│       ├── persistence/{feature}.repository.ts
│       └── http/
│           ├── {feature}.controller.ts
│           └── {feature}.route.ts
├── test/
│   ├── setup.ts
│   └── app.ts
└── generated/   # Prisma output — do not edit manually
```

Dependency direction: `infrastructure` → `application` → `domain`

---

## COMMANDS

| Task            | Command                         |
| --------------- | ------------------------------- |
| Dev server      | `pnpm dev`                      |
| Build           | `pnpm build`                    |
| Test            | `pnpm test`                     |
| Type check      | `pnpm typecheck`                |
| Prisma generate | `pnpm db:generate`              |
| Migrate         | `pnpm db:migrate --name <name>` |
| Push schema     | `pnpm db:push`                  |

---

## SAFETY & PERMISSIONS

**Autonomous:** read files, `tsc --noEmit`, `eslint`, single test.

**Require approval:** `pnpm add/remove`, `git push`, file/dir deletion, `db:migrate`, build/deploy.

---

## GIT CONVENTIONS

Inherits from `/AGENTS.md`. Conventional Commits. Pass `tsc --noEmit` + lint before PR.

---

## FILE REFERENCES

```
@{TARGET_DIR}/AGENTS.md      # this file
@RULES.md                     # root coding rules (required)
@{TARGET_DIR}/ARCHITECTURE.md # DDD layer rules + code templates
```

| When              | Load                      |
| ----------------- | ------------------------- |
| Writing code      | `RULES.md#code-standards` |
| Module/layer work | `ARCHITECTURE.md`         |
| Using the logger  | `RULES.md#logger`         |
| DB schema         | `prisma/schema.prisma`    |
| Env vars          | `.env.example`            |
````

---

### 3-2. Standalone mode → `$ARGUMENTS/AGENTS.md` + `$ARGUMENTS/RULES.md`

**AGENTS.md** — same structure as 3-1 but without the inheritance table header. Include full COMMANDS, SAFETY, GIT, FILE REFERENCES sections. FILE REFERENCES must reference `ARCHITECTURE.md` (not `RULES.md#api`).

**RULES.md** — fill `{…}` with PHASE 1 values:

```markdown
# {PROJECT_NAME} — RULES.md

> Load on demand via section anchors.

- [`#code-standards`](#code-standards)
- [`#logger`](#logger)
- [`#bigint-json`](#bigint-json) — BigInt-safe JSON · Express middleware

> DDD layer architecture, module structure, code templates, and new-module checklist → [`ARCHITECTURE.md`](./ARCHITECTURE.md)

---

## CODE STANDARDS {#code-standards}

### TypeScript

- `strict: true` required
- No `any` — use `unknown` + type guard
- Explicit return types
- Minimize `!` — prefer guards or optional chaining

### Naming

| Target          | Rule                               | Example                      |
| --------------- | ---------------------------------- | ---------------------------- |
| File            | {FILE_NAMING}                      | `{FILE_NAMING_EXAMPLE}`      |
| Class           | {CLASS_NAMING}                     | `{CLASS_NAMING_EXAMPLE}`     |
| Interface       | {INTERFACE_NAMING}                 | `{INTERFACE_NAMING_EXAMPLE}` |
| Type alias      | PascalCase                         | `UserDto`, `ApiResponse<T>`  |
| Function/method | {METHOD_NAMING}                    | `{METHOD_NAMING_EXAMPLE}`    |
| Constant        | SCREAMING_SNAKE_CASE               | `MAX_RETRY_COUNT`            |
| Enum            | PascalCase / SCREAMING_SNAKE value | `Status.ACTIVE`              |

### Type style

{TYPE_DEFINITION_STYLE}

### Function style

{FUNCTION_STYLE}

### Bundling

- Bundle: Vite (`vite build`)
- Run TS directly: `vite-node`
- `tsc` for type-check only — never as build tool
- Forbidden: esbuild, webpack, ts-node, tsx

### Do / Don't

- `async/await`, functional, single responsibility
- Throw errors to upper layers; narrow types
- No `console.log` in production; no magic numbers; no `any`

---

## LOGGER {#logger}

> **Monorepo:** if the monorepo provides a shared logger (e.g. `@{scope}/core`), this section describes that logger — `src/lib/logger.ts` is NOT generated locally.
> **Standalone:** the logger lives at `src/lib/logger.ts`, generated by the `ts-logger` skill.

### Import

```typescript
// Named logger — recommended for any class or module
import { createLogger } from '@/lib/logger';
const logger = createLogger('FeatureName');

// Default logger — for one-off scripts or simple entry points
import { logger } from '@/lib/logger';
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

## BIGINT JSON {#bigint-json}

> Apply when any Prisma column or API field uses the `BigInt` type.
> Run the `ts-bigint-json` skill to generate the implementation files.

`JSON.stringify` throws `TypeError` on `bigint` values; `JSON.parse` never produces `bigint`.
All BigInt ↔ JSON serialization must go through `bigintJson` — never use native `JSON.*` directly.

### Wire format

`{ "__bigint__": "9007199254740993" }` — tagged object, lossless, portable to any JSON client.

### Import

```typescript
import { bigintJson } from '@/lib/bigint-json';
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

### Do / Don't

- **Do** use `bigintJson.stringify` / `.parse` for any payload that may contain `bigint`
- **Do** use `bigintJsonMiddleware()` in the Express app instead of `express.json()`
- **Don't** use native `JSON.stringify` / `JSON.parse` for API payloads containing BigInt
- **Don't** cast `bigint` to `Number` — values > 2^53 − 1 lose precision silently
- **Don't** use `.toString()` inline as a workaround — it breaks downstream type contracts
```

### 3-3. `$ARGUMENTS/ARCHITECTURE.md` (always, both modes)

````markdown
# {PROJECT_NAME} — ARCHITECTURE.md

DDD 4-layer architecture reference for this project.
Read this file before creating or modifying any module.

---

## Layer diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    HTTP Layer (infrastructure/http)          │
│  Routes → Controllers (arrow functions for `this` binding)  │
├─────────────────────────────────────────────────────────────┤
│                  Application Layer                           │
│  Services — Business logic, orchestration                    │
├─────────────────────────────────────────────────────────────┤
│                       Domain Layer                           │
│  Interfaces (Repository), Validation Schemas (Zod)          │
│  DTOs, Entities                                             │
├─────────────────────────────────────────────────────────────┤
│              Persistence Layer (infrastructure/persistence)  │
│  Repository implementations (Prisma)                        │
└─────────────────────────────────────────────────────────────┘
```

Dependency direction: `HTTP → Application → Domain ← Persistence`

- **Domain**: depends on nothing. Defines interfaces, DTOs, entities, schemas.
- **Persistence**: implements Domain interfaces using Prisma.
- **Application**: depends only on Domain interfaces — never on concrete classes.
- **HTTP**: depends only on Application services.

---

## Directory structure

```
prisma/
├── schema.prisma          # Prisma schema — outside src/
└── migrations/

src/
├── app.ts
├── lib/
│   ├── index.ts
│   ├── db.ts              # Prisma client singleton
│   └── db-error.ts
├── modules/{module}/
│   ├── domain/
│   │   ├── {module}.repository.interface.ts
│   │   └── {module}.validation.ts
│   ├── application/
│   │   └── {module}.service.ts
│   └── infrastructure/
│       ├── http/
│       │   ├── {module}.controller.ts
│       │   └── {module}.route.ts
│       └── persistence/
│           └── {module}.repository.ts
├── common/infrastructure/http/
│   └── ip-filter.middleware.ts   # [optional]
├── test/
│   ├── setup.ts
│   └── app.ts
└── generated/             # Prisma output — do not edit manually
```

> `prisma/` lives at project root (outside `src/`).
> `src/generated/` is the Prisma client output path — never edit manually, regenerate with `pnpm db:generate`.

---

## Dependency injection

Constructor injection, assembled in the route file:

```typescript
import { prisma } from '@/lib/db';   // singleton — never new PrismaClient() in route files

const repository  = new {Module}Repository(prisma);
const service     = new {Module}Service(repository);
const controller  = new {Module}Controller(service);
```

---

## Layer rules

**Domain — `{module}.repository.interface.ts`**

- Repository interface named `I{Module}Repository`
- DTOs: `Create{Module}Dto`, `Update{Module}Dto`
- Entity: Prisma type as-is, or custom type excluding sensitive fields

**Domain — `{module}.validation.ts`**

- Zod schemas only (`create{Module}Schema`, `update{Module}Schema`)
- Infer DTO types with `z.infer<typeof schema>`
- No business logic here

**Persistence — `{module}.repository.ts`**

- Must `implements I{Module}Repository`
- Constructor injects `PrismaClient` (`private readonly prisma: PrismaClient`)
- DB access only — no business logic
- Exclude sensitive fields via `select` or destructuring

**Application — `{module}.service.ts`**

- Constructor parameter type must be the **interface** — never the concrete class
- No HTTP code (`req`, `res`, `next`) — ever
- Business rules (duplicate checks, existence checks, transactions) live here

**HTTP — `{module}.controller.ts`**

- All handlers must be **arrow functions** (`getAll = async (...) => {}`)
- Parse input with Zod: `schema.parse(req.body)`, `schema.parse(req.params)`
- No business logic — delegate entirely to service
- Always forward errors with `next(error)`

**HTTP — `{module}.route.ts`**

- Import `prisma` from `@/lib/db` — never `new PrismaClient()` here
- Assemble DI: Repository → Service → Controller
- Chain middleware at registration: `router.patch('/:id', authenticate, controller.update)`
- Pass handlers without calling: `controller.getAll` (arrow fn, no `.bind()` needed)

---

## HTTP conventions

- Status codes: 200, 201, 204, 400, 401, 403, 404, 422, 500
- Success: `{ ok: true, data: ... }`
- Error: `{ ok: false, message: string, code?: string }`

---

## Code templates

### `{module}.repository.interface.ts`

```typescript
import { User } from '@/generated/client';

export interface Create{Module}Dto { /* required fields */ }
export interface Update{Module}Dto { /* optional fields */ }
export type {Module}Entity = Omit<User, 'password'>;

export interface I{Module}Repository {
  findById(id: string): Promise<{Module}Entity | null>;
  findAll(): Promise<{Module}Entity[]>;
  create(dto: Create{Module}Dto): Promise<{Module}Entity>;
  update(id: string, dto: Update{Module}Dto): Promise<{Module}Entity>;
  delete(id: string): Promise<void>;
}
```

### `{module}.validation.ts`

```typescript
import { z } from 'zod';

export const create{Module}Schema = z.object({ /* fields */ });
export const update{Module}Schema = z.object({ /* optional fields */ });
export const {module}IdParamSchema = z.object({ id: z.string().uuid() });

export type Create{Module}Input = z.infer<typeof create{Module}Schema>;
export type Update{Module}Input = z.infer<typeof update{Module}Schema>;
```

### `{module}.repository.ts`

```typescript
import { PrismaClient } from '@/generated/client';
import { I{Module}Repository, Create{Module}Dto, Update{Module}Dto, {Module}Entity }
  from '../../domain/{module}.repository.interface';

export class {Module}Repository implements I{Module}Repository {
  constructor(private readonly prisma: PrismaClient) {}

  async findById(id: string): Promise<{Module}Entity | null> {
    return this.prisma.{model}.findUnique({ where: { id } });
  }
  async findAll(): Promise<{Module}Entity[]> {
    return this.prisma.{model}.findMany();
  }
  async create(dto: Create{Module}Dto): Promise<{Module}Entity> {
    return this.prisma.{model}.create({ data: dto });
  }
  async update(id: string, dto: Update{Module}Dto): Promise<{Module}Entity> {
    return this.prisma.{model}.update({ where: { id }, data: dto });
  }
  async delete(id: string): Promise<void> {
    await this.prisma.{model}.delete({ where: { id } });
  }
}
```

### `{module}.service.ts`

```typescript
import { I{Module}Repository, Create{Module}Dto, Update{Module}Dto, {Module}Entity }
  from '../domain/{module}.repository.interface';

export class {Module}Service {
  constructor(private readonly repository: I{Module}Repository) {}

  async getById(id: string): Promise<{Module}Entity> {
    const item = await this.repository.findById(id);
    if (!item) throw new Error(`{Module} not found: ${id}`);
    return item;
  }
  async getAll(): Promise<{Module}Entity[]> { return this.repository.findAll(); }
  async create(dto: Create{Module}Dto): Promise<{Module}Entity> { return this.repository.create(dto); }
  async update(id: string, dto: Update{Module}Dto): Promise<{Module}Entity> {
    await this.getById(id);
    return this.repository.update(id, dto);
  }
  async delete(id: string): Promise<void> {
    await this.getById(id);
    await this.repository.delete(id);
  }
}
```

### `{module}.controller.ts`

```typescript
import { Request, Response, NextFunction } from 'express';
import { {Module}Service } from '../../application/{module}.service';
import { create{Module}Schema, update{Module}Schema, {module}IdParamSchema }
  from '../../domain/{module}.validation';

export class {Module}Controller {
  constructor(private readonly service: {Module}Service) {}

  getAll = async (_req: Request, res: Response, next: NextFunction): Promise<void> => {
    try { res.status(200).json({ ok: true, data: await this.service.getAll() }); }
    catch (error) { next(error); }
  };
  getById = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    try {
      const { id } = {module}IdParamSchema.parse(req.params);
      res.status(200).json({ ok: true, data: await this.service.getById(id) });
    } catch (error) { next(error); }
  };
  create = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    try {
      const dto = create{Module}Schema.parse(req.body);
      res.status(201).json({ ok: true, data: await this.service.create(dto) });
    } catch (error) { next(error); }
  };
  update = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    try {
      const { id } = {module}IdParamSchema.parse(req.params);
      const dto = update{Module}Schema.parse(req.body);
      res.status(200).json({ ok: true, data: await this.service.update(id, dto) });
    } catch (error) { next(error); }
  };
  delete = async (req: Request, res: Response, next: NextFunction): Promise<void> => {
    try {
      const { id } = {module}IdParamSchema.parse(req.params);
      await this.service.delete(id);
      res.status(204).send();
    } catch (error) { next(error); }
  };
}
```

### `{module}.route.ts`

```typescript
import { Router } from 'express';
import { prisma } from '@/lib/db';
import { {Module}Repository } from '../persistence/{module}.repository';
import { {Module}Service } from '../../application/{module}.service';
import { {Module}Controller } from './{module}.controller';

const repository  = new {Module}Repository(prisma);
const service     = new {Module}Service(repository);
const controller  = new {Module}Controller(service);

const router = Router();
router.get('/',      controller.getAll);
router.get('/:id',   controller.getById);
router.post('/',     controller.create);
router.patch('/:id', controller.update);
router.delete('/:id', controller.delete);

export default router;
```

---

## New module checklist

- [ ] `domain/{module}.repository.interface.ts` — interface, DTOs, entity type
- [ ] `domain/{module}.validation.ts` — Zod schemas
- [ ] `infrastructure/persistence/{module}.repository.ts` — implements interface
- [ ] `application/{module}.service.ts` — business logic (inject interface, not class)
- [ ] `infrastructure/http/{module}.controller.ts` — arrow function handlers
- [ ] `infrastructure/http/{module}.route.ts` — DI assembly
- [ ] Application layer does not import any concrete repository class
- [ ] Controller contains no business logic
````

### 3-4. Save paths

```
$ARGUMENTS/AGENTS.md
$ARGUMENTS/RULES.md          (standalone mode only)
$ARGUMENTS/ARCHITECTURE.md
```

Report saved paths to the user.

---

## PHASE 4: Scaffold files

Create all files under `$ARGUMENTS/` using `write_file`. Apply Q-A1/A2/A3 choices where noted.

---

### `package.json`

Base — always included. Merge DB and option deps below.

```jsonc
{
  "name": "{project-name}",
  "version": "1.0.0",
  "type": "module",
  "main": "dist/app.js",
  "packageManager": "pnpm@10.28.2",
  "scripts": {
    "start": "node dist/app.js",
    "build": "vite build",
    "dev": "vite-node --watch src/app.ts",
    "typecheck": "tsc --noEmit",
    "db:generate": "prisma generate",
    "db:migrate": "prisma migrate dev",
    "db:push": "prisma db push",
    "db:push:test": "prisma db push --skip-generate",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage",
  },
  "dependencies": {
    "@prisma/client": "^7.4.1",
    "cors": "^2.8.5",
    "dotenv": "^17.2.3",
    "express": "^5.1.0",
    "zod": "^4.3.6",
    // PostgreSQL: "@prisma/adapter-pg": "^7.4.2", "pg": "^8.19.0"
    // MySQL:      "@prisma/adapter-mariadb": "^7.4.1"
    // SQLite:     "@prisma/adapter-better-sqlite3": "^7.4.1", "better-sqlite3": "^11.0.0"
    // Auth:       "bcrypt": "^6.0.0", "jose": "^6.1.0"
    // IP filter:  "ip-range-check": "^0.2.0"
  },
  "devDependencies": {
    "@prisma/config": "^7.4.1",
    "@types/cors": "^2.8.19",
    "@types/express": "^5.0.3",
    "@types/supertest": "^7.2.0",
    "prisma": "^7.4.1",
    "supertest": "^7.2.2",
    "typescript": "^5.8.3",
    "vite": "^6.0.0",
    "vite-node": "^6.0.0",
    "vitest": "^3.2.4",
    // PostgreSQL: "@types/pg": "^8.16.0"
    // Auth:       "@types/bcrypt": "^6.0.0"
    // SQLite:     "@types/better-sqlite3": "^7.6.0"
  },
}
```

---

### `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "es2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",
    "esModuleInterop": true,
    "forceConsistentCasingInFileNames": true,
    "strict": true,
    "skipLibCheck": true,
    "rootDir": "./src",
    "outDir": "./dist",
    "resolveJsonModule": true,
    "paths": { "@/*": ["./src/*"] }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

In monorepo mode: add `"extends": "@{scope}/core/tsconfig/app.json"` and remove duplicated fields already covered by base.

> ⚠️ **`moduleResolution: "bundler"` is NEVER removed in monorepo mode.** Even when extending a base tsconfig that uses `NodeNext`, this field must be explicitly present in the local `tsconfig.json`. Vite resolves `@/` path aliases at build time; TypeScript needs `bundler` resolution for `paths` to work at type-check time. Do not treat it as a duplicate.

---

### `vite.config.ts`

```typescript
import { defineConfig } from "vite";
import path from "path";
import { fileURLToPath } from "url";
import { builtinModules } from "module";

const __dirname = path.dirname(fileURLToPath(import.meta.url));

const prodDeps = ["express", "cors", "dotenv", "jose", "ip-range-check", "zod"];

// PostgreSQL: include '@prisma/adapter-pg', 'pg', 'pg-native'
// MySQL:      include '@prisma/adapter-mariadb'
// SQLite:     include 'better-sqlite3' (native binary)
const nativeModules = [
  "bcrypt",
  "pg",
  "pg-native",
  "better-sqlite3",
  "@prisma/adapter-mariadb",
  "@prisma/adapter-pg",
  "prisma",
];

const isExternal = (id: string): boolean => {
  if (builtinModules.includes(id) || id.startsWith("node:")) return true;
  if (
    id.includes("@prisma/client") ||
    id.startsWith("@prisma/") ||
    id.includes("/prisma/")
  )
    return true;
  if (nativeModules.some((m) => id === m || id.startsWith(`${m}/`)))
    return true;
  if (prodDeps.includes(id)) return true;
  return false;
};

export default defineConfig({
  build: {
    outDir: "dist",
    minify: false,
    sourcemap: true,
    ssr: true,
    ssrEmitAssets: false,
    target: "node22",
    rollupOptions: {
      input: { app: path.resolve(__dirname, "src/app.ts") },
      external: (id) => isExternal(id),
      output: {
        entryFileNames: "[name].js",
        chunkFileNames: "chunks/[name]-[hash].js",
        assetFileNames: "assets/[name]-[hash][extname]",
        format: "es",
      },
    },
  },
  resolve: { alias: { "@": path.resolve(__dirname, "src") } },
});
```

---

### `vitest.config.ts`

```typescript
import { defineConfig } from "vitest/config";
import path from "path";
import { fileURLToPath } from "url";

const __dirname = path.dirname(fileURLToPath(import.meta.url));

export default defineConfig({
  test: {
    globals: true,
    environment: "node",
    include: ["src/**/*.test.ts", "src/**/*.spec.ts"],
    testTimeout: 30000,
    hookTimeout: 30000,
    setupFiles: ["./src/test/setup.ts"],
    fileParallelism: false,
    sequence: { concurrent: false },
  },
  resolve: { alias: { "@": path.resolve(__dirname, "src") } },
});
```

---

### `prisma.config.ts`

```typescript
import "dotenv/config";
import { defineConfig } from "prisma/config";

export default defineConfig({
  schema: "prisma/schema.prisma",
  migrations: { path: "prisma/migrations" },
  datasource: { url: process.env["DATABASE_URL"] },
});
```

---

### `prisma/schema.prisma` — PostgreSQL (Q-A1 = PostgreSQL)

> `prisma/` is at project root, outside `src/`. The `output = "../src/generated"` path is relative to this file.

```prisma
generator client {
  provider = "prisma-client"
  output   = "../src/generated"
}

datasource db {
  provider = "postgresql"
}

model User {
  id        String   @id @default(uuid())
  loginId   String   @unique @db.VarChar(128)
  password  String   @db.VarChar(255)
  name      String   @db.VarChar(128)
  email     String?  @db.VarChar(255)
  isUse     Boolean  @default(true)
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  @@map("user")
}
```

### `prisma/schema.prisma` — MySQL (Q-A1 = MySQL)

> Same location: `prisma/` at project root, outside `src/`.

```prisma
generator client {
  provider = "prisma-client"
  output   = "../src/generated"
}

datasource db {
  provider = "mysql"
}

model User {
  id        String   @id @default(uuid()) @db.VarChar(36)
  loginId   String   @unique @db.VarChar(128)
  password  String   @db.VarChar(255)
  name      String   @db.VarChar(128)
  email     String?  @db.VarChar(255)
  isUse     Boolean  @default(true)
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  @@map("user")
}
```

> MySQL: `@id @default(uuid())` must include `@db.VarChar(36)`.

---

### `prisma/schema.prisma` — SQLite (Q-A1 = SQLite)

> Same location: `prisma/` at project root, outside `src/`.
> SQLite maps all `String` fields to `TEXT` — do **not** use `@db.VarChar()` annotations.

```prisma
generator client {
  provider = "prisma-client"
  output   = "../src/generated"
}

datasource db {
  provider = "sqlite"
}

model User {
  id        String   @id @default(uuid())
  loginId   String   @unique
  password  String
  name      String
  email     String?
  isUse     Boolean  @default(true)
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  @@map("user")
}
```

---

### `src/app.ts`

```typescript
import express, { Application, Request, Response, NextFunction } from "express";
import cors from "cors";
import { createLogger } from "@/lib/logger";
// [Q-A3 = include] import { allowIP } from "@/common/infrastructure/http/ip-filter.middleware";
//   ↑ generated by the ts-ip-filter skill
// [Q-A2 = include] import authRouter from "@/modules/auth/infrastructure/http/auth.route";
//   ↑ generated by the ts-auth-module skill

const logger = createLogger("app");
const app: Application = express();
app.use(cors());
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// [Q-A3 = include]
// const ADMIN_ALLOWED_IPS = ["::1", "127.0.0.1"];
// app.use("/auth", allowIP(ADMIN_ALLOWED_IPS), authRouter);

app.get("/ping", (_req: Request, res: Response) => {
  res.status(200).send("pong");
});

app.use((_req: Request, res: Response) => {
  res.status(404).json({ ok: false, message: "Not found" });
});

// ⚠️ Express error-handling middleware REQUIRES exactly 4 parameters.
// Without the 4th (_next), Express treats this as a regular middleware and errors are not caught.
// If ESLint flags _next as unused, add eslint-disable on the line — NEVER remove the parameter.
// eslint-disable-next-line @typescript-eslint/no-unused-vars
app.use((err: unknown, _req: Request, res: Response, _next: NextFunction) => {
  const error = err as { status?: number; message?: string; stack?: string };
  logger.error("Unhandled error", { stack: error.stack });
  res.status(error.status ?? 500).json({
    ok: false,
    message: error.message ?? "Internal server error",
  });
});

const PORT = process.env.PORT ?? 3000;
app.listen(PORT, () => logger.log(`Server ready → http://localhost:${PORT}`));

export default app;
```

---

### `src/lib/index.ts`

```typescript
export * from "./db";
export * from "./db-error";
export * from "./logger";
```

> **Logger resolution — run exactly one branch, no deliberation:**
>
> | Mode | Condition | Action |
> |------|-----------|--------|
> | **Monorepo** | Root `RULES.md` mentions a shared logger package (e.g. `@{scope}/core`) | **Skip `ts-logger` skill.** Do NOT generate `src/lib/logger.ts`. Remove `export * from "./logger"` from `index.ts`. All logger imports in generated files use the shared package path. |
> | **Standalone** | No shared logger found | **Run `ts-logger` skill** in standalone mode. Keep `export * from "./logger"` in `index.ts`. |

---

### `src/lib/db.ts` — PostgreSQL (Q-A1 = PostgreSQL)

```typescript
import "dotenv/config";
import { PrismaClient } from "@/generated/client";
import { PrismaPg } from "@prisma/adapter-pg";
import pg from "pg";

const pool = new pg.Pool({ connectionString: process.env.DATABASE_URL });
const adapter = new PrismaPg(pool);

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    adapter,
    log: process.env.NODE_ENV === "development" ? ["error", "warn"] : ["error"],
  });

if (process.env.NODE_ENV !== "production") globalForPrisma.prisma = prisma;

// beforeExit supports async; exit does not — never use process.on("exit") for $disconnect()
process.on("beforeExit", async () => {
  await prisma.$disconnect();
});
```

### `src/lib/db.ts` — MySQL (Q-A1 = MySQL)

```typescript
import "dotenv/config";
import { PrismaClient } from "@/generated/client";
import { PrismaMariaDb } from "@prisma/adapter-mariadb";

const adapter = new PrismaMariaDb(process.env["DATABASE_URL"]!);

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    adapter,
    log: process.env["NODE_ENV"] === "development" ? ["error", "warn"] : ["error"],
  });

if (process.env["NODE_ENV"] !== "production") globalForPrisma.prisma = prisma;

process.on("beforeExit", async () => {
  await prisma.$disconnect();
});
// Note: process.on("exit") does not support async — $disconnect() is handled by beforeExit only.
```

---

### `src/lib/db.ts` — SQLite (Q-A1 = SQLite)

```typescript
import "dotenv/config";
import { PrismaClient } from "@/generated/client";
import { PrismaBetterSqlite3 } from "@prisma/adapter-better-sqlite3";

const adapter = new PrismaBetterSqlite3({
  url: process.env["DATABASE_URL"] ?? "file:./prisma/dev.db",
});

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    adapter,
    log: process.env["NODE_ENV"] === "development" ? ["error", "warn"] : ["error"],
  });

if (process.env["NODE_ENV"] !== "production") globalForPrisma.prisma = prisma;

process.on("beforeExit", async () => {
  await prisma.$disconnect();
});
// Note: process.on("exit") does not support async — $disconnect() is handled by beforeExit only.
```

---

### `src/lib/db-error.ts`

```typescript
import { Prisma } from "@/generated/client";
import { createLogger } from "@/lib/logger";

const logger = createLogger("db-error");

export function isPrismaError(
  error: unknown,
): error is Prisma.PrismaClientKnownRequestError {
  return error instanceof Prisma.PrismaClientKnownRequestError;
}

export function handleControllerError(error: unknown, context: string) {
  logger.error(`[${context}] Error`, error);

  if (isPrismaError(error)) {
    return {
      status: 400,
      response: { ok: false, message: `DB error: ${error.message}` },
    };
  }
  if (error instanceof Error) {
    return { status: 400, response: { ok: false, message: error.message } };
  }
  return { status: 500, response: { ok: false, message: "Unknown error" } };
}
```

---

### IP Filter Middleware (Q-A3 = include)

> **Skill delegation:** Load and execute the `ts-ip-filter` skill.
> Pass `TARGET_DIR = $ARGUMENTS` as context.
> The skill handles file creation and integration instructions.

---

### `src/test/setup.ts`

```typescript
import { beforeAll, afterAll } from "vitest";
import { prisma } from "@/lib/db";

beforeAll(async () => {
  await prisma.$connect();
});
afterAll(async () => {
  await prisma.$disconnect();
});

export async function cleanAllTables(): Promise<void> {
  // e.g. await prisma.userToken.deleteMany({});
  //      await prisma.user.deleteMany({});
}

export { prisma };
```

---

### `src/test/app.ts`

```typescript
import express from "express";
import cors from "cors";

const app = express();
app.use(cors());
app.use(express.json());

app.get("/ping", (_req, res) => {
  res.status(200).send("pong");
});

export const createTestApp = () => app;
```

---

### JWT Auth Module (Q-A2 = include)

> **Skill delegation:** Load and execute the `ts-auth-module` skill.
> Pass `TARGET_DIR = $ARGUMENTS` and `Q-A1 = {selected DB engine}` as context.
> The skill handles the full DDD auth module generation, Prisma schema patch instructions, and app.ts integration.

---

### `.env`

```env
NODE_ENV=development
PORT=3000

# PostgreSQL
DATABASE_URL="postgresql://user:password@localhost:5432/dbname"

# MySQL (use instead of PostgreSQL)
# DATABASE_URL="mysql://user:password@localhost:3306/dbname"

# SQLite (use instead of PostgreSQL/MySQL)
# DATABASE_URL="file:./prisma/dev.db"

# Auth (Q-A2 = include)
# JWT_ACCESS_SECRET="min-32-chars"
# JWT_REFRESH_SECRET="min-32-chars"
# JWT_ACCESS_EXPIRE="3600"
# JWT_REFRESH_EXPIRE="1209600"
# JWT_HASH_SALT="10"
# PASSWORD_HASH_SALT="10"
```

### `.env.example` — same as `.env` but with placeholder values (always commit this)

---

### `.gitignore` — skip if already exists

```
node_modules/
dist/
.env
.env.local
*.tsbuildinfo
coverage/
src/generated/
prisma/*.db
prisma/*.db-journal
```

---

## PHASE 4-S: Skill delegation

Run the following skills **in order**. Each has a hard branch — pick one and execute, no investigation needed.

### Logger

| MODE | Rule | Action |
|------|------|--------|
| Monorepo | Root `RULES.md` references a shared logger (e.g. `@{scope}/core`) | **Skip `ts-logger`.** `src/lib/logger.ts` is NOT created. `index.ts` already omits the logger export (see template). All `createLogger` imports in generated files must use the shared package (e.g. `@chkvec/core`). |
| Standalone | No shared logger found | **Run `ts-logger` skill** in standalone mode with `TARGET_DIR = $ARGUMENTS`. |

### IP Filter (Q-A3 = include only)

Run `ts-ip-filter` skill. Pass `TARGET_DIR = $ARGUMENTS`.

### JWT Auth (Q-A2 = include only)

Run `ts-auth-module` skill. Pass `TARGET_DIR = $ARGUMENTS`, `DB = {Q-A1}`.

---

## PHASE 5: git init (standalone mode only)

**Skip this phase if MODE=monorepo** — the root repository already covers this package.

If `.git/` already exists in `$ARGUMENTS`, skip and proceed to the next phase.

Otherwise:

```bash
git init
git branch -M main
```

If `git user.name` / `user.email` is not configured, notify the user and record the warning.
The initial commit will be requested in **PHASE 8** after verification and troubleshooting.

---

## PHASE 6: Verify

```bash
cd $ARGUMENTS
pnpm install
pnpm typecheck
pnpm eslint --fix .
pnpm db:generate   # requires DATABASE_URL — skip and notify user if not set
pnpm test          # skip if DATABASE_URL not set; notify user
```

If `DATABASE_URL` is not configured, report which steps were skipped and what to run manually.

---

## PHASE 7: Record troubleshooting

Call `@document-writer` with:

> Extract AI mistakes and incorrect implementations from this entire workflow and record them in `TROUBLE_SHOOT.md` at the project root.
>
> **What to extract** — record only these types (exclude normal design decisions or user requirement changes):
> - Code incorrectly implemented by AI that required fixes — **even if fixed during scaffolding**
> - Repeated error patterns in the self-correction loop
> - Gaps or errors flagged by Metis or Momus
> - Assertion translation errors found in spec↔test validation
>
> **Trigger rule:** if ANY item above occurred during this workflow, create or update `TROUBLE_SHOOT.md`.
> The fact that an issue was resolved before completion does NOT exempt it from being recorded —
> resolved issues are the most valuable entries because they contain the fix.
> Skip only if the workflow completed with zero AI mistakes and zero self-correction loops.
>
> **Format** — write each item as:
>
> ```markdown
> ## [YYYY-MM-DD] {task keyword}
>
> - {one-line rule or checkpoint to prevent recurrence}
> - {add lines if multiple items}
> ```
>
> **File handling:**
> - If `TROUBLE_SHOOT.md` already exists — keep existing content and prepend the new entry
> - If it does not exist — create it
> - If there are no troubleshooting items from this workflow — skip this step

---

## PHASE 8: Initial Commit ⚠️

If `.git/` does not exist in the project root, skip this phase entirely.

If `git user.name` / `user.email` was not configured, remind the user and skip.

Present the following via `ask_user_input_v0`:

```
Verification and troubleshooting recording are complete.
Proceed with the initial git commit?

  Commit message:
  "chore: initialize TypeScript Express API project

  - Add AGENTS.md, RULES.md, ARCHITECTURE.md
  - Scaffold DDD + Prisma project structure"

  - Yes — run git commit now
  - No  — skip commit (leave files untracked)
```

If the user selects **No**, record "commit skipped (user declined)" in the PHASE 9 report.

If the user selects **Yes**:

```bash
git add .
git commit -m "chore: initialize TypeScript Express API project\n\n- Add AGENTS.md, RULES.md, ARCHITECTURE.md\n- Scaffold DDD + Prisma project structure"
```

---

## PHASE 9: Report

```
AGENTS.md           : $ARGUMENTS/AGENTS.md
RULES.md            : {$ARGUMENTS/RULES.md (standalone) | root/RULES.md (inherited)}
ARCHITECTURE.md     : $ARGUMENTS/ARCHITECTURE.md
Mode                : {monorepo | standalone}

Scaffold
  config            : package.json, tsconfig.json
                      vite.config.ts, vitest.config.ts, prisma.config.ts
  prisma            : prisma/schema.prisma ({PostgreSQL | MySQL | SQLite})
  src               : app.ts, lib/{index,db,db-error}.ts
                      test/{setup,app}.ts
  optional          : {ip-filter.middleware.ts | —} (Q-A3)
                      {auth module placeholder | —} (Q-A2)
  env               : .env, .env.example, .gitignore

Verification
  pnpm install      : {PASS | FAIL}
  tsc --noEmit      : {PASS | FAIL}
  eslint            : {PASS | FAIL}
  db:generate       : {PASS | SKIPPED — DATABASE_URL not set}
  vitest run        : {PASS | SKIPPED — DATABASE_URL not set}
```

---

## Constraints

- Write files with `write_file` only — no `echo` or heredoc.
- Abort if `$ARGUMENTS` already contains source files.
- Apply Q-A1/A2/A3 choices precisely — do not include excluded modules.
- SQLite (Q-A1 = SQLite): do **not** use `@db.VarChar()` or any `@db.*` annotations — SQLite maps all `String` fields to `TEXT`.
- In monorepo mode: `tsconfig.json` must extend `@{scope}/core/tsconfig/app.json`; do not duplicate fields — **except `moduleResolution: "bundler"` which must always be present** even when the base uses `NodeNext`.
- `.env` is never committed — always create `.env.example` alongside it.
- `prisma/` lives at project root, outside `src/`. Never place schema files inside `src/`.
- `src/generated/` is Prisma client output — never edit manually; regenerate with `pnpm db:generate`.
- **Prisma 7 + driver adapter:** do NOT include `url = env("DATABASE_URL")` in `datasource db {}` block of `schema.prisma`. The connection URL is handled entirely by the adapter constructor and `prisma.config.ts`.
- **Express error handler:** the 4-parameter signature `(err, _req, res, _next)` is mandatory. Express detects error middleware by parameter count. Never remove `_next` even if unused — add `// eslint-disable-next-line @typescript-eslint/no-unused-vars` above the line instead.
- **`process.on("exit")`** does not support async callbacks — `$disconnect()` will not complete. Use `beforeExit` only.
