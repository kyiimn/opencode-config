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
3. Rewrite as a **one-line deterministic constraint** using `MUST` / `MUST NOT` / `NEVER` / `ALWAYS` keywords — generic enough to apply to any project of the same type, but specific enough for an AI agent to verify compliance mechanically. Include the triggering lint/tsc rule name if applicable.
4. If multiple entries reduce to the same rule, deduplicate.

### STEP 4 — Read existing RULES.md and compose merged content

**4-1. Read existing RULES.md first.**

```
cat RULES.md 2>/dev/null || echo "__EMPTY__"
```

If the file exists, parse its full structure — headers, sub-headers, and every rule line.
Store this as `existing_rules`.

**4-2. Compose new rules block.**

Group the generalized rules under headers by project type.
The RULES.md file MUST begin with the following mandatory compliance header:

```markdown
<!-- MANDATORY: All AI coding agents MUST read and comply with every rule in this file before generating or modifying code. Violations of these rules have caused production bugs in prior sessions. -->
# CODING RULES

> ⚠️ **MANDATORY COMPLIANCE** — Every rule below was derived from actual implementation failures.
> All AI agents (including delegated sub-agents) MUST treat these rules as hard constraints during code generation, review, and refactoring.
> Violating any rule listed here is equivalent to introducing a known bug.

## NODE.JS BACKEND RULES {#backend-rules}

### Architecture
- ...

### Lint
- ...

### Test
- ...

## FRONTEND RULES {#frontend-rules}

### ...

## SHARED / LIBRARY RULES {#shared-rules}

### ...
```

**4-3. Deduplicate against existing rules.**

For each new rule, compare against `existing_rules`:
- If an existing rule covers the **same constraint** (even if worded differently), skip the new rule.
- If a new rule is a **stricter or more specific** version of an existing rule, replace the existing rule with the new one.
- If no semantic overlap exists, append the new rule under the matching header.

When comparing, match on semantic intent — not on exact string equality.

**4-4. Write rules in AI-agent-optimized language.**

Rules are consumed by LLM coding agents, not humans. Apply these writing guidelines:
- Use deterministic, unambiguous phrasing: `MUST`, `MUST NOT`, `NEVER`, `ALWAYS`.
- Prefer machine-parseable patterns: `[category] CONSTRAINT: <rule>` format.
- Include the triggering lint rule or error name when applicable (e.g. `@typescript-eslint/no-explicit-any`).
- Avoid hedging words like "prefer", "consider", "try to" — state the constraint absolutely.
- Each rule should be self-contained — an agent reading a single rule in isolation must understand the full constraint without needing surrounding context.

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

Write the merged content to the root `RULES.md`.
- If the file already existed, this is a **merge** — existing rules are preserved, new rules are appended under matching headers, duplicates are removed.
- If the file did not exist, create it fresh with the mandatory compliance header.

Verify with:
```
head -5 RULES.md
```
Confirm the mandatory compliance HTML comment is present on line 1.

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
- Use deterministic constraint language: `MUST` / `MUST NOT` / `NEVER` / `ALWAYS` — avoid hedging words like "prefer", "consider", "should".
- Do NOT include the original "What went wrong" or "Detected by" details — only the prevention constraint.
- The RULES.md mandatory compliance header (HTML comment + blockquote) MUST always be present at the top of the file. If the existing file lacks it, prepend it.
- When merging, preserve the existing rule order within each section — append new rules at the end of each sub-header group.
