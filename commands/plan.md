---
description: Design-only planning workflow. Reads project context files, writes user request snapshot, optionally writes API contract, invokes Prometheus to produce an implementation plan, then Metis to cross-validate. No code is written.
---

You are the orchestrator. Execute the workflow below **in order, without interruption**.
Print `▶ PHASE N start` at the beginning of each step and `✅ PHASE N done` upon completion.
Never skip or reorder steps.

**Progress tracking**: Use the `TodoWrite` tool to manage workflow progress.
Mark each PHASE as `in_progress` when it starts, and `completed` immediately when it finishes.
Do not batch completions — update the todo list as each PHASE concludes.

> **File ownership rule**: `$REPO_ROOT/.sisyphus/plans/{keyword}.md` is owned exclusively by `@prometheus`.
> The orchestrator must never create, edit, or patch this file directly.
> All content inside the plan file is written and maintained by Prometheus.

---

## Pre-flight — Declare keyword

### No-argument entry check

If `$ARGUMENTS` is empty or blank:

```
⚠️  No task description provided.
Please call /plan with a task description to begin.
```

Then **terminate immediately**.

---

Before PHASE 0, determine the **keyword** by the rules below. Use it consistently in every step.

- If the first word of `$ARGUMENTS` is a valid identifier (letters, digits, hyphens only), use it as the keyword.
- Otherwise summarize `$ARGUMENTS` into a ≤3-word lowercase kebab-case slug.
  - e.g. "Add user login feature" → `user-login`
  - e.g. "Implement payment refund API" → `payment-refund`

### Keyword deduplication

After deriving the candidate keyword, check for an existing plan file at `$REPO_ROOT/.sisyphus/plans/{keyword}.md`.

- If the file **does not exist** → use the keyword as-is.
- If the file **exists** → append a numeric suffix, incrementing until a free slot is found:
  - `{keyword}-2`, `{keyword}-3`, … (start at `-2`; never use `-1`)
  - Use the first suffix whose corresponding plan file does not exist.
  - Print:
    ```
    ⚠️  '{original}' already exists. Using '{keyword}' instead.
    ```

All subsequent references to `{keyword}` in this workflow use the final deduplicated value.

---

## Pre-flight — Detect project roots

### Detect REPO_ROOT (repository root)

```bash
REPO_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
```

- If the command **fails** (not a git repo) → set `$REPO_ROOT` to the current working directory.
- If the command **succeeds** → store as `$REPO_ROOT`.

All `.sisyphus/` metadata files (plans, contract, user-request) are anchored to `$REPO_ROOT/.sisyphus/`.

### Detect PROJECT_DIR (sub-project root, not monorepo root)

```bash
PROJECT_DIR=""
dir="$(pwd)"

while [ "$dir" != "/" ]; do
  if [ -f "$dir/package.json" ]; then
    has_workspaces=$(node -e \
      "try{const p=require('$dir/package.json');process.exit(p.workspaces?1:0)}catch(e){process.exit(0)}" \
      2>/dev/null; echo $?)
    if [ "$has_workspaces" = "0" ]; then
      PROJECT_DIR="$dir"
      break
    fi
  fi
  dir=$(dirname "$dir")
done

# Fallback for non-Node projects
if [ -z "$PROJECT_DIR" ]; then
  dir="$(pwd)"
  while [ "$dir" != "/" ]; do
    if [ -f "$dir/pyproject.toml" ] || [ -f "$dir/Cargo.toml" ] || [ -f "$dir/go.mod" ]; then
      PROJECT_DIR="$dir"
      break
    fi
    dir=$(dirname "$dir")
  done
fi

# Final fallback
if [ -z "$PROJECT_DIR" ]; then
  PROJECT_DIR="$(pwd)"
fi
```

Store as `$PROJECT_DIR`. Context files (`AGENTS.md`, `ARCHITECTURE.md`, `TROUBLE_SHOOT.md`, `DESIGN.md`) are read from this path.
`RULES.md` is always read from `$REPO_ROOT` (monorepo root if applicable).

