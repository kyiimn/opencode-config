---
description: Initialize an empty TypeScript project at the specified directory with AGENTS.md, RULES.md, and ARCHITECTURE.md. Generates project structure, naming conventions, and config files — no framework, no CLI tooling.
---

Initialize an empty TypeScript project at the specified directory with AGENTS.md, RULES.md, and ARCHITECTURE.md.

---

## Behavior

When this command is invoked, immediately start from **PHASE 1** without waiting for further instructions.

The project name is derived from the **target directory name** — do not ask the user for a project name.

---

## TODO MANAGEMENT — Required

커멘드 시작 직후 `todowrite`를 호출하여 전체 페이즈를 등록하라. 각 페이즈 진입 시 `in_progress`, 완료 시 `completed`, 조건부 스킵 시 `cancelled`로 업데이트하라.

**`todowrite`는 전체 목록을 교체하므로, 호출 시 항상 전체 항목을 포함해야 한다.**

### 초기 `todowrite` 호출 (커멘드 시작 즉시)

```json
[
  { "id": "p1",   "content": "PHASE 1: Monorepo Detection",                          "priority": "high",   "status": "pending" },
  { "id": "p2a",  "content": "PHASE 2A: Inherit CODE STANDARDS (monorepo only)",     "priority": "high",   "status": "pending" },
  { "id": "p2b",  "content": "PHASE 2B: Interview — naming/style (standalone only)", "priority": "high",   "status": "pending" },
  { "id": "p3",   "content": "PHASE 3: Generate AGENTS.md",                          "priority": "high",   "status": "pending" },
  { "id": "p4",   "content": "PHASE 4: Generate RULES.md",                           "priority": "high",   "status": "pending" },
  { "id": "p5",   "content": "PHASE 5: Generate ARCHITECTURE.md",                    "priority": "high",   "status": "pending" },
  { "id": "p6",   "content": "PHASE 6: Save Files",                                  "priority": "high",   "status": "pending" },
  { "id": "p6m",  "content": "PHASE 6-M: Update root AGENTS.md (monorepo only)",     "priority": "medium", "status": "pending" },
  { "id": "p7",   "content": "PHASE 7: Greenfield Detection",                        "priority": "high",   "status": "pending" },
  { "id": "p8",   "content": "PHASE 8: Initialization Approval Gate (user approval)","priority": "high",   "status": "pending" },
  { "id": "p9",   "content": "PHASE 9: Initialize Project Structure",                "priority": "high",   "status": "pending" },
  { "id": "p10",  "content": "PHASE 10: git init (standalone only)",                 "priority": "medium", "status": "pending" },
  { "id": "p11",  "content": "PHASE 11: Verification Pipeline",                      "priority": "high",   "status": "pending" },
  { "id": "p12",  "content": "PHASE 12: Record troubleshooting",                     "priority": "medium", "status": "pending" },
  { "id": "p13",  "content": "PHASE 13: Initial Commit (user approval required)",    "priority": "medium", "status": "pending" },
  { "id": "p14",  "content": "PHASE 14: Completion Report",                          "priority": "low",    "status": "pending" }
]
```

---

## PHASE 1: Monorepo Detection

> ▶ `todowrite`: `p1` → `in_progress`

Check if a `pnpm-workspace.yaml` exists by traversing parent directories up from the target path.

```
Detection rule:
  Walk upward from $ARGUMENTS until filesystem root.
  If pnpm-workspace.yaml is found → monorepo mode (PHASE 2A)
  If not found                    → standalone mode (PHASE 2B)
```

> ✅ `todowrite`: `p1` → `completed`

---

## PHASE 2A: Monorepo Mode — Inherit CODE STANDARDS from root

> ▶ `todowrite`: `p2a` → `in_progress`, `p2b` → `cancelled`

Read the root `RULES.md` found in PHASE 1.

Extract the `## CODE STANDARDS {#code-standards}` section and use those values directly:
- File naming convention
- Class naming convention
- Interface naming convention
- Method/function naming convention
- Type definition style (interface-first / type-first / mixed)
- Function declaration style (arrow / function keyword / mixed)

**Do not interview the user for any item that is explicitly defined in the root RULES.md.**

If any item is missing or ambiguous in the root RULES.md, interview only for those items using the questions defined in **PHASE 2B**.

