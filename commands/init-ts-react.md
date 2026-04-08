---
description: >
  Initialize a TypeScript React frontend project.
  Generates AGENTS.md, RULES.md, and ARCHITECTURE.md for a new or existing project.
  Supports monorepo sub-packages. Scaffolds greenfield projects on approval.
---

Initialize a TypeScript React project at `$ARGUMENTS`.

**Default stack:** React 19+ · TypeScript strict · Vite · shadcn/ui · Pretendard font  
**Outputs:** `AGENTS.md` · `RULES.md` · `ARCHITECTURE.md`

---

## PHASE 1: Resolve Scope

Set `DIR=$ARGUMENTS`, `PROJECT_NAME=basename(DIR)`.  
Detect monorepo by searching ancestors for `pnpm-workspace.yaml`.  
Set `MODE=monorepo` or `MODE=standalone`.

---

## PHASE 2: Inherit or Collect CODE STANDARDS

**If MODE=monorepo AND root RULES.md exists:**  
Read `<root>/RULES.md#code-standards`. Inherit as-is. Skip to PHASE 3.

**Otherwise:** Collect via `ask_user_input_v0`:

```
Q-N1. File naming:     kebab-case | camelCase | PascalCase
Q-N2. Class naming:    PascalCase | camelCase
Q-N3. Interface:       PascalCase (no prefix) | I-prefix
Q-N4. Function naming: camelCase + verb required | verb recommended
Q-T1. Type style:      interface-first | type-first | mixed
Q-T2. Fn style:        arrow-first | function-first | mixed
```

---

## PHASE 3: Router Interview

```
Does this project need client-side routing?
  - Yes — file-based TanStack Router
  - No  — no router
```

Set `ROUTER=tanstack | none`.

---

## PHASE 4: Scan Codebase

Scan `DIR` (max 2 levels). Detect file naming patterns, tech stack from `package.json`, package manager.  
Show results. Confirm with user. Skip re-asking scan-confirmed items.

---

## PHASE 5: Write AGENTS.md

Save to `DIR/AGENTS.md`. Write two variants based on MODE.

**If MODE=monorepo:**

```markdown
# {PROJECT_NAME} — AGENTS.md

**Stack:** TypeScript (strict) · React 19+ · Vite · shadcn/ui[· TanStack Router]
**Package Manager:** pnpm
**Docs:** root [`RULES.md`](../../RULES.md) · [`ARCHITECTURE.md`](./ARCHITECTURE.md)

## COMMANDS

| Purpose          | Command                                  |
| ---------------- | ---------------------------------------- |
| Type check       | `pnpm tsc --noEmit`                      |
| Test             | `pnpm vitest run`                        |
| Lint             | `pnpm eslint --fix .`                    |
| Dev              | `pnpm dev`                               |
| Add UI component | `pnpm dlx shadcn@latest add {name}`      |
| Build            | `pnpm build` — **explicit request only** |

## PERMISSIONS

Auto: read files · tsc · eslint · vitest (single file) · `shadcn add`
Approval required: `pnpm add/remove` · git push · delete files · build · `.env`

## GIT

Inherits from root `AGENTS.md`. Conventional Commits. Pass `tsc --noEmit` + lint before PR.

## FILE REFERENCES

Load root `RULES.md` before any task. Load only the needed anchor.

```
@AGENTS.md              # this file
@{root-path}/RULES.md   # root coding rules (required)
@ARCHITECTURE.md        # component structure, data flow, shadcn catalog
```

| Task | Anchor |
|------|--------|
| Write code | `{root}/RULES.md#code-standards` |
| Frontend structure / patterns | `ARCHITECTURE.md` |
| Using the logger | `{root}/RULES.md#logger` |
| DB schema | `prisma/schema.prisma` |
| Env vars | `.env.example` |
```

**If MODE=standalone:**

```markdown
# {PROJECT_NAME} — AGENTS.md

