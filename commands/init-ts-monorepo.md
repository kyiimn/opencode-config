---
description: Initialize a TypeScript pnpm monorepo. generate AGENTS.md / RULES.md → scaffold workspace → git init → verify.
---

Initialize a TypeScript pnpm monorepo: generate AGENTS.md / RULES.md → scaffold workspace → git init → verify.

---

## PHASE 0: Check environment

Check whether `.git/` exists (used in PHASE 4 to decide if `git init` is needed).

**Abort condition:** If `apps/` or `packages/` already contain subdirectories (other than `packages/core`), stop immediately and report:

```
ERROR: Non-empty workspace detected.
  Found existing packages in apps/ or packages/.
  init-ts-monorepo only initializes greenfield monorepos.
  Use init-ts-api / init-ts-cli / init-ts-react to add packages to an existing monorepo.
```

---

## PHASE 1: Collect conventions

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

## PHASE 2: Write AGENTS.md and RULES.md

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
├── apps/               # Deployable apps (each has its own AGENTS.md)
├── packages/
│   └── core/           # Shared configs and libraries
│       ├── tsconfig/
│       │   ├── base.json        # Base tsconfig extended by all packages
│       │   ├── app.json         # Node/Bun apps
│       │   └── react-lib.json   # React packages
│       ├── eslint/
│       │   ├── index.js         # Base ESLint flat config
│       │   ├── react.js         # React extension
│       │   └── node.js          # Node/API extension
│       └── src/index.ts         # Shared types/utils entry point
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

Load `RULES.md` before running `/implement`. Never start without it.

```
@AGENTS.md   # this file
@RULES.md    # coding rules (required)
```

Load only the relevant section anchor:

| When                 | Section                                  |
| -------------------- | ---------------------------------------- |
| Writing/editing code | `RULES.md#code-standards`                |
| Using the logger     | `RULES.md#logger`                        |
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
```

### 2-3. Save paths

```
./AGENTS.md
./RULES.md
```

Report saved paths to the user.

---

## PHASE 3: Scaffold workspace files

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

> Core 패키지 자체의 `tsc --noEmit` 및 ESLint `projectService` 를 위해 필요합니다.

```jsonc
{
  "extends": "./tsconfig/base.json",
  "include": ["src/**/*.ts"]
}
```

**13. `packages/core/eslint.config.js`**

> Core 패키지는 자기 자신(`@{scope}/core`)을 self-reference 할 수 없으므로 반드시 상대 경로를 사용합니다.

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

> `packages/core/src/lib/logger.ts` 는 `ts-logger` 스킬이 생성합니다.
> PHASE 3 파일 생성 직후 `ts-logger` 스킬을 monorepo 모드로 호출하세요.

**16. `packages/core/src/index.test.ts`**

```typescript
import { describe, it, expect } from "vitest";

describe("packages/core smoke test", () => {
  it("workspace is initialized", () => {
    expect(true).toBe(true);
  });
});
```

**17. `apps/.gitkeep`** — empty file to track directory

**18. `.gitignore`** — skip if already exists

```
node_modules/
dist/
.env
.env.local
*.tsbuildinfo
coverage/
```

---

## PHASE 4: git init

Skip if `.git/` already exists. Otherwise:

```bash
git init
git branch -M main
```

If `git user.name` / `user.email` is not set, notify user and record the warning.
The initial commit will be requested in **PHASE 7: Initial Commit** after verification and troubleshooting.

---

## PHASE 5: Verify

Run in order. Do not report completion without running this.

```bash
pnpm install
pnpm --filter @{scope}/core tsc --noEmit
pnpm --filter @{scope}/core lint
pnpm --filter @{scope}/core test
```

Report immediately if any command is missing or fails.

---

## PHASE 6: Record troubleshooting

Call `@document-writer` with:

> Extract AI mistakes and incorrect implementations from this entire workflow and record them in `TROUBLE_SHOOT.md` at the project root.
>
> **What to extract** — record only these types (exclude normal design decisions or user requirement changes):
> - Code incorrectly implemented by AI that required fixes
> - Repeated error patterns in the self-correction loop
> - Gaps or errors flagged by Metis or Momus
> - Assertion translation errors found in spec↔test validation
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

## PHASE 7: Initial Commit ⚠️

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
- Add AGENTS.md, RULES.md"
```

---

## PHASE 8: Report

```
AGENTS.md / RULES.md  : ./AGENTS.md, ./RULES.md ✅

Workspace files
  root          : package.json, pnpm-workspace.yaml, tsconfig.json, .npmrc, .gitignore
  packages/core : tsconfig/{base,app,react-lib}.json
                  eslint/{index,react,node}.js
                  vitest.config.ts, src/index.ts, src/index.test.ts
  apps/         : .gitkeep

git
  git init      : {done | skipped (already exists)} ✅
  initial commit: done ✅

Verification
  pnpm install  : {PASS | FAIL}
  tsc --noEmit  : {PASS | FAIL}
  lint          : {PASS | FAIL}
  vitest run    : {PASS | FAIL}
```

---

## Constraints

- Write files with `write_file` only — no `echo` or heredoc.
- Only commit if `git user.name` and `user.email` are configured.
- Stop immediately and report if `pnpm install` fails.