After resolving all values, proceed to **PHASE 3**.

> ✅ `todowrite`: `p2a` → `completed`

---

## PHASE 2B: Standalone Mode — Interview

> ▶ `todowrite`: `p2b` → `in_progress`, `p2a` → `cancelled`

Collect the following via `ask_user_input_v0`:

```
[Naming Conventions]

Q-N1. File naming
  - kebab-case  (e.g. user-service.ts)
  - camelCase   (e.g. userService.ts)
  - PascalCase  (e.g. UserService.ts)

Q-N2. Class naming
  - PascalCase  (e.g. UserService, DataProcessor)
  - camelCase

Q-N3. Interface naming
  - PascalCase, no prefix  (e.g. User, Config)
  - I-prefix + PascalCase  (e.g. IUser, IConfig)

Q-N4. Method/function naming
  - camelCase + verb prefix enforced  (e.g. getUser, handleError)
  - camelCase, verb prefix recommended but not enforced

[TypeScript Style]

Q-T1. Type definition style
  - interface-first  (objects → interface; unions/intersections → type)
  - type-first       (all type definitions use type alias)
  - mixed            (choose based on context)

Q-T2. Function declaration style
  - arrow-first      (const fn = () => {})
  - function-first   (function fn() {})
  - mixed: top-level → function, class methods/callbacks → arrow
```

After collecting all answers, proceed to **PHASE 3**.

> ✅ `todowrite`: `p2b` → `completed`

---

## PHASE 3: Generate AGENTS.md

> ▶ `todowrite`: `p3` → `in_progress`

Create `$ARGUMENTS/AGENTS.md` with the following content.

Replace all `{placeholders}` with resolved values. Hardcode the structure — no template files exist for this command.

**If MODE=monorepo**, use this template:

```markdown
# {DIR_NAME} — AGENTS.md

**Package Manager:** pnpm
**Architecture:** Stack · project structure → [`ARCHITECTURE.md`](./ARCHITECTURE.md)
**Rules:** Coding standards → root [`RULES.md`](../../RULES.md)

## COMMANDS

| Purpose | Command | Where |
|---------|---------|-------|
| Type check | `pnpm tsc --noEmit` | package dir |
| Run single test | `pnpm vitest run {filepath}` | package dir |
| Run all tests | `pnpm vitest run` | root |
| Lint | `pnpm eslint --fix {filepath}` | root |
| Build | `pnpm build` | **only on explicit request** |

> Type check and tests run autonomously after implementation. Build/deploy requires user approval.

## SAFETY & PERMISSIONS

### Autonomous execution allowed
- File reads, directory listing
- `tsc --noEmit`, `eslint`, `prettier`
- Single test file execution

### Requires user approval before execution
- Package install/remove (`pnpm add`, `pnpm remove`)
- `git push`, branch creation, PR creation
- File/directory deletion
- Full build or deployment
- `.env` file modification

## GIT CONVENTIONS

Inherits from root `AGENTS.md`. Conventional Commits. Pass `tsc --noEmit` + lint before PR.

## FILE REFERENCES

```
@AGENTS.md                  # this file (package-specific rules)
@{root-path}/RULES.md       # root coding rules (required)
@ARCHITECTURE.md            # project structure and build model
```

If root path is unknown: run `!git rev-parse --show-toplevel` to find it.
Starting work without root `RULES.md` is prohibited.

### Load by section — only when relevant

| When | File and section |
|------|-----------------|
| Writing or modifying code | `{root}/RULES.md#code-standards` |
| Project structure / patterns | `ARCHITECTURE.md` |
| Writing tests | `{root}/RULES.md#code-standards` |
| Using the logger | `{root}/RULES.md#logger` |
| Build / execution rules | `ARCHITECTURE.md#execution-model` |
| Environment variables | `.env.example` |
```

**If MODE=standalone**, use this template:

```markdown
# {DIR_NAME} — AGENTS.md

**Package Manager:** pnpm
**Architecture:** Stack · project structure → [`ARCHITECTURE.md`](./ARCHITECTURE.md)
**Rules:** Coding standards → [`RULES.md`](./RULES.md)

## COMMANDS

