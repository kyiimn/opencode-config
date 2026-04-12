---
description: Standalone test scenario pipeline for oh-my-opencode. Writes SDV-compliant test scenarios (Oracle), validates spec quality (Metis items 5–10), generates test code, validates spec↔test translation (Momus), runs the full test suite with self-correction, and delegates AI mistake recording to /gen-trouble-shoot. Can be called standalone or invoked automatically by /implement.
---

You are the orchestrator. Execute the workflow below **in order, without interruption**.
Print `▶ PHASE {letter} start` at the beginning of each phase and `✅ PHASE {letter} done` upon completion.
Never skip or reorder phases unless the resume state shows they are already complete.

---

## Pre-flight — Resolve keyword

### Argument check

**If `$ARGUMENTS` is present:**
- Use the first word as the keyword.
- Proceed to "Load context files" below.

**If `$ARGUMENTS` is empty:**
- Read `.y/boulder.json`.
  - If it exists → extract `plan_name` and use it as the keyword. Extract `user_request_file`, `contract_file`, `project_root`, `run_tests` from the file.
    Print:
    ```
    🔁 No arguments given. Using active task from boulder.json: {plan_name}
    ```
  - If it does not exist → print:
    ```
    ⚠️  No keyword provided and no active task found in .y/boulder.json.
    Please call /implement-test-scenario with a keyword to begin.
    ```
    **Terminate immediately.**

Print:
```
keyword          : {keyword}
user request     : .y/user-request/{keyword}.md
contract         : .y/contracts/{keyword}.md
test scenarios   : .y/tests/{keyword}.md
test checklist   : .y/tests/{keyword}.checklist.md
```

---

## Pre-flight — Detect project root

Detect `$PROJECT_DIR` — the root of the current sub-project.
This ensures `TROUBLE_SHOOT.md` is read from and written to the sub-project, not the monorepo root.

**If `project_root` was read from `boulder.json`**, use that value directly:
```
$PROJECT_DIR = {boulder.project_root}
```

**Otherwise**, detect it from the current working directory:

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

Store as `$PROJECT_DIR`. `TROUBLE_SHOOT.md` is read from this path to build the prevention checklist. Writing to `TROUBLE_SHOOT.md` is delegated to `/gen-trouble-shoot` (PHASE F).

Print: `$PROJECT_DIR = {$PROJECT_DIR}`

---

## Pre-flight — Load context files

Read the following files if they exist:

| File | Required | Purpose |
|---|---|---|
| `.y/user-request/{keyword}.md` | **Yes** | Verbatim original user requirement. Oracle reads this as the sole requirements source. |
| `.y/contracts/{keyword}.md` | No | HTTP interface contract. Present only for API projects. Oracle and all test agents may read this if it exists. |
| `$PROJECT_DIR/TROUBLE_SHOOT.md` | No | Past AI mistakes and prevention rules. Extract into prevention checklist (see below). |
| `$PROJECT_DIR/RULES.md` | No | Project coding rules. Apply to all test code written in PHASE C. |

**[Abort condition]** If `.y/user-request/{keyword}.md` does not exist:
```
⚠️  .y/user-request/{keyword}.md not found.
    Cannot determine requirements. Terminating.
```
**Terminate immediately.**

If `TROUBLE_SHOOT.md` was found, extract every rule or open issue and print as a numbered prevention checklist:
```
[!] TROUBLE_SHOOT prevention checklist ({N} items)
   1. {rule or open issue, one line}
   2. ...
```
This checklist is injected into every agent call in this workflow as a `[TROUBLE_SHOOT constraints]` block.
If `TROUBLE_SHOOT.md` was not found, the checklist is empty and the injection block is omitted.

---

## Pre-flight — Resume state

Read `.y/tests/{keyword}.checklist.md` if it exists.

```markdown
<!-- format of .y/tests/{keyword}.checklist.md -->
- [ ] PHASE A: Write test scenarios
- [ ] PHASE B: Validate scenario spec
- [ ] PHASE C: Write test code
- [ ] PHASE D: Validate spec↔test translation
- [ ] PHASE E: Run tests and self-correction loop
- [ ] PHASE F: Record troubleshooting
```