Print:
```
keyword      : {keyword}
REPO_ROOT    : {$REPO_ROOT}
PROJECT_DIR  : {$PROJECT_DIR}
plan         : {$REPO_ROOT}/.sisyphus/plans/{keyword}.md
contract     : {$REPO_ROOT}/.sisyphus/contract/{keyword}.md
user-request : {$REPO_ROOT}/.sisyphus/user-request/{keyword}.md
```

---

## Pre-flight — Initialize todo list

Use `TodoWrite` to register all workflow phases before any PHASE begins:

| id | content | status |
|---|---|---|
| `phase-0` | `PHASE 0: Load project context` | `pending` |
| `phase-1a` | `PHASE 1a: Write API contract` | `pending` |
| `phase-1b` | `PHASE 1b: Write implementation plan (Prometheus)` | `pending` |
| `phase-2` | `PHASE 2: Cross-validate plan and contract (Metis)` | `pending` |
This gives the user visibility into the full plan workflow from the start.

---

## Pre-flight — User request snapshot

Save `$ARGUMENTS` verbatim to:

```
{$REPO_ROOT}/.sisyphus/user-request/{keyword}.md
```

File format:
```markdown
# User Request: {keyword}

{$ARGUMENTS verbatim}
```

This file is the authoritative record of the original user requirement.
Any agent in this workflow may read it.

---

## Requirements

$ARGUMENTS

---

## PHASE 0 — Load project context

Mark `phase-0` as `in_progress` via `TodoWrite`.

▶ PHASE 0 start

### Load AGENTS.md

Read `{$PROJECT_DIR}/AGENTS.md` if it exists.
This file defines the agent roster, delegation rules, and project-specific agent instructions.
Internalize before any agent call.

### Load RULES.md

Read `{$REPO_ROOT}/RULES.md` if it exists.
In a monorepo, this is the monorepo root. In a single-project repo, this is the same as `$PROJECT_DIR`.
This file defines project-wide coding rules and conventions. Enforce without exception in all planning steps.

### Load project context files

Check for these files at `$PROJECT_DIR`. Read each one that exists.

| File | Purpose | Usage |
|---|---|---|
| `ARCHITECTURE.md` | System architecture, layer structure, tech-stack conventions | Use as the basis for planning (Prometheus) |
| `TROUBLE_SHOOT.md` | Past AI mistakes and prevention rules | Apply throughout all steps to avoid repeating errors |
| `DESIGN.md` | UX/UI principles, interaction guide, design conventions | Load only if project is a frontend project (React, Vue, etc.) |

After reading, print:

```
✅ PHASE 0 done
   AGENTS.md        : {found/not found} ($PROJECT_DIR/AGENTS.md)
   RULES.md         : {found/not found} ($REPO_ROOT/RULES.md)
   ARCHITECTURE.md  : {found/not found} ($PROJECT_DIR/ARCHITECTURE.md)
   TROUBLE_SHOOT.md : {found/not found} ($PROJECT_DIR/TROUBLE_SHOOT.md)
   DESIGN.md        : {found/not found} ($PROJECT_DIR/DESIGN.md)
```

If `TROUBLE_SHOOT.md` was found, **immediately extract every rule or open issue from it** and print as a numbered prevention checklist:

```
[!] TROUBLE_SHOOT prevention checklist ({N} items)
   1. {rule or open issue, one line}
   2. ...
```

This checklist is the **active guard rail for the entire workflow**.
It must be passed verbatim to every agent call in PHASE 1b and PHASE 2 as a `[TROUBLE_SHOOT constraints]` block.
If `TROUBLE_SHOOT.md` was not found, the checklist is empty and the injection block is omitted.

Mark `phase-0` as `completed` via `TodoWrite`.

✅ PHASE 0 done

---

## PHASE 1a — Write API contract

Mark `phase-1a` as `in_progress` via `TodoWrite`.