| Purpose | Command | Where |
|---------|---------|-------|
| Type check | `pnpm tsc --noEmit` | project dir |
| Run single test | `pnpm vitest run {filepath}` | project dir |
| Run all tests | `pnpm vitest run` | project dir |
| Lint | `pnpm eslint --fix {filepath}` | project dir |
| Build | `pnpm build` | **only on explicit request** |

> Type check and tests run autonomously after implementation. Build/deploy requires user approval.

## SAFETY & PERMISSIONS

### Autonomous execution allowed
- File reads, directory listing
- `tsc --noEmit`, `eslint`, `prettier`
- Single test file execution

### Requires user approval before execution
- Package install/remove (`pnpm add`, `pnpm remove`)
- `git push`, branch creation, PR creation
- File/directory deletion
- Full build or deployment
- `.env` file modification

## GIT CONVENTIONS

- Branches: `feature/{task}`, `fix/{issue}`, `chore/{task}`
- Commits: Conventional Commits (`feat:` `fix:` `chore:` `docs:` `refactor:` `test:`)
- `tsc --noEmit` and lint must pass before PR

## FILE REFERENCES

```
@AGENTS.md       # this file
@RULES.md        # coding rules (required)
@ARCHITECTURE.md # project structure and build model
```

### Load by section — only when relevant

| When | File and section |
|------|-----------------|
| Writing or modifying code | `RULES.md#code-standards` |
| Project structure / patterns | `ARCHITECTURE.md` |
| Writing tests | `RULES.md#code-standards` |
| Using the logger | `RULES.md#logger` |
| Build / execution rules | `ARCHITECTURE.md#execution-model` |
| Environment variables | `.env.example` |
```

> ✅ `todowrite`: `p3` → `completed`

---

## PHASE 4: Generate RULES.md

> ▶ `todowrite`: `p4` → `in_progress`

Create `$ARGUMENTS/RULES.md` with the following content.

Fill in `{placeholders}` from the values resolved in PHASE 2A or PHASE 2B.

```markdown
# {DIR_NAME} — RULES.md

> Load on demand — do not read the entire file at once.
> Specify the section anchor for the part you need.