- If the file does not exist → create it with all boxes unchecked. This is a **new session**.
- If the file exists → read the checkbox states. **Skip any phase already marked `[x]`** and resume from the first unchecked phase.
  Print:
  ```
  🔁 Resume state detected
     Completed: {list of done phases or "none"}
     Resuming from: PHASE {letter}
  ```

---

## PHASE A — Write test scenarios (Oracle)

**[Skip if already done]** If PHASE A is `[x]` in the checklist, skip to PHASE B.

Call `@oracle` with:

> Read the requirements from `.y/user-request/{keyword}.md` and write a functional test scenario spec.
>
> **⚠️ Do NOT read `.y/plans/` or any implementation plan files.**
> This spec must be based on requirements only. Reading the plan contaminates SDV independence.
>
> ※ **Isolation scope**: This is a filesystem-level isolation rule. Oracle must honor the file access prohibition and reason only from the requirements text and the contract file.
>
> [permitted] `.y/contracts/{keyword}.md` may be read.
> This file contains the agreed HTTP interface derived from requirements — it is not an implementation plan.
> If it exists, read it before writing scenarios.
> Use the exact status codes, response field names, and error codes defined there.
> Do not invent or assume interface details not present in the contract or requirements.
>
> **[TROUBLE_SHOOT constraints]**
> {Insert the numbered prevention checklist, or omit this block if the list is empty.}
>
> ### Deliverable — `.y/tests/{keyword}.md`
>
> This file is the **sole and absolute reference** for all validation. Follow every rule below without exception.
>
> #### Writing principles
>
> **[Rule 1] Express every expected result as an observable pseudo-assertion.**
> Never use prose for expected results. Use only:
>
> ```
> assert <target> <operator> <expected>
> ```
>
> Allowed examples:
> ```
> assert response.status === 200
> assert response.body.userId is string
> assert response.body.token.length > 0
> assert db.users.count() === (before.count + 1)
> assert error.code === 'INVALID_INPUT'
> assert result is null
> assert called(sendEmail) === true
> assert called(sendEmail).times === 1
> assert elapsedTime < 500  // ms
> assert state(db.user[1]) contains { active: true }
> assert state(queue.jobs) length === 3
> assert not called(externalApi)
> ```
>
> `called()` = whether a function/API/event was invoked. `state()` = final state of a side-effect target (DB, cache, queue, etc.).
>
> Forbidden (prose — never use):
> ```
> - User is created successfully       ← forbidden
> - An appropriate error is returned   ← forbidden
> - Token is issued correctly          ← forbidden
> ```
>
> **[Rule 2] No ambiguous adjectives.** Ban "appropriate", "correct", "normal", "sufficient", "fast" inside assertions.
>
> **[Rule 3] Each scenario must be independently executable.** No preconditions that depend on another scenario's result.
>
> **[Rule 4] Separate success and failure cases into distinct scenarios.** Never mix success and failure in one scenario.
>
> #### Scenario format
>
> ```markdown
> ### TC-{NNN}: {title}
>
> - **Type**: Happy Path | Edge Case | Error Case | Security
> - **Goal**: {one sentence describing what this scenario verifies}
> - **Preconditions**:
>   - {independently configurable state}
> - **Input / Action**:
>   - {concrete input values or actions}
> - **Expected results** (pseudo-assertions):
>   ```
>   assert ...
>   assert ...
>   ```
> - **Edge / boundary notes**: {N/A if none}
> ```

**[Checklist update]** In `.y/tests/{keyword}.checklist.md`, update:
```
- [x] PHASE A: Write test scenarios
```

✅ PHASE A done

---

## PHASE B — Validate scenario spec (Metis items 5–10)

**[Skip if already done]** If PHASE B is `[x]` in the checklist, proceed to the wait point.

**[Pre-check]** `.y/tests/{keyword}.md` must exist before calling Metis.

Call `@metis` with:

