---
description: Initialize a TypeScript pnpm monorepo. generate AGENTS.md / RULES.md → scaffold workspace → git init → verify.
---

Initialize a TypeScript pnpm monorepo: generate AGENTS.md / RULES.md → scaffold workspace → git init → verify.

---

## PHASE 0: Check environment

Check whether `.git/` exists (used in PHASE 4 to decide if `git init` is needed).

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

// Each package's eslint.config.js
import base from '@{scope}/core/eslint/node.js';
export default [...base];
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

| When                 | Section                   |
| -------------------- | ------------------------- |
| Writing/editing code | `RULES.md#code-standards` |
| API development      | `RULES.md#api`            |
| Frontend development | `RULES.md#frontend`       |
| CLI development      | `RULES.md#cli`            |
| Shared library       | `RULES.md#shared`         |
| DB schema            | `prisma/schema.prisma`    |
| Env vars             | `.env.example`            |
````

### 2-2. `./RULES.md`

Fill `{FILE_NAMING}`, `{CLASS_NAMING}`, `{INTERFACE_NAMING}`, `{METHOD_NAMING}`, `{TYPE_DEFINITION_STYLE}`, `{FUNCTION_STYLE}` with PHASE 1 values.

```markdown
# {PROJECT_NAME} — RULES.md

> Load on demand. Use section anchors — do not read the entire file at once.

- [`#code-standards`](#code-standards) — TypeScript · naming · bundling · Do/Don't

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

### Don't

- No `console.log` in production — use a logger
- No magic numbers/strings — extract as constants
- No `// TODO` without a commit
- No new packages without user approval
- No `any`
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
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
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

export default tseslint.config(...tseslint.configs.recommendedTypeChecked, {
  rules: {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/explicit-function-return-type": "warn",
  },
});
```

**10. `packages/core/eslint/react.js`**

```javascript
import base from "./index.js";
export default [...base, { rules: {} }];
```

**11. `packages/core/eslint/node.js`**

```javascript
import base from "./index.js";
export default [...base, { rules: {} }];
```

**12. `packages/core/vitest.config.ts`**

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

**13. `packages/core/src/index.ts`**

```typescript
// Export shared types and utilities from this entry point only.
// Never import across package boundaries via relative paths.
```

**14. `packages/core/src/index.test.ts`**

```typescript
import { describe, it, expect } from "vitest";

describe("packages/core smoke test", () => {
  it("workspace is initialized", () => {
    expect(true).toBe(true);
  });
});
```

**15. `apps/.gitkeep`** — empty file to track directory

**16. `.gitignore`** — skip if already exists

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

If `git user.name` / `user.email` is not set, notify user and pause before committing.

```bash
git add .
git commit -m "chore: initialize pnpm monorepo workspace

- Add pnpm-workspace.yaml with apps/* and packages/* globs
- Add packages/core with tsconfig, eslint, vitest shared configs
- Add AGENTS.md, RULES.md"
```

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

## PHASE 6: Report

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