**Stack:** TypeScript (strict) · React 19+ · Vite · shadcn/ui[· TanStack Router]
**Package Manager:** {pnpm | yarn | npm}
**Docs:** [RULES.md](./RULES.md) · [ARCHITECTURE.md](./ARCHITECTURE.md)

## COMMANDS

| Purpose          | Command                                  |
| ---------------- | ---------------------------------------- |
| Type check       | `pnpm tsc --noEmit`                      |
| Test             | `pnpm vitest run`                        |
| Lint             | `pnpm eslint --fix .`                    |
| Dev              | `pnpm dev`                               |
| Add UI component | `pnpm dlx shadcn@latest add {name}`      |
| Build            | `pnpm build` — **explicit request only** |

## PERMISSIONS

Auto: read files · tsc · eslint · vitest (single file) · `shadcn add`
Approval required: `pnpm add/remove` · git push · delete files · build · `.env`

## GIT

Branch: `feature/*` `fix/*` `chore/*`
Commits: Conventional Commits (`feat:` `fix:` `chore:` `docs:` `refactor:` `test:`)
`tsc --noEmit` + lint must pass before PR.

## FILE REFERENCES

```
@AGENTS.md       # this file
@RULES.md        # coding rules (required)
@ARCHITECTURE.md # component structure, data flow, shadcn catalog
```

| Task | Anchor |
|------|--------|
| Write code | `RULES.md#code-standards` |
| Frontend structure / patterns | `ARCHITECTURE.md` |
| Using the logger | `RULES.md#logger` |
| Env vars | `.env.example` |
```

---

## PHASE 6: Write RULES.md

Save to `DIR/RULES.md`.  
In monorepo: save to `<root>/RULES.md` only if it does not exist — never overwrite.

````markdown
# {PROJECT_NAME} — RULES.md

> Load only the section you need. Do not read the whole file.
> Sections: [#code-standards] · [#frontend] · [#logger]

---

## CODE STANDARDS {#code-standards}

**TypeScript:** strict · no `any` (use `unknown` + guard) · explicit return types · minimize `!`

**Naming**
| Target | Rule | Example |
|--------|------|---------|
| File | {FILE_NAMING} | `{example}` |
| Class | {CLASS_NAMING} | `{example}` |
| Interface | {INTERFACE_NAMING} | `{example}` |
| Type alias | PascalCase | `UserDto` |
| Function | {FUNCTION_NAMING} | `{example}` |
| Constant | SCREAMING_SNAKE | `MAX_RETRY` |

Type style: {TYPE_STYLE} · Function style: {FUNCTION_STYLE}

**Bundling:** Vite only · `tsc --noEmit` for type check only · no ts-node / tsx / webpack

Do: functional style · SRP · `async/await` · narrow types
Don't: no `console.log` in prod · no magic strings · no `any` · no unapproved packages

---

## FRONTEND {#frontend}

**Components:** PascalCase names · `type Props = {...}` at file top · one named export per file
**Hooks:** `use` prefix camelCase · no `useEffect` for data fetching
**React 19:** prefer `useActionState`, `useFormStatus`

**shadcn/ui**

- Check shadcn catalog before implementing any UI component
- Add: `pnpm dlx shadcn@latest add {component}`
- Available: Button · Input · Dialog · Card · Table · Form · Select · Tabs · Toast · Badge · Avatar · Dropdown · …
- Never modify shadcn components — wrap instead
- Icons: Lucide React (bundled) · no separate icon library
  ```ts
  import { Search, ChevronDown, X } from "lucide-react";
  ```
````

- Use `cn()` from `@/lib/utils` for conditional classes

**Font (Pretendard)**
`index.css`: `@import url("https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard.min.css");`
`index.css body`: `font-family: 'Pretendard', -apple-system, sans-serif;`

**TanStack Router** _(ROUTER=tanstack only)_

- Routes: files under `src/routes/` · root `__root.tsx` · index `index.tsx` · dynamic `$param.tsx`
- `routeTree.gen.ts` is auto-generated — never edit manually
- Navigate: `<Link>` or `useNavigate()` · no `window.location`

---

## LOGGER {#logger}

The logger lives at `src/lib/logger.ts` (generated by the `ts-logger` skill).
No source scan needed — import paths and level rules are fixed.

### Import

```typescript
// Named logger — recommended for hooks, API clients, and server actions
import { createLogger } from '@/lib/logger';
const logger = createLogger('FeatureName');