> Read `.y/tests/{keyword}.md` and `.y/contracts/{keyword}.md` and strictly review all items below.
>
> #### [Scenario spec review]
> 5. **Assertion format compliance**: Every expected result in `assert <target> <op> <value>` form?
> 6. **Ambiguous expressions**: Any "appropriate", "correct", "normal" etc. remaining?
> 7. **Scenario independence**: Every scenario independently executable without depending on another?
> 8. **Success/failure separation**: Any scenario mixing success and failure?
> 9. **Coverage sufficiency**: Any missing scenarios across Happy Path, Edge Case, Error Case, Security?
> 10. **Contract↔scenario consistency**: Do scenario assertions use the exact status codes, field names, and error codes from the contract?
>     Flag any discrepancy (e.g. scenario asserts `status === 200` but contract specifies `201`).
>
> #### How to report issues
>
> Report issues to the **orchestrator** as a list. Do not call Oracle directly.
> The orchestrator re-calls Oracle with the full original PHASE A instructions plus the Metis report.
> The isolation rule ("never read `.y/plans/`") applies on every re-call without exception.
> The contract file permission ("`.y/contracts/{keyword}.md` may be read") also applies on every re-call.
>
> After each fix, re-review all applicable items.
> **Declare "Scenario spec validated" only when all items pass.**
>
> #### Escape condition
>
> The orchestrator tracks Oracle re-call count.
> **If it exceeds 3**, stop the review loop and escalate via `ask_user_input_v0`:
>
> ```
> ⚠️  Scenario validation loop escape — user intervention required
> ─────────────────────────────────────
> Iterations   : exceeded 3
> Failed items : (item numbers and descriptions)
> ─────────────────────────────────────
> ```
>
> - **Question**: "Scenario validation did not converge. How would you like to proceed?"
> - **Options**: `Proceed with current state` / `Abort`
> - If proceed: log failed items as risks and continue.
> - If abort: terminate.

**[Checklist update]** In `.y/tests/{keyword}.checklist.md`, update:
```
- [x] PHASE B: Validate scenario spec
```

✅ PHASE B done

---

## PHASE C — Write test code

**[Skip if already done]** If PHASE C is `[x]` in the checklist, skip to PHASE D.

**[Pre-check]** Verify `$TEST_OUTPUT_PATH` is available.

- If boulder.json was loaded in Pre-flight and contains `test_output_path` → use that value.
- Otherwise ask via `ask_user_input_v0`:
  - **Question**: "Where should test files be written? (e.g. `src/test/scenario/`)"
  - Store the answer as `$TEST_OUTPUT_PATH`.

Delegate via `delegate_task(subagent_type="deep")` with:

> Read `.y/tests/{keyword}.md` and create **one test file per TC-NNN** in `{$TEST_OUTPUT_PATH}`.
>
> **[TROUBLE_SHOOT constraints]**
> {Insert the numbered prevention checklist, or omit this block if the list is empty.}
> Pay special attention to any items related to assertion errors or test setup mistakes from past sessions.
>
> **⚠️ Implementation isolation**: Do NOT read feature implementation files.
> Write tests based solely on the `Expected results (pseudo-assertions)` in `.y/tests/{keyword}.md`.
>
> [permitted] `.y/contracts/{keyword}.md` may be read.
> Use it to confirm endpoint paths and envelope shapes when constructing HTTP request/response fixtures.
> Do not use it to infer implementation details.
>
> **[Required constraints]**
> - Create one file per TC: `{$TEST_OUTPUT_PATH}/TC-NNN.test.ts`
>   Each file contains **only that one TC**. Never bundle multiple TCs in one file.
> - Add the scenario ID and title as a comment at the top of each file:
>   ```
>   // TC-001: {title}
>   ```
> - Translate `assert A === B` to `expect(A).toBe(B)` or the equivalent framework assertion.
>   Do NOT change the **meaning** of assertions. Translation is purely syntactic.
> - Each file must be independently runnable — include all necessary imports and setup.
> - **Do NOT run tests yet.** Writing only.
> - After writing all files, report the TC list to the orchestrator:
>   ```
>   [PHASE C report]
>   TC files written:
>     - TC-001.test.ts: {title}
>     - TC-002.test.ts: ...
>   Total: N files
>   ```

