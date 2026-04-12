---
description: Records AI mistakes, wrong approaches, and recurring problems from the current session and all sub-sessions into TROUBLE_SHOOT.md at the correct project root. Can be called after /implement-test-scenario, /start-work, or any coding session. Automatically detects the sub-project root in monorepos to prevent writing to the wrong directory.
---

You are the orchestrator. Execute the steps below in order.

---

## Pre-flight — Detect project root ($PROJECT_DIR)

This command must write to the **sub-project root**, not the monorepo root.

Run the following detection script:

```bash
# Start from the current working directory and walk up.
# Find the nearest directory that contains a package.json WITHOUT a "workspaces" field.
# That directory is the sub-project root.
# If every package.json found has "workspaces" (i.e. we are at the monorepo root with no sub-project),
# use the current working directory as the fallback.

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

# Fallback: if no sub-project found, try pyproject.toml / Cargo.toml / go.mod
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

# Final fallback: use cwd
if [ -z "$PROJECT_DIR" ]; then
  PROJECT_DIR="$(pwd)"
fi
```

Print: `$PROJECT_DIR = {$PROJECT_DIR}`

> **Monorepo check**: If `$PROJECT_DIR` matches the monorepo root (i.e. its `package.json` has a `workspaces` field), print a warning:
> ```
> ⚠️  PROJECT_DIR resolved to monorepo root: {$PROJECT_DIR}
>     TROUBLE_SHOOT.md will be written here. If this is wrong,
>     run this command from inside the target sub-project directory.
> ```

---

## Pre-flight — Resolve context source

Determine where to read the session's work history from.

### If `$ARGUMENTS` is present

Parse `$ARGUMENTS`:

- **`--project <path>`** — override `$PROJECT_DIR` with the given path.
- **`--label <text>`** — use as the entry title instead of auto-generating one.
- **`--source test`** — explicitly treat the current session as an `/implement-test-scenario` session.
- **`--source start-work`** — explicitly treat the current session as a `/start-work` session.
- **`--source init`** — explicitly treat the current session as a project initialization session (e.g. `init-ts-api`, `init-ts-cli`, `init-ts-empty`, `init-ts-monorepo`, `init-ts-react`).
- Any bare word or phrase without a flag → use as the entry label.

Multiple flags may be combined. Unknown arguments are ignored.

If `--project` was given, override `$PROJECT_DIR` with the supplied path and skip the detection script above.

### If `$ARGUMENTS` is empty

Try the following sources in order, stopping at the first that yields usable information:

1. **boulder.json** — Read `$REPO_ROOT/.y/boulder.json` if it exists.
   If found, set:
   - `$CONTEXT_SOURCE = start-work`
   - `$ENTRY_LABEL = {plan_name}` (from `plan_name` field)
   - Override `$PROJECT_DIR` with `project_root` field if present.

2. **Current session history** — Scan the current conversation and all sub-sessions (agent outputs, `_workspace/` files, delegated task logs) for evidence of:
   - `/implement-test-scenario` invocation → `$CONTEXT_SOURCE = test`
   - `/start-work` invocation → `$CONTEXT_SOURCE = start-work`
   - `/init-ts-api`, `/init-ts-cli`, `/init-ts-empty`, `/init-ts-monorepo`, or `/init-ts-react` invocation → `$CONTEXT_SOURCE = init`
   - Any other coding/development work → `$CONTEXT_SOURCE = session`

3. **Fallback** — If nothing is determinable:
   - `$CONTEXT_SOURCE = session`
   - `$ENTRY_LABEL` = `{YYYY-MM-DD} general`

Print:
```
Context source : {test | start-work | init | session}
Entry label    : {$ENTRY_LABEL}
Target file    : {$PROJECT_DIR}/TROUBLE_SHOOT.md
```

---

## Extract — Gather mistakes from the session

Scan the current session **and all sub-sessions** (delegated agent outputs, `_workspace/` message files, named agent logs such as Prometheus / Metis / Sisyphus / Atlas / Momus / Oracle, and any other task-level artifacts produced during the session).