// Default logger — for simple one-off files
import { logger } from '@/lib/logger';
```

### Levels

| Level | When to use |
|-------|-------------|
| `debug` | Component state, fetch payloads, tracing — **suppressed in production** |
| `log` | Normal events (route change, form submit, data loaded) |
| `warn` | Unexpected but recoverable situation (stale cache, fallback render) |
| `error` | Fetch failure, unhandled exception, broken invariant |

### Usage

```typescript
logger.debug('Form state', { values, errors });     // dev only
logger.log('Route changed', { to: pathname });
logger.warn('Cache miss', { key });
logger.error('API call failed', error);
```

### Do / Don't

- **Do** use logger in API client functions, server actions, and custom hooks
- **Do** pass structured data as the second argument instead of string interpolation
- **Don't** use `console.log`, `console.warn`, or `console.error` in application code
- **Don't** log sensitive data (tokens, PII, form passwords) at any level
- **Don't** add logger calls inside render functions or JSX — use event handlers instead

````

---

## PHASE 7: Write ARCHITECTURE.md

Save to `DIR/ARCHITECTURE.md`. Use this template:

```markdown
# {PROJECT_NAME} — ARCHITECTURE.md

**Stack:** TypeScript (strict) · React 19+ · Vite · shadcn/ui[· TanStack Router (file-based)]
**Package Manager:** {pnpm | yarn | npm}

## Project Structure

{PROJECT_NAME}/
├── src/
│   ├── components/
│   │   └── ui/           # shadcn/ui generated components
│   ├── routes/           # [tanstack] file-based routes
│   │   ├── __root.tsx
│   │   └── index.tsx
│   ├── pages/            # [no-router]
│   ├── hooks/
│   ├── stores/
│   ├── lib/
│   │   └── utils.ts      # cn() utility
│   └── types/
├── public/
├── AGENTS.md · RULES.md · ARCHITECTURE.md
├── components.json
├── package.json · tsconfig.json · vite.config.ts

## Key Files

| Role | Path |
|------|------|
| App entry | `src/main.tsx` |
| Root layout | `src/routes/__root.tsx` or `src/App.tsx` |
| Route tree (generated) | `src/routeTree.gen.ts` |
| API client | `src/lib/api.ts` |
| State | `src/stores/` |
| Shared types | `src/types/index.ts` |
| shadcn utils | `src/lib/utils.ts` |

## Data Flow

User action → Route / Page → Custom hook → Store or Server Action → API → UI update

## Decisions Log

| Date | Decision | Reason |
|------|----------|--------|
| {date} | Initial setup | /init-ts-react |
````

---

## PHASE 8: Scaffold Project

**Abort condition:** If `DIR/src/` already exists, stop immediately and report:

```
ERROR: Source files detected at DIR/src/.
  init-ts-react only initializes greenfield projects.
  AGENTS.md, RULES.md, and ARCHITECTURE.md have been saved.
  To add these configs to an existing project, apply config files manually.
```

Create the following files under `DIR`:

**`package.json`** — scripts: dev · build · typecheck · lint · test · test:watch  
dependencies: `react ^19` · `react-dom ^19` [+ `@tanstack/react-router ^1` if ROUTER=tanstack]  
devDependencies: `typescript ^5` · `vite ^6` · `@vitejs/plugin-react ^4` · `vitest ^3` · `@vitest/coverage-v8 ^3` · `jsdom ^26` · `@testing-library/react ^16` · `@testing-library/user-event ^14` · `eslint ^9` · `typescript-eslint ^8` · `@types/react ^19` · `@types/react-dom ^19` · `tailwindcss ^4` · `@tailwindcss/vite ^4` · `clsx ^2` · `tailwind-merge ^2` · `lucide-react` [+ `@tanstack/router-plugin ^1` if ROUTER=tanstack]