**[Checklist update]** In `.y/tests/{keyword}.checklist.md`, update:
```
- [x] PHASE C: Write test code
```

✅ PHASE C done

---

## PHASE D — Validate spec↔test translation (Momus)

**[Skip if already done]** If PHASE D is `[x]` in the checklist, skip to PHASE E.

Call `@momus` with:

> Read `.y/tests/{keyword}.md` and verify each `TC-NNN.test.ts` file in `{$TEST_OUTPUT_PATH}` one by one.
> For each TC file, verify that every pseudo-assertion was translated accurately.
>
> #### Validation criteria
>
> For each TC-NNN, judge:
>
> 1. **File existence**: Does `TC-NNN.test.ts` exist for every TC-NNN in the spec? List any missing files immediately.
>
> 2. **Assertion meaning preserved**: Is each `assert` item logically identical to the test code assertion?
>    Find these error types:
>    - Value error: `assert status === 200` → `expect(status).toBe(201)` (value changed)
>    - Direction error: `assert count === (before + 1)` → `expect(count).toBeGreaterThan(0)` (weaker assertion)
>    - Target error: `assert response.body.token is string` → `expect(response.status).toBe(200)` (wrong target)
>    - Omission: an `assert` item from the spec not reflected in the test
>
> 3. **Precondition setup fidelity**: Are scenario preconditions actually implemented in the test setup code?
>
> #### How to handle issues
>
> - Report each flawed TC:
>   ```
>   [TC-NNN] Error type: {value error | direction error | target error | omission}
>   Spec : assert response.status === 200
>   Code : expect(response.status).toBe(201)
>   Fix  : Update TC-NNN.test.ts to match spec
>   ```
> - Fix the individual `TC-NNN.test.ts` file directly. **Never modify `.y/tests/{keyword}.md`.**
> - Re-verify each fixed TC file.
> - Declare "Spec↔test translation validated" only when all TC files pass.
> - **If any fixes were made**, report to the orchestrator:
>   ```
>   [Momus fix report]
>   TC-NNN:
>     Before: expect(response.status).toBe(201)
>     After : expect(response.status).toBe(200)
>     Reason: matched spec assertion response.status === 200
>   ```
>   If no fixes: "No fixes."

Print the Momus report summary:
```
PHASE D — Momus fix report
─────────────────────────────────────
{fix details or "No fixes"}
─────────────────────────────────────
```

**[Checklist update]** In `.y/tests/{keyword}.checklist.md`, update:
```
- [x] PHASE D: Validate spec↔test translation
```

✅ PHASE D done

---

## PHASE E — Run tests and self-correction loop

**[Skip if already done]** If PHASE E is `[x]` in the checklist, skip to PHASE F.

**[Compact context]** Before entering this phase, summarize PHASE A–D logs in the format below, then immediately run `/compact` to remove the raw logs from context.

```
Context summary — PHASE A–D
  Scenario count    : Happy Path N / Edge N / Error N / Security N
  Oracle re-calls   : {N}
  Metis issues      : {summary or "none"}
  Momus fix summary : {fixed TC list and types or "no fixes"}
  TC files          : TC-001 ~ TC-NNN, total N
  Test output path  : {$TEST_OUTPUT_PATH}
```

The orchestrator declares the **active Scope** and **TC queue**:

```
▶ PHASE E entry — Active Scope
   Source files (from boulder.json implementation_scope):
     - {implementation_scope entries, one per line}
   Test output path : {$TEST_OUTPUT_PATH}
   TC queue         : [TC-001, TC-002, ..., TC-NNN]
   Passed TCs       : []
   Failed TCs       : []
   Excluded TCs     : []
   Retry counters   : {}
```

**Sequential TC execution — process one TC at a time:**

For each TC in the queue (in order), repeat the following cycle:

1. Run **only** `TC-NNN.test.ts` for the current TC.
   - **If it passes**: add to Passed TCs, print `✅ TC-NNN passed`, move to the next TC.
   - **If it fails**: go to step 2.

2. Call `@momus` with:
   > `TC-NNN.test.ts` failed. Compare the failure against the spec in `.y/tests/{keyword}.md`.
   > Classify as **"feature bug"** or **"test bug"** and state the reason.
   > Error log: {error}

