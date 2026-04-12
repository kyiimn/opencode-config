---
description: Consolidate TROUBLE_SHOOT.md entries into generalized RULES.md prevention rules
agent: build
---

## Objective

Find all `TROUBLE_SHOOT.md` files in the project, generalize their resolved entries into reusable prevention rules, append to the root `RULES.md`, then delete the source files.

## Workflow

Use `todo_write` to track each step below.

### STEP 1 — Discover TROUBLE_SHOOT.md files

Run:
```
find . -name 'TROUBLE_SHOOT.md' -not -path '*/node_modules/*' -not -path '*/.git/*'
```

Record every path found. If none exist, inform the user and stop.

### STEP 2 — Classify each file's project type

For each `TROUBLE_SHOOT.md` path, inspect the **nearest** `package.json` (same directory or closest ancestor).

Classification rules:
- **backend** — has `express`, `fastify`, `hono`, `koa`, `nestjs`, `@types/node` as dependency, OR has `prisma`/`drizzle`/`typeorm`, OR has a `tsconfig.json` with `"module": "commonjs"/"nodenext"`, OR the directory name suggests `api`/`server`/`cli`/`worker`.
- **frontend** — has `react`, `vue`, `svelte`, `next`, `nuxt`, `vite`, `@angular/core`, OR has `tailwindcss`/`postcss` as dependency, OR has an `index.html` entry point.
- **shared** — libraries, DTOs, core packages that are neither purely backend nor frontend (e.g. `packages/dto`, `packages/core`).
- **monorepo-root** — the file is at the workspace root (same level as `pnpm-workspace.yaml` or root `package.json` with `"workspaces"`).

### STEP 3 — Extract and generalize resolved entries

For each `TROUBLE_SHOOT.md`, read the file and process only `### Resolved` entries.
Skip `### Unresolved` sections entirely.

For every resolved entry:
1. Strip the date, feature name, and project-specific identifiers (file names, class names, variable names, package names like `@chkvec/*`).
2. Keep the **category tag** (e.g. `[Architecture]`, `[Lint]`, `[Test]`, `[Routing]`, `[Config]`, `[Import]`, `[Type]`, `[Error]`, `[Scope]`, `[JSDoc]`, `[Code-smell]`).
3. Rewrite as a **one-line imperative prevention rule** — generic enough to apply to any project of the same type, but specific enough to be actionable.
4. If multiple entries reduce to the same rule, deduplicate.

### STEP 4 — Compose RULES.md content

Group the generalized rules under headers by project type:

```markdown
## Node.js Backend Rules

### Architecture
- ...

### Lint
- ...

### Test
- ...

## Frontend Rules

### ...

## Shared / Library Rules

### ...
```

If the root `RULES.md` already exists, read it first. Merge new rules under matching headers; do NOT duplicate rules that already exist.

### STEP 5 — Present to user for approval

**Before writing anything**, output the full proposed RULES.md content to the conversation so the user can review it.

Then use `ask_user` to ask:

> The above rules will be appended to RULES.md. Proceed?
> - Yes, write as-is
> - Let me edit first (I'll provide corrections)
> - Cancel

If the user says "Let me edit first", wait for their corrections, re-display the updated content, and ask again.
If the user says "Cancel", stop without modifying any files.

### STEP 6 — Write RULES.md

Write (or merge into) the root `RULES.md` with the approved content.
Verify with:
```
cat RULES.md
```

### STEP 7 — Delete processed TROUBLE_SHOOT.md files

Delete every `TROUBLE_SHOOT.md` file that was successfully processed in STEP 3.

```
rm <path>
```

For each deletion, confirm the file no longer exists:
```
test -f <path> && echo "STILL EXISTS" || echo "DELETED"
```

Report a summary of files deleted.

## Important constraints

- Never invent rules that are not grounded in an actual TROUBLE_SHOOT.md entry.
- Preserve the category tags verbatim — they are used for cross-referencing.
- Rules must be written in English.
- Use short imperative phrasing: "Do X" / "Never do Y" / "Always verify Z".
- Do NOT include the original "What went wrong" or "Detected by" details — only the prevention rule.