Based on `$CONTEXT_SOURCE`, extract the following:

### When `$CONTEXT_SOURCE = start-work`

Scan the session for:
- Repeated test failure patterns (retry count ≥ 2 for any TC)
- Assertion translation errors found during spec↔test validation
- Escaped TCs (unresolved after 3 retries)
- Scenario spec quality issues flagged by Metis (items 5–10), even if fixed
- Any sub-task that required more than 2 attempts

Exclude: normal design decisions, user-initiated requirement changes.

### When `$CONTEXT_SOURCE = init`

Scan the session for mistakes that occurred during project scaffolding (any `init-ts-*` command):
- Files generated with incorrect content that required manual correction or re-generation
- Wrong package versions installed (e.g. mismatched peer deps, incorrect major version)
- Incorrect configuration generated (tsconfig, eslint.config.js, package.json scripts, etc.)
- Phases that had to be re-executed due to AI error (not user requirement change)
- Repeated self-correction loops (same type of fix applied more than once)
- Gaps or errors flagged by the verification pipeline (lint, type-check, build failures caused by wrong scaffolding)
- Out-of-scope files created or modified by AI during scaffolding

Exclude: user-initiated requirement changes, Q&A design decisions made during the questionnaire phase.

### When `$CONTEXT_SOURCE = start-work`

Scan the session and all sub-sessions for:
- Code incorrectly implemented by any agent that required fixes (including partial rewrites)
- Cases where an agent chose the wrong architecture or approach and had to backtrack
- Gaps or errors flagged by Metis (contract↔plan or plan review), even if fixed
- Any requirement misunderstanding that led to rework
- Any sub-task (across any agent) that required more than 2 attempts for any reason
- Out-of-scope file modification requests during implementation or refactoring
- Tasks that failed and required re-attempts
- AI misunderstandings of the user's intent that led to rework
- Patterns of error (e.g. wrong file edited, wrong API used repeatedly)
- Any explicit corrections the user had to make

Exclude: normal iterative design discussion, refinement, and user-initiated requirement changes.

### When `$CONTEXT_SOURCE = session`

Scan the full session and all sub-sessions for:
- Any mistake the AI made that required correction
- Any approach that was tried and abandoned
- Repeated errors of the same type
- Explicit user corrections

Exclude: normal back-and-forth and design decisions.

---

## Write — Append entry to TROUBLE_SHOOT.md

**If no mistakes were found** across all categories:

Print:
```
✅ No mistakes found in this session. TROUBLE_SHOOT.md was not modified.
```
Terminate.

**If mistakes were found**, write one dated entry to `{$PROJECT_DIR}/TROUBLE_SHOOT.md`.

#### Entry format

```markdown
## [YYYY-MM-DD] {$ENTRY_LABEL}

### Resolved
- **[Rule]** {one-line prevention rule}
  - What went wrong: {brief description}
  - Detected by: {Metis | Momus | Test failure | User correction | Self-correction}

### Unresolved
- **[Open]** {description of the issue}
  - Last error: {one-line summary}
  - Attempted: {what was tried}
```

Rules:
- **Resolved** = mistake occurred but was fixed before the session ended.
- **Unresolved** = mistake or failure was not resolved (e.g. escaped TC, open issue).
- If Resolved has nothing to record, write `(none)`. Do not omit the header.
- If Unresolved has nothing to record, write `(none)`. Do not omit the header.
- Keep each rule to one line — details are already in memory. Do not duplicate verbose descriptions.

#### File handling

- If `{$PROJECT_DIR}/TROUBLE_SHOOT.md` already exists → keep all existing content and **prepend** the new entry at the top.
- If it does not exist → create the file with the new entry.

---

## Final report

Print and terminate:

```
✅ /gen-trouble-shoot complete

target file    : {$PROJECT_DIR}/TROUBLE_SHOOT.md
context source : {test | start-work | init | session}
entry label    : {$ENTRY_LABEL}
resolved items : {N}
unresolved     : {N}
action         : {created | updated | no changes}
```