> Tailwind v4 removes `autoprefixer` and `postcss` as separate deps — bundled via `@tailwindcss/vite`.

**`tsconfig.json`** — target ES2022 · strict · moduleResolution Bundler · paths `"@/*": ["./src/*"]`

**`vite.config.ts`**

- plugins: `@tailwindcss/vite` · `@vitejs/plugin-react` [+ `TanStackRouterVite({ routesDirectory: './src/routes' })` if ROUTER=tanstack]
- resolve alias: `@/ → src/`
- test: environment jsdom · coverage v8 · thresholds 80%

> No `tailwind.config.js` or `postcss.config.js` needed — Tailwind v4 is configured via CSS.

**`components.json`** — shadcn/ui · style default · cssVariables true · aliases `@/components` `@/lib/utils`

**`src/lib/utils.ts`** — `export function cn(...inputs: ClassValue[]) { return twMerge(clsx(inputs)) }`

**`src/index.css`**

```css
@import url("https://cdn.jsdelivr.net/gh/orioncactus/pretendard@v1.3.9/dist/web/static/pretendard.min.css");
@import "tailwindcss";
@layer base {
  body {
    font-family:
      "Pretendard",
      -apple-system,
      sans-serif;
  }
}
```

**`index.html`** — `lang="ko"` · `<div id="root">` · `src/main.tsx` · no extra `<link>` tags

**`src/main.tsx`**

- ROUTER=none: `StrictMode` + `createRoot` + `<App />`
- ROUTER=tanstack: `createRouter({ routeTree })` + `<RouterProvider router={router} />`

**`src/App.tsx`** _(ROUTER=none)_ — `export function App(): React.ReactElement { return <h1>{PROJECT_NAME}</h1> }`

**`src/routes/__root.tsx`** _(ROUTER=tanstack)_ — `createRootRoute` with `<Outlet />`

**`src/routes/index.tsx`** _(ROUTER=tanstack)_ — `createFileRoute('/')` with index component

**`src/App.test.tsx`** — vitest smoke test: render App, assert heading exists

**`eslint.config.js`** — typescript-eslint strictTypeChecked · no-explicit-any error · explicit-function-return-type warn

**`.gitignore`** — `node_modules/` `dist/` `coverage/` `.env` `.env.local`

> `src/lib/logger.ts` 는 `ts-logger` 스킬이 생성합니다.
> 위 파일 생성 직후 `ts-logger` 스킬을 {MODE} 모드로 호출하세요.
> (React 프로젝트에서는 서버 사이드 코드 / API 호출 레이어에서 logger 를 활용합니다.)

---

## PHASE 9: Validate

Run automatically. Do not report completion without running all steps.

```bash
pnpm install
pnpm tsc --noEmit
pnpm eslint --fix .
pnpm vitest run
```

---

## PHASE 10: Report

```
AGENTS.md       → DIR/AGENTS.md ✅
RULES.md        → {path} ✅
ARCHITECTURE.md → DIR/ARCHITECTURE.md ✅
Router          → {TanStack Router (file-based) | none}
Scaffolding     → React 19 + Vite + shadcn/ui ✅

Validation
  install  {PASS|FAIL}
  tsc      {PASS|FAIL}
  eslint   {PASS|FAIL}
  vitest   {PASS|SKIP|FAIL}

Next
  Add shadcn component : pnpm dlx shadcn@latest add {name}
  Implement features   : /implement {task}
```

---

## Guardrails

- Never re-ask scan-confirmed items.
- `PROJECT_NAME = basename(DIR)` — never ask the user.
- Never overwrite existing root `RULES.md` in monorepo.
- Package `AGENTS.md` references root rules by path — no duplication.
- Always run validation before reporting completion.
- Write files with file-write tool only — shell redirection forbidden.
- Never modify shadcn components directly.
- Check shadcn catalog before implementing any UI component.
- Abort if `src/` exists — do not scaffold over existing projects.