**Section Index**
- [`#code-standards`](#code-standards) — TypeScript · naming · Do/Don't
- [`#logger`](#logger) — Logger import paths · level rules · Do/Don't
- [`#bigint-json`](#bigint-json) — BigInt-safe JSON · fetch wrapper

> Build toolchain and execution model → [`ARCHITECTURE.md`](./ARCHITECTURE.md)

---

## CODE STANDARDS {#code-standards}

### TypeScript
- **strict mode required** (`"strict": true` in tsconfig)
- `any` is prohibited — use `unknown` with type guards
- Explicit return types required (`noImplicitReturns: true`)
- Non-null assertion (`!`) — minimize; prefer guards or optional chaining

### Naming Conventions

| Target | Rule | Example |
|--------|------|---------|
| File | {FILE_NAMING_CONVENTION} | `{FILE_NAMING_EXAMPLE}` |
| Class | {CLASS_NAMING} | `{CLASS_NAMING_EXAMPLE}` |
| Interface | {INTERFACE_NAMING} | `{INTERFACE_NAMING_EXAMPLE}` |
| Type alias | PascalCase | `UserDto`, `AppConfig<T>` |
| Function/method | {METHOD_NAMING} | `{METHOD_NAMING_EXAMPLE}` |
| Constant | SCREAMING_SNAKE_CASE | `MAX_RETRY_COUNT` |
| Enum | PascalCase (name) / SCREAMING_SNAKE (value) | `ExitCode.SUCCESS` |

### Type Definition Style

{TYPE_DEFINITION_STYLE}

### Function Declaration Style

{FUNCTION_STYLE}

### Do
- Functional style, single responsibility principle
- `async/await` (no callbacks)
- Always throw errors up to the caller layer
- Keep types as narrow as possible (use union and literal types)

### Don't
- No `console.log` in production — use a structured logger
- No magic numbers/strings — extract as constants
- Do not commit with `// TODO` left unresolved
- Do not install new external packages without user approval
- No `any`

---

## LOGGER {#logger}

> **Logger dispatch — branch immediately, no investigation:**
>
> | MODE | Condition | Action |
> |------|------|--------|
> | Monorepo | Root `RULES.md` references a shared logger package (e.g. `@{scope}/core`) | **Skip `ts-logger` skill.** Do not generate `src/lib/logger.ts`. Update logger import paths in generated files to use the shared package. Add `"@{scope}/core": "workspace:*"` to `package.json` dependencies. |
> | Standalone | No shared logger found | **Run `ts-logger` skill in standalone mode.** |
>
> No source scan needed — import paths and level rules are fixed.

### Import

```typescript
// Named logger — recommended for any module or service file
import { createLogger } from '@/lib/logger';
const logger = createLogger('FeatureName');

// Default logger — for simple entry points
import { logger } from '@/lib/logger';
```

### Levels

| Level | When to use |
|-------|-------------|
| `debug` | Internal state, tracing — **suppressed in production** |
| `log` | Normal lifecycle events (job start, job complete) |
| `warn` | Unexpected but recoverable situation (retry, fallback, deprecation) |
| `error` | Exception, external service failure, unrecoverable error |

### Usage

```typescript
logger.debug('Parsed input', { value });          // dev only
logger.log('Operation complete', { output });
logger.warn('Config key deprecated', { key });
logger.error('Unexpected failure', error);
```

### Do / Don't

- **Do** call `createLogger('ContextName')` once at the top of each module file
- **Do** pass structured data as the second argument instead of string interpolation
- **Don't** use `console.log`, `console.warn`, or `console.error` in application code
- **Don't** log sensitive data (tokens, credentials, PII) at any level
---

## BIGINT JSON {#bigint-json}

> Apply when any API endpoint returns Prisma `BigInt` fields.
> Run the `ts-bigint-json` skill to generate the implementation files.

`JSON.stringify` throws `TypeError` on `bigint` values; `JSON.parse` never produces `bigint`.
All BigInt ↔ JSON serialization must go through `bigintJson` — never use native `JSON.*` directly.

### Wire format

`{ "__bigint__": "9007199254740993" }` — tagged object, lossless, portable to any JSON client.

### Import

```typescript
// monorepo
import { bigintJson } from '@{scope}/core';

// standalone
import { bigintJson } from '@/lib/bigint-json';
```

### Fetch wrapper

```typescript
import { bigintFetch } from '@/lib/bigint-fetch';
import { bigintJson } from '@/lib/bigint-json'; // or @{scope}/core

// GET — response body deserialized with bigintJson.parse
const user = await bigintFetch<User>('/api/users/1');

// POST — body serialized with bigintJson.stringify
const order = await bigintFetch<Order>('/api/orders', {
  method: 'POST',
  body: bigintJson.stringify({ productId: 1n, quantity: 2n }),
});
```

### Do / Don't

- **Do** use `bigintFetch()` for all API calls to endpoints that return BigInt fields
- **Do** use `bigintJson.stringify` when building request bodies that contain `bigint`
- **Don't** use native `fetch` + `response.json()` for endpoints that return BigInt fields
- **Don't** cast `bigint` to `Number` — values > 2^53 − 1 lose precision silently
- **Don't** use `.toString()` inline as a workaround — it breaks downstream type contracts
```

> ✅ `todowrite`: `p4` → `completed`

---

## PHASE 5: Generate ARCHITECTURE.md

> ▶ `todowrite`: `p5` → `in_progress`

Create `$ARGUMENTS/ARCHITECTURE.md` with the following content.

```markdown
# {DIR_NAME} — ARCHITECTURE.md

> Describes the structural and technical foundations of this project.
> Read this before starting any implementation work.

## STACK

| Layer | Choice |
|-------|--------|
| Language | TypeScript 5 (`strict`) |
| Bundler | Vite 6 |
| Test framework | Vitest 3 |
| Linter | ESLint 9 + typescript-eslint |
| Module system | ESM (`"type": "module"`) |

## PROJECT STRUCTURE

```
{dir-name}/
├── src/
│   └── index.ts      # Entry point
├── AGENTS.md
├── RULES.md
├── ARCHITECTURE.md
├── package.json
├── tsconfig.json
├── eslint.config.js
└── vitest.config.ts
```

## EXECUTION MODEL {#execution-model}

| Mode | Command | Notes |
|------|---------|-------|
| Type check | `tsc --noEmit` | Type check only — never used as build tool |
| Test | `vitest run` | Unit tests via Vitest |
| Build | `vite build` | Outputs to `dist/`; only on explicit request |

### Prohibited tools
`ts-node`, `tsx`, `esbuild` (direct), `webpack` — do not introduce these.

## DEPENDENCY POLICY

- New **runtime** dependencies require user approval before install (`pnpm add`)
- `@types/*` dev dependencies may be added autonomously
- Prefer built-in Node.js APIs over external packages where feasible
```

> ✅ `todowrite`: `p5` → `completed`

---

## PHASE 6: Save Files

> ▶ `todowrite`: `p6` → `in_progress`

Write all three files using the file write tool (not shell redirection — escape issues may occur).

**Save paths:**
- `$ARGUMENTS/AGENTS.md`
- `$ARGUMENTS/RULES.md`
- `$ARGUMENTS/ARCHITECTURE.md`

After saving, print the saved paths.

> ✅ `todowrite`: `p6` → `completed`

---

## PHASE 6-M: Update root AGENTS.md PROJECT STRUCTURE (monorepo mode only)

> ▶ `todowrite`: `p6m` → `in_progress` (monorepo) or `cancelled` (standalone)

**Skip this phase if MODE=standalone.**

Open the root `AGENTS.md` (found during PHASE 1 monorepo detection).

Locate the `## PROJECT STRUCTURE` fenced code block and find the line for the parent directory (`apps/` or `packages/`). Append the new entry as a child item:

```
│   └── {TARGET_DIR_NAME}/   # TypeScript library
```

If the parent directory line already has child entries, use `├──` for existing entries and `└──` for the new last entry. Save the updated root `AGENTS.md`.

> ✅ `todowrite`: `p6m` → `completed`

---

## PHASE 7: Greenfield Detection

> ▶ `todowrite`: `p7` → `in_progress`

Check whether `$ARGUMENTS/src/` exists.

- **Exists** → abort immediately and report:

```
ERROR: Source files detected at $ARGUMENTS/src/.
  init-ts-empty only initializes greenfield projects.
  AGENTS.md, RULES.md, and ARCHITECTURE.md have been saved.
  To add TypeScript tooling to an existing project, apply config files manually.
```

- **Does not exist** → proceed to **PHASE 8**.

> ✅ `todowrite`: `p7` → `completed`

---

## PHASE 8: Initialization Approval Gate ⚠️ User approval required

> ▶ `todowrite`: `p8` → `in_progress`

Do not create any files or directories without approval.

```
Q. AGENTS.md, RULES.md, and ARCHITECTURE.md have been created.
   Empty project detected (greenfield). Proceed with initialization?
     - Yes, initialize now
       (creates: tsconfig.json, eslint.config.js, vitest.config.ts,
        package.json, src/index.ts, src/index.test.ts)
     - No, use AGENTS.md, RULES.md, and ARCHITECTURE.md only
```

If the user selects **No**, print the completion report and exit.

> ✅ `todowrite`: `p8` → `completed` (Yes) or `cancelled` (No → exit)

---

## PHASE 9: Initialize Project Structure

> ▶ `todowrite`: `p9` → `in_progress`

Create the following files in `$ARGUMENTS/`:

### `package.json`
```json
{
  "name": "{dir-name}",
  "version": "0.1.0",
  "private": true,
  "type": "module",
  "scripts": {
    "build": "vite build",
    "typecheck": "tsc --noEmit",
    "lint": "eslint --fix .",
    "test": "vitest run",
    "test:watch": "vitest",
    "test:coverage": "vitest run --coverage"
  },
  "devDependencies": {
    "typescript": "^5",
    "vite": "^6",
    "vitest": "^3",
    "@vitest/coverage-v8": "^3",
    "eslint": "^9",
    "typescript-eslint": "^8",
    "@types/node": "^22"
  }
}
```

### `tsconfig.json`
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,
    "noImplicitReturns": true,
    "skipLibCheck": true,
    "esModuleInterop": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

### `eslint.config.js`
```js
import eslint from '@eslint/js';
import tseslint from 'typescript-eslint';

export default tseslint.config(
  eslint.configs.recommended,
  ...tseslint.configs.strictTypeChecked,
  {
    languageOptions: {
      parserOptions: {
        projectService: true,
        tsconfigRootDir: import.meta.dirname,
      },
    },
    rules: {
      '@typescript-eslint/no-explicit-any': 'error',
      '@typescript-eslint/explicit-function-return-type': 'warn',
    },
  },
  {
    ignores: ['dist/', 'eslint.config.js'],
  },
);
```

### `vitest.config.ts`
```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: false,
    coverage: {
      provider: 'v8',
      reporter: ['text', 'lcov'],
      thresholds: { lines: 80, functions: 80, branches: 80 },
    },
  },
});
```

### `src/index.ts`
```typescript
// Entry point for {dir-name}
export {};
```

### `src/index.test.ts`
```typescript
import { describe, it, expect } from 'vitest';

describe('{dir-name} smoke test', () => {
  it('entry point exists', () => {
    expect(true).toBe(true);
  });
});
```

> ✅ `todowrite`: `p9` → `completed`

---

## PHASE 10: git init (standalone mode only)

> ▶ `todowrite`: `p10` → `in_progress` (standalone) or `cancelled` (monorepo)

**Skip this phase if MODE=monorepo** — the root repository already covers this package.

If `.git/` already exists in `$ARGUMENTS`, skip and proceed to the next phase.

Otherwise:

```bash
git init
git branch -M main
```

If `git user.name` / `user.email` is not configured, notify the user and record the warning.
The initial commit will be requested in **PHASE 13: Initial Commit** after verification and troubleshooting.

> ✅ `todowrite`: `p10` → `completed`

---

## PHASE 11: Verification Pipeline (auto-run)

> ▶ `todowrite`: `p11` → `in_progress`

Run automatically after initialization. Do not report completion without running verification.

```bash
pnpm install
pnpm tsc --noEmit
pnpm eslint --fix .
pnpm vitest run
```

If a command is unavailable or config is missing, report to the user immediately.

> ✅ `todowrite`: `p11` → `completed`

---

## PHASE 12: Record troubleshooting

> ▶ `todowrite`: `p12` → `in_progress`

Run `/gen-trouble-shoot --source init --label init-ts-empty`

> `/gen-trouble-shoot` will scan this session, extract AI mistakes and incorrect
> implementations that occurred during scaffolding, and write them to `TROUBLE_SHOOT.md`.
> No action is needed here beyond invoking the command.

> ✅ `todowrite`: `p12` → `completed`

---

## PHASE 13: Initial Commit ⚠️

> ▶ `todowrite`: `p13` → `in_progress`

If `.git/` does not exist in the project root, skip this phase entirely.

If `git user.name` / `user.email` was not configured, remind the user and skip.

Present the following via `ask_user_input_v0`:

```
Verification and troubleshooting recording are complete.
Proceed with the initial git commit?

  Commit message:
  "chore: initialize TypeScript project

  - Add AGENTS.md, RULES.md, ARCHITECTURE.md
  - Scaffold empty TypeScript structure"

  - Yes — run git commit now
  - No  — skip commit (leave files untracked)
```

If the user selects **No**, record "commit skipped (user declined)" in the PHASE 14 report.

If the user selects **Yes**:

```bash
git add .
git commit -m "chore: initialize TypeScript project\n\n- Add AGENTS.md, RULES.md, ARCHITECTURE.md\n- Scaffold empty TypeScript structure"
```

> ✅ `todowrite`: `p13` → `completed`

---

## PHASE 14: Completion Report

> ▶ `todowrite`: `p14` → `in_progress`

```
AGENTS.md saved          : $ARGUMENTS/AGENTS.md
RULES.md saved           : $ARGUMENTS/RULES.md
ARCHITECTURE.md saved    : $ARGUMENTS/ARCHITECTURE.md
Project initialized      : empty TypeScript structure
Files created
  Config                 : package.json, tsconfig.json, eslint.config.js, vitest.config.ts
  Source                 : src/index.ts, src/index.test.ts
Verification pipeline
  pnpm install           : {PASS | FAIL — error details}
  tsc --noEmit           : {PASS | FAIL — error details}
  eslint                 : {PASS | FAIL — error details}
  vitest run             : {PASS | FAIL}
```

> ✅ `todowrite`: `p14` → `completed`