3. Branch on Momus's verdict:
   - **Feature bug** → delegate via `delegate_task(subagent_type="deep")`:
     > **[TROUBLE_SHOOT constraints]**
     > {Insert the numbered prevention checklist, or omit this block if the list is empty.}
     > Check whether this failure matches any past pattern above before writing a fix.
     >
     > Fix only the **feature code** causing `TC-NNN` to fail. Do not touch `TC-NNN.test.ts`.
     > Error: {message}
     > **[Scope constraint]** Only modify files listed in the active Scope declaration above (from boulder.json `implementation_scope`).
     > If you need a file outside that list, stop and report to the orchestrator.
   - **Test bug** → delegate via `delegate_task(subagent_type="deep")`:
     > **[TROUBLE_SHOOT constraints]**
     > {Insert the numbered prevention checklist, or omit this block if the list is empty.}
     > Check whether this failure matches any past assertion-translation error above before writing a fix.
     >
     > Fix only the bug in `TC-NNN.test.ts`. Do not touch feature code or the scenario spec.
     > Bug: {Momus's reason}
     > After fixing, Momus will re-verify assertion meaning preservation.
     > If meaning preservation fails, count this as a retry and return to step 1.

4. Re-run `TC-NNN.test.ts` only.
   - If it passes: add to Passed TCs, print `✅ TC-NNN passed`, move to the next TC.
   - If it fails: increment retry counter and return to step 2.

5. **Escape condition**: If the **retry counter for TC-NNN exceeds 3**, escalate via `ask_user_input_v0`:

```
⚠️  TC escape — user intervention required
─────────────────────────────────────
TC           : TC-NNN
Last error   : (error message summary)
Momus verdict: (feature bug / test bug)
Retry count  : 3
─────────────────────────────────────
```

- **Question**: "TC-NNN auto-correction failed. How would you like to proceed?"
- **Options**: `Skip this TC and continue` / `Abort`
- If skip: add TC-NNN to Excluded TCs and continue. Record in PHASE F.
- If abort: terminate.

**When all TCs are processed** (passed or excluded), print the summary:

```
PHASE E summary
─────────────────────────────────────
Passed  : {TC list}
Excluded: {TC list or "none"}
─────────────────────────────────────
```

**[Checklist update]** In `.y/tests/{keyword}.checklist.md`, update:
```
- [x] PHASE E: Run tests and self-correction loop
```

✅ PHASE E done

---

## PHASE F — Record troubleshooting

**[Mandatory execution rule]**
This phase is **skippable ONLY IF all of the following are true**:
- No TC had retry count ≥ 1 in PHASE E
- No Oracle re-calls occurred in PHASE B
- No Momus fixes occurred in PHASE D
- No escaped TCs in PHASE E

If even one condition is false, this phase is **mandatory and must not be skipped**.

Invoke `/gen-trouble-shoot` with the `test` source flag and the project root:

```
/gen-trouble-shoot --source test --label {keyword} --project {$PROJECT_DIR}
```

The sub-command reads the current session history, extracts test-level AI mistakes (TC failures, assertion translation errors, Metis scenario issues, escaped TCs), and writes them to `{$PROJECT_DIR}/TROUBLE_SHOOT.md`.

**[Checklist update]** In `.y/tests/{keyword}.checklist.md`, update:
```
- [x] PHASE F: Record troubleshooting
```

✅ PHASE F done

---

## Final report

Print and terminate:

```
✅ /implement-test-scenario complete

keyword          : {keyword}
test scenarios   : .y/tests/{keyword}.md
test checklist   : .y/tests/{keyword}.checklist.md
test code        : {$TEST_OUTPUT_PATH}  (n/a if PHASE C was not run)
spec↔test check  : done / Momus fixes: {N or "none"}  (n/a if PHASE D was not run)
test results     : passed {N} / excluded {N} / total {N}  (n/a if PHASE E was not run)
                   excluded: {TC ID list or "none"}
troubleshooting  : delegated to /gen-trouble-shoot  (skipped if nothing to record)
```