▶ PHASE 1a start

**[Conditional]** This phase only applies to projects that expose HTTP endpoints (e.g. Express API servers).

First, scan `$ARGUMENTS` and `ARCHITECTURE.md` (if loaded) for HTTP indicators:
keywords such as `POST`, `GET`, `PUT`, `DELETE`, `PATCH`, route paths (`/api/...`), REST, endpoint, handler, controller, or Express.

- If **no HTTP indicators are found** → set `$HAS_CONTRACT = false`. Skip this phase entirely.
  Mark `phase-1a` as `completed` via `TodoWrite` with content updated to `PHASE 1a: Write API contract [skipped]`.
  Print: `PHASE 1a skipped — no HTTP endpoints detected.`
  Proceed directly to PHASE 1b.

- If **HTTP indicators are found** → set `$HAS_CONTRACT = true`. Proceed with the steps below.

---

The orchestrator writes the API contract file before spawning any agent.
This step must complete before PHASE 1b begins.

Read `ARCHITECTURE.md` (loaded in PHASE 0) for HTTP conventions, then extract every HTTP endpoint mentioned or implied by `$ARGUMENTS`.

### Deliverable — `{$REPO_ROOT}/.sisyphus/contract/{keyword}.md`

This file is the **single source of truth for all HTTP interfaces** in this task.
It is derived from requirements only — not from implementation decisions.
Any agent in this workflow may read it.

Write one section per endpoint using this format:

```markdown
## {METHOD} {path}

**Request**
- Body / Params:
  ```ts
  { fieldName: type; ... }
  ```

**Success**
- Status: {code}
- Body:
  ```ts
  { ok: true; data: { fieldName: type; ... } }
  ```

**Errors**
| status | code | condition |
|--------|------|-----------|
| {4xx}  | {ERROR_CODE} | {trigger condition} |
```

### Writing rules

- Use the HTTP conventions from `ARCHITECTURE.md` for status codes and envelope shapes.
  If `ARCHITECTURE.md` is not found, default to:
  - Success: `{ ok: true, data: ... }`
  - Error: `{ ok: false, message: string, code?: string }`
- Use TypeScript inline types for all shapes. Do not reference external type files.
- Derive error codes from domain semantics (e.g. `PAYMENT_NOT_FOUND`, `ALREADY_REFUNDED`).
  Do not invent codes not inferable from requirements.
- If a requirement implies an endpoint but does not specify the full shape,
  mark the ambiguous field as `unknown /* clarify */` and note it in a `> ⚠️ Ambiguity:` block beneath the section.

After writing, print:

```
✅ PHASE 1a done
   API contract : {$REPO_ROOT}/.sisyphus/contract/{keyword}.md
   Endpoints    : {list of METHOD /path}
   Ambiguities  : {count or "none"}
```

Mark `phase-1a` as `completed` via `TodoWrite`.

✅ PHASE 1a done

---

## PHASE 1b — Write implementation plan (Prometheus)

Mark `phase-1b` as `in_progress` via `TodoWrite`.

▶ PHASE 1b start

**Agent dispatch**: Invoke the `@prometheus` subagent via the Task tool.
Do NOT substitute with the Plan agent, Build agent, or any other agent.
Do NOT perform this step yourself as the orchestrator.
The Task tool call MUST name `@prometheus` as the target agent.

Prompt for `@prometheus`:

> Write an **implementation plan only** for the requirements below.
> **Do NOT write any code or tests at this step.**
>
> Before planning, read and internalize the following context files:
> - `{$PROJECT_DIR}/AGENTS.md` — agent roster and project-specific agent instructions.
> - `{$REPO_ROOT}/RULES.md` — follow coding rules and conventions without exception.
> - `{$PROJECT_DIR}/ARCHITECTURE.md` — use layer structure and tech-stack conventions as design baseline.
> - `{$PROJECT_DIR}/TROUBLE_SHOOT.md` — apply prevention rules to avoid past mistakes.
> - `{$PROJECT_DIR}/DESIGN.md` — apply UX/UI principles to any frontend-related output (if present).
>
> **[TROUBLE_SHOOT constraints]**
> {Insert the numbered prevention checklist extracted in PHASE 0, or omit this block if the list is empty.}
> Before writing a single line of the plan, go through each item above and explicitly confirm your approach does not violate it.
> For every item, add a corresponding guard note inside the plan section it applies to (e.g. `<!-- TS-3: validated -->`).
> If an item conflicts with the current requirements, stop and surface the conflict to the orchestrator before proceeding.
>
> **Also read `{$REPO_ROOT}/.sisyphus/contract/{keyword}.md` if it exists.**
> The contract file defines the agreed HTTP interface for this task.
> Do not contradict or extend it — plan exactly what it specifies.
> If the contract contains `⚠️ Ambiguity` notes, resolve them conservatively and document the decision.
>
> **[Inline contract rule]**
> When the contract file exists, every implementation step that touches an HTTP endpoint
> MUST include an `### Expected Contract` subsection containing:
> - Method, path, success status code, success body shape (TypeScript inline type)
> - Error table (status, code, condition) — copied verbatim from the contract
>
> This makes each step self-contained so the implementing agent does not need to
> cross-reference the contract file separately.
> The contract file remains the single source of truth — if it is updated,
> the plan's inline copies must be updated to match.
>
> **Also read `{$REPO_ROOT}/.sisyphus/user-request/{keyword}.md`** for the verbatim original requirement.
>
> ### Deliverable — `{$REPO_ROOT}/.sisyphus/plans/{keyword}.md`
>
> You are the sole owner of this file. Write and maintain it yourself.
> The orchestrator will never modify this file.
>
> Write the detailed implementation plan:
> - Write detailed steps including function names, major logic flows, and dependency relationships.
> - Use Librarian/Explore to find reusable utilities in the codebase first; document the reuse plan.
> - **Do NOT write any code (implementation or test) at this step.**
> - Include a **[File Scope]** section in this format — it is the boundary for all subsequent implementation steps:
>
>   ```markdown
>   ## File Scope
>
>   ### New files
>   - {path to create}
>
>   ### Allowed modifications
>   - {existing file path} (reason)
>
>   ### Test output path
>   - {test directory path, e.g. apps/api/src/test/scenario/}
>
>   ### Out-of-scope (never modify)
>   - All existing files not listed above
>   ```
>
> Requirements:
> {$ARGUMENTS verbatim}

Mark `phase-1b` as `completed` via `TodoWrite`.

✅ PHASE 1b done

---

## PHASE 2 — Cross-validate plan and contract (Metis)

Mark `phase-2` as `in_progress` via `TodoWrite`.

▶ PHASE 2 start

**[Join point]** Start only after `{$REPO_ROOT}/.sisyphus/plans/{keyword}.md` exists.
If `$HAS_CONTRACT = true`, also confirm `{$REPO_ROOT}/.sisyphus/contract/{keyword}.md` exists.

**Agent dispatch**: Invoke the `@metis` subagent via the Task tool.
Do NOT substitute with the Plan agent, Build agent, or any other agent.
Do NOT perform this step yourself as the orchestrator.
The Task tool call MUST name `@metis` as the target agent.

Prompt for `@metis`:

> Read `{$REPO_ROOT}/.sisyphus/user-request/{keyword}.md` for the original requirement.
> Read `{$REPO_ROOT}/.sisyphus/plans/{keyword}.md` for the implementation plan.
> If it exists, read `{$REPO_ROOT}/.sisyphus/contract/{keyword}.md` for the API contract.
>
> Also re-read the following context files as a review baseline:
> - `{$PROJECT_DIR}/ARCHITECTURE.md` (if present)
> - `{$REPO_ROOT}/RULES.md` (if present)
> - `{$PROJECT_DIR}/DESIGN.md` (if present, frontend projects only)
>
> **[TROUBLE_SHOOT constraints]**
> {Insert the numbered prevention checklist extracted in PHASE 0, or omit this block if the list is empty.}
> Verify the plan does not violate any item above.
>
> Strictly review all items below:
>
> #### [Contract review] — skip if no contract exists
> 0. **Contract↔plan consistency**: Does the plan implement every endpoint in the contract exactly?
>    Flag any status code, field name, or error code the plan would contradict or omit.
> 0-1. **Inline contract completeness**: Does every implementation step that touches an endpoint
>    include an `### Expected Contract` subsection? Flag any step missing it or deviating from the contract file.
>
> #### [Plan review]
> 1. **Missing requirements**: Any user requirement not reflected in the plan?
> 2. **Error handling gaps**: Missing handling for error cases, boundary values, or abnormal inputs?
> 3. **Security vulnerabilities**: Auth, input validation, or data exposure gaps?
> 4. **Architecture gaps**: Missing dependencies, wrong layer responsibility, or scalability issues?
> 5. **RULES.md violations**: Any part of the plan that conflicts with the project coding rules?
>
> #### How to report issues
>
> Report issues to the **orchestrator** as a list. Do not call any agent directly.
> The orchestrator re-invokes `@prometheus` via the Task tool with your full report, and `@prometheus` updates the plan file.
> Do NOT invoke `@prometheus` yourself from within this agent call.
>
> After each fix, re-review all applicable items.
> **Declare "Cross-validation complete" only when all items pass.**
>
> #### Escape condition
>
> The orchestrator tracks `@prometheus` re-call count.
> **If it exceeds 3**, stop the review loop and escalate to the user via `ask_user_input_v0`:
>
> ```
> ⚠️  Cross-validation loop escape — user intervention required
> ─────────────────────────────────────
> Iterations   : exceeded 3
> Failed items : (item numbers and descriptions)
> Last fix by  : Prometheus
> ─────────────────────────────────────
> ```
>
> - **Question**: "Validation loop did not converge. How would you like to proceed?"
> - **Options**: `Accept current plan` / `Abort`
> - If accept: report the open risks to the orchestrator; Prometheus will record them in the plan file.
> - If abort: terminate the workflow.

When `@metis` declares "Cross-validation complete" (or the user accepts open risks),
re-invoke `@prometheus` via the Task tool with the full Metis report and instruct it to
finalize `{$REPO_ROOT}/.sisyphus/plans/{keyword}.md` with any revisions, and append
accepted open risks under an `## Open Risks` section if applicable.
The finalized plan file is the primary deliverable of this workflow.

Mark `phase-2` as `completed` via `TodoWrite`.

✅ PHASE 2 done

---

## Final report

Print the final report and terminate the workflow.
Each block below must be printed as a separate output with a blank line between them.

```
✅ /plan workflow complete
```

```
keyword      : {keyword}
REPO_ROOT    : {$REPO_ROOT}
PROJECT_DIR  : {$PROJECT_DIR}
```

```
user request : {$REPO_ROOT}/.sisyphus/user-request/{keyword}.md
API contract : {$REPO_ROOT}/.sisyphus/contract/{keyword}.md  (n/a if no HTTP endpoints)
plan         : {$REPO_ROOT}/.sisyphus/plans/{keyword}.md
```

```
Context loaded:
  AGENTS.md        : {found / not found}
  RULES.md         : {found / not found}
  ARCHITECTURE.md  : {found / not found}
  TROUBLE_SHOOT.md : {found / not found}
  DESIGN.md        : {found / not found}
```

```
Metis validation : {complete / accepted with open risks / aborted}
```

```
─────────────────────────────────────
Design complete. Next steps:

  Implement         /start-work {keyword}
  Test scenarios    /implement-test-scenario {keyword}
  Refactor          /refactor {keyword}
  Troubleshooting   /gen-trouble-shoot {keyword}
  Endpoint docs     /gen-endpoint-docs {keyword}   (API 프로젝트인 경우)
─────────────────────────────────────
```
