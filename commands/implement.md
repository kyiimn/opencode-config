---
description: End-to-end implementation workflow that prevents AI bias through requirement-based test scenario planning, strict cross-validation by Metis, and separation of coding from test writing. Enforces specs as observable pseudo-assertions and achieves Spec-Driven Verification (SDV) via Momus's spec↔test translation validation.
---

You are the orchestrator. Execute the workflow below **in order, without interruption**.
Print `▶ PHASE N start` at the beginning of each step and `✅ PHASE N done` upon completion.
Never skip or reorder steps.

After printing `✅ PHASE N done`, immediately mark that PHASE as complete in the plan file by changing its checkbox from `[ ]` to `[x]` in the **PHASE Checklist** section of `.sisyphus/plans/{keyword}.md`. This updates the boulder progress tracker. If the plan file does not yet exist (PHASEs 0–1a), perform the mark as soon as the file is created.

---

## Pre-flight — Declare keyword

Before PHASE 0, determine the **keyword** by the rules below. Use it consistently in every step.

- If the first word of `$ARGUMENTS` is a valid identifier (letters, digits, hyphens only), use it as the keyword.
- Otherwise summarize `$ARGUMENTS` into a ≤3-word lowercase kebab-case slug.
  - e.g. "Add user login feature" → `user-login`
  - e.g. "Implement payment refund API" → `payment-refund`

Print:
```
keyword  : {keyword}
plan     : .sisyphus/plans/{keyword}.md
contract : .sisyphus/contracts/{keyword}.md
scenarios: .sisyphus/tests/{keyword}.md
```

After declaring keyword, initialize or resume boulder state:

- Read `.sisyphus/boulder.json` if it exists.
  - If `plan_name` matches the current keyword → **resume mode**: print `🔁 Resuming: {keyword}`, read the plan file's checkbox list to identify completed PHASEs, skip those PHASEs and continue from the first unchecked one.
  - If `plan_name` differs → a different task is active. Print a warning and proceed as a new session (overwrite).
- If the file does not exist → **new session**: create `.sisyphus/boulder.json` with the structure below. Use the current working directory as the absolute path base. Obtain the session ID from the environment if available; otherwise use `"unknown"`.

```json
{
  "active_plan": "{absolute path to .sisyphus/plans/{keyword}.md}",
  "plan_name": "{keyword}",
  "started_at": "{ISO 8601 timestamp}"
}
```



---

## Requirements

$ARGUMENTS

---

## PHASE 0 — Load project context

Check for these files at the project root. Read each one that exists.

| File | Purpose | Usage |
|---|---|---|
| `TROUBLE_SHOOT.md` | Past AI mistakes and prevention rules | Apply throughout all steps to avoid repeating errors |
| `RULES.md` | Project-wide coding rules and conventions | Enforce without exception in all implementation, test, and refactor steps |
| `DESIGN.md` | UX/UI principles, interaction guide, design conventions | Follow in all frontend-related work |
| `ARCHITECTURE.md` | System architecture, layer structure, tech-stack conventions | Use as the basis for planning (Prometheus) and implementation (Atlas) |

- If none of the four files exist, skip to PHASE 1a.
- After reading, print:

```
▶ PHASE 0 done
   TROUBLE_SHOOT.md : {found/not found}
   RULES.md         : {found/not found}
   DESIGN.md        : {found/not found}
   ARCHITECTURE.md  : {found/not found}
```

- If `TROUBLE_SHOOT.md` was found, **immediately extract every rule or open issue from it** and print as a numbered prevention checklist:

```
[!] TROUBLE_SHOOT prevention checklist ({N} items)
   1. {rule or open issue, one line}
   2. ...
```

  This checklist is the **active guard rail for the entire workflow**.
  It must be passed verbatim to every agent call in PHASE 1b, PHASE 2, PHASE 5, PHASE 7, PHASE 9, and PHASE 10 as a `[TROUBLE_SHOOT constraints]` block (see each PHASE for the exact injection point).
  If `TROUBLE_SHOOT.md` was not found, the checklist is empty and the injection block is omitted.

---

## PHASE 1a — Write API contract

**The orchestrator** writes the API contract file before spawning any agent.
This step must complete before PHASE 1b and PHASE 2 begin.

Read `ARCHITECTURE.md` (loaded in PHASE 0) for HTTP conventions, then extract every HTTP endpoint mentioned or implied by `$ARGUMENTS`.

### Deliverable — `.sisyphus/contracts/{keyword}.md`

This file is the **single source of truth for all HTTP interfaces** in this task.
It is derived from requirements only — not from implementation decisions.
Any agent in this workflow may read it, **including Oracle**.
(The SDV isolation rule forbids reading `.sisyphus/plans/` only — the contract file is explicitly permitted.)

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
  If `ARCHITECTURE.md` is not found, default to the project-standard envelope:
  - Success: `{ ok: true, data: ... }`
  - Error: `{ ok: false, message: string, code?: string }`
- Use TypeScript inline types for all shapes. Do not reference external type files.
- Derive error codes from domain semantics (e.g. `PAYMENT_NOT_FOUND`, `ALREADY_REFUNDED`).
  Do not invent codes not inferable from requirements.
- If a requirement implies an endpoint but does not specify the full shape,
  mark the ambiguous field as `unknown /* clarify */` and note it in a `> ⚠️ Ambiguity:` block beneath the section.
- If the requirements contain **no HTTP endpoints** (e.g. pure CLI or background job),
  write a single line: `> No HTTP endpoints in this task.` and proceed to PHASE 1b immediately.

After writing, print:

```
API contract written: .sisyphus/contracts/{keyword}.md
   Endpoints defined: {list of METHOD /path, or "none"}
   Ambiguities      : {count or "none"}
```

---

## PHASE 1b — Write implementation plan (Prometheus)

**Run in parallel with PHASE 2.** PHASE 1b and PHASE 2 do not depend on each other's output — start both simultaneously. **Wait for both to finish before proceeding to PHASE 3.**

Call `@prometheus` with:

> Write an **implementation plan only** for the requirements below.
> **Do NOT write test scenarios at this step.** A separate agent handles tests independently.
>
> If context files were loaded in PHASE 0, internalize them before planning:
> - `RULES.md` → follow coding rules and conventions without exception.
> - `ARCHITECTURE.md` → use layer structure and tech-stack conventions as design baseline.
> - `DESIGN.md` → apply UX/UI principles to any frontend-related output.
>
> **[TROUBLE_SHOOT constraints]**
> {Insert the numbered prevention checklist extracted in PHASE 0, or omit this block if the list is empty.}
> Before writing a single line of the plan, go through each item above and explicitly confirm your approach does not violate it.
> For every item, add a corresponding guard note inside the plan section it applies to (e.g. `<!-- TS-3: validated -->`).
> If an item conflicts with the current requirements, stop and surface the conflict to the orchestrator before proceeding.
>
> **Also read `.sisyphus/contracts/{keyword}.md` if it exists.**
> The contract file defines the agreed HTTP interface for this task.
> Do not contradict or extend it — implement exactly what it specifies.
> If the contract contains `⚠️ Ambiguity` notes, resolve them conservatively and document the decision.
>
> ### Deliverable — `.sisyphus/plans/{keyword}.md`
>
> The plan file must begin with a **PHASE checklist** section before any other content.
> This checklist is the progress tracker read by the boulder state system (`getPlanProgress`).
>
> ```markdown
> ## PHASE Checklist
>
> - [ ] PHASE 0: Load project context
> - [ ] PHASE 1a: Write API contract
> - [ ] PHASE 1b: Write implementation plan
> - [ ] PHASE 2: Write test scenarios
> - [ ] PHASE 3: Cross-validate plan and scenarios
> - [ ] PHASE 4: Confirm implementation
> - [ ] PHASE 5: Implement code
> - [ ] PHASE 6: Confirm test generation
> - [ ] PHASE 7: Write test code
> - [ ] PHASE 8: Validate spec↔test translation
> - [ ] PHASE 9: Run tests and self-correction loop
> - [ ] PHASE 10: Refactor
> - [ ] PHASE 11: Save context memory
> - [ ] PHASE 12: Record troubleshooting
> - [ ] PHASE 13: Final report
> ```
>
> After the checklist, write the detailed implementation plan:
> - Write a detailed plan including function names, major logic flows, and dependency relationships.
> - Use Librarian/Explore to find reusable utilities in the codebase first; document the reuse plan.
> - **Do NOT write any code (implementation or test) at this step.**
> - Include a **[File Scope]** section in this format — it is the absolute boundary for all subsequent steps:
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
> $ARGUMENTS

---

## PHASE 2 — Write test scenarios (Oracle)

**Run in parallel with PHASE 1b.** PHASE 1b and PHASE 2 do not depend on each other's output — start both simultaneously. **Wait for both to finish before proceeding to PHASE 3.**

Call `@oracle` with:

> Read the requirements below and write a functional test scenario spec.
>
> **⚠️ Do NOT read `.sisyphus/plans/` or any implementation plan files.**
> This spec must be based on requirements only. Reading the plan contaminates SDV independence.
>
> ※ **Isolation scope**: This is a filesystem-level isolation rule. If `@oracle` runs as a sub-agent with an independent context window in oh-my-opencode, isolation is guaranteed. Otherwise, Prometheus's plan content may remain in the orchestrator context. In either case, Oracle must honor the file access prohibition and reason only from the requirements text.
>
> [permitted] `.sisyphus/contracts/{keyword}.md` may be read.
> This file contains the agreed HTTP interface derived from requirements — it is not an implementation plan.
> If it exists, read it before writing scenarios.
> Use the exact status codes, response field names, and error codes defined there.
> Do not invent or assume interface details not present in the contract or requirements.
>
> ### Deliverable — `.sisyphus/tests/{keyword}.md`
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
> For async responses, streaming, or complex state changes that cannot be expressed as a single comparison, decompose into observable assertions using these keywords.
>
> Forbidden (prose — never use):
> ```
> - User is created successfully       ← forbidden
> - An appropriate error is returned   ← forbidden
> - Token is issued correctly          ← forbidden
> ```
>
> **[Rule 2] No ambiguous adjectives.** Ban "appropriate", "correct", "normal", "sufficient", "fast" inside assertions. Replace with concrete values, types, ranges, or call counts.
>
> **[Rule 3] Each scenario must be independently executable.** No preconditions that depend on another scenario's result. Express preconditions only as states the test environment can set up independently.
>
> **[Rule 4] Separate success and failure cases into distinct scenarios.** Never mix success and failure in one scenario.
>
> #### Scenario format (apply to every scenario without exception)
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
>
> Requirements:
> $ARGUMENTS

---

## PHASE 3 — Cross-validate plan and scenarios (Metis)

**[Join point]** Start only after all three of the following exist:
- `.sisyphus/contracts/{keyword}.md`
- `.sisyphus/plans/{keyword}.md`
- `.sisyphus/tests/{keyword}.md`

Call `@metis` with:

> Read `.sisyphus/contracts/{keyword}.md`, `.sisyphus/plans/{keyword}.md`, and `.sisyphus/tests/{keyword}.md` and strictly review all items below.
>
> #### [Contract review]
> 0. **Contract↔plan consistency**: Does the plan implement every endpoint in the contract exactly?
>    Flag any status code, field name, or error code the plan would contradict or omit.
>
> #### [Plan review]
> 1. **Missing requirements**: Any user requirement not reflected in the plan?
> 2. **Error handling gaps**: Missing handling for error cases, boundary values, or abnormal inputs?
> 3. **Security vulnerabilities**: Auth, input validation, or data exposure gaps?
> 4. **Architecture gaps**: Missing dependencies, wrong layer responsibility, or scalability issues?
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
> Report issues to the **orchestrator** as a list. Do not call Prometheus or Oracle directly.
> The orchestrator re-calls the relevant agent with your full report.
> - Contract↔plan issues (0) → orchestrator re-calls `@prometheus` with the full Metis report.
> - Plan issues (1–4) → orchestrator re-calls `@prometheus` with the full Metis report.
> - Scenario issues (5–10) → orchestrator re-calls `@oracle` with the **full original PHASE 2 instructions (including isolation rules) plus the Metis report**.
>   The isolation rule ("never read `.sisyphus/plans/`") applies on every re-call without exception.
>   The contract file permission ("`.sisyphus/contracts/{keyword}.md` may be read") also applies on every re-call.
>
> After each fix, re-review all items.
> **Declare "Cross-validation complete" only when all 10 items pass.**
>
> #### Escape condition
>
> The orchestrator tracks `@prometheus` re-call count and `@oracle` re-call count independently.
> **If either exceeds 3**, stop the review loop and escalate to the user via `ask_user_input_v0`:
>
> ```
> ⚠️  Cross-validation loop escape — user intervention required
> ─────────────────────────────────────
> Iterations   : exceeded 3
> Failed items : (item numbers and descriptions)
> Last fix by  : (Prometheus / Oracle)
> ─────────────────────────────────────
> ```
>
> - **Question**: "Validation loop did not converge. How would you like to proceed?"
> - **Options**: `Proceed with current state` / `Abort workflow`
> - If proceed: log failed items as risks and move to PHASE 4.
> - If abort: terminate the workflow.

---

## PHASE 4 — Confirm implementation (User Confirmation)

**[Notepad init]** Ensure `.sisyphus/notepads/{keyword}/` directory exists. After each major decision or discovery in this workflow, append a brief note to `.sisyphus/notepads/{keyword}/notes.md` (architectural decisions, edge cases found, risks identified). On resume, read this file first to regain situational awareness.

**[Compact context]** Before entering this step, summarize PHASE 0–3 logs in the format below, then immediately run `/compact` to remove the raw logs from context. Pass the summary block as the compaction hint so it remains accessible in later steps.

```
Context summary — PHASE 0–3
  Context files loaded : {found/not found list}
  API contract         : {endpoint count or "none"} — .sisyphus/contracts/{keyword}.md
  Prometheus plan core : {2–3 lines of key design decisions}
  Oracle scenario count: Happy Path N / Edge N / Error N / Security N
  Metis issues         : {summary of fix requests or "none"}
  Prometheus re-calls  : {N}, Oracle re-calls: {N}
```

**[Pre-check]** Before calling `ask_user_input_v0`, verify:
- Metis declared "Cross-validation complete"?
- All three files exist: `.sisyphus/contracts/{keyword}.md`, `.sisyphus/plans/{keyword}.md`, `.sisyphus/tests/{keyword}.md`?

Wait until both conditions are met before continuing.

**[4-1] Print plan summary**

Print before calling `ask_user_input_v0`:

```
Plan summary
─────────────────────────────────────
Files to create/modify:
  (list from Scope section of the plan)

API contract:
  (endpoint list from .sisyphus/contracts/{keyword}.md, or "none")

Risks and notes:
  (risks flagged by Metis or Prometheus, or "none")

Test scenario count:
  Happy Path N / Edge Case N / Error Case N / Security N / Total N

Shared utilities to reuse:
  (items from reuse plan, or "none")
─────────────────────────────────────
Full plan    : .sisyphus/plans/{keyword}.md
API contract : .sisyphus/contracts/{keyword}.md
```

**[4-2] Ask user**

Call `ask_user_input_v0`:
- **Question**: "Proceed with implementation as planned?"
- **Options**: `Yes (Y)` / `Abort (N)`
- If **N**: terminate immediately and print "Workflow aborted by user."

---

## PHASE 5 — Implement code (Atlas)

If user chose Y, call `@atlas` with:

> Read `.sisyphus/plans/{keyword}.md` and implement the feature code.
> This plan was created by Prometheus, validated by Metis, and approved by the user.
> **Start implementing immediately — no re-planning, re-investigation, or user confirmation.**
>
> **Also read `.sisyphus/contracts/{keyword}.md` if it exists.**
> Implement HTTP handlers to match the contract exactly — do not deviate from specified status codes, envelope shapes, or error codes.
>
> **[TROUBLE_SHOOT constraints]**
> {Insert the numbered prevention checklist extracted in PHASE 0, or omit this block if the list is empty.}
> Before writing any code, verify your approach against each item above.
> If an item is relevant to the current file or function, add an inline comment: `// TS-N: {rule summary}` at the point where the guard applies.
>
> **[Required constraints]**
> - **Do NOT write test code at this step.** Focus solely on feature implementation.
> - Reuse utilities listed in the plan — import them, do not rewrite.
> - Write complete JSDoc for every function, class, and interface **now** — do not defer.
>   Always include `@param`, `@throws`, `@returns`. Add `@example` for complex branching logic.
>   Simple accessors (getter/setter) may omit JSDoc.
> - After implementation, self-check all JSDoc for missing required tags (`@param`, `@throws`, `@returns`).
>   Fix any gaps immediately and report to the orchestrator:
>   ```
>   [Atlas JSDoc report]
>   Fixed: {file path} — added: {@throws, etc.}
>   ```
>   If nothing to fix, report: "JSDoc check passed."
> - **[Scope constraint — strict]** Only create or modify files listed in the File Scope section of `.sisyphus/plans/{keyword}.md`.
>   If you need to touch an out-of-scope file, stop immediately and report to the orchestrator.
>   The orchestrator will request user approval via `ask_user_input_v0` before allowing it:
>   > Out-of-scope file modification request
>   > File: {path}
>   > Reason: {why it is needed}
>   > Impact: {potential effect on other features}
>   Modifying out-of-scope files without approval is strictly forbidden.

---

## PHASE 6 — Confirm test generation (User Confirmation)

**[Pre-check]** Before calling `ask_user_input_v0`, verify:
- Atlas completed feature implementation?
- Orchestrator received Atlas's JSDoc report (and confirmed any fixes)?

Wait until both conditions are met.

Call `ask_user_input_v0`:
- **Question**: "Implementation complete. Generate test code and verify functionality based on `.sisyphus/tests/{keyword}.md`?"
- **Options**: `Yes (Y)` / `Skip (N)`
- If **N**: jump to PHASE 10 immediately. Record "tests not run" in PHASE 11.

---

## PHASE 7 — Write test code

If user chose Y, delegate via `delegate_task(subagent_type="business-logic")` with:

> Read `.sisyphus/tests/{keyword}.md` and create **one test file per TC-NNN**.
>
> **[TROUBLE_SHOOT constraints]**
> {Insert the numbered prevention checklist extracted in PHASE 0, or omit this block if the list is empty.}
> Pay special attention to any items related to assertion errors or test setup mistakes from past sessions.
>
> **⚠️ Implementation isolation**: Do NOT open or read the new/modified files listed in the Scope section of `.sisyphus/plans/{keyword}.md`.
> Write tests based solely on the `Expected results (pseudo-assertions)` in the spec.
> Referencing implementation internals (function signatures, class structure, variable names) ties tests to implementation and violates SDV.
>
> [permitted] `.sisyphus/contracts/{keyword}.md` may be read.
> Use it to confirm endpoint paths and envelope shapes when constructing HTTP request/response fixtures.
> Do not use it to infer implementation details.
>
> **[Required constraints]**
> - Create one file per TC: `{test output path}/TC-NNN.test.ts` (e.g. `TC-001.test.ts`, `TC-002.test.ts`).
>   Each file contains **only that one TC**. Never bundle multiple TCs in one file.
> - Add the scenario ID and title as a comment at the top of each file:
>   ```
>   // TC-001: Normal user creation
>   ```
> - Translate `assert A === B` to `expect(A).toBe(B)` or the equivalent framework assertion.
>   Do NOT change the **meaning** of assertions. Translation is purely syntactic.
> - Each file must be independently runnable — include all necessary imports and setup.
> - **Do NOT run tests yet.** Writing only.
> - After writing all files, report the TC list to the orchestrator:
>   ```
>   [PHASE 7 report]
>   TC files written:
>     - TC-001.test.ts: Normal user creation
>     - TC-002.test.ts: ...
>   Total: N files
>   ```

---

## PHASE 8 — Validate spec↔test translation (Momus)

Immediately after test writing completes, call `@momus` with:

> Read `.sisyphus/tests/{keyword}.md` and verify each `TC-NNN.test.ts` file in the test output path one by one.
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
> - Fix the individual `TC-NNN.test.ts` file directly. **Never modify `.sisyphus/tests/{keyword}.md`.**
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

After PHASE 8, the orchestrator prints and proceeds to PHASE 9:

```
PHASE 8 Momus fix report
─────────────────────────────────────
{fix details or "No fixes"}
─────────────────────────────────────
```

---

## PHASE 9 — Run tests and self-correction loop

**[Compact context]** Before entering this step, summarize PHASE 5–8 logs in the format below, then immediately run `/compact` to remove the raw logs from context. Pass the summary block as the compaction hint so it remains accessible in later steps.

```
Context summary — PHASE 5–8
  Implemented files : {file list}
  JSDoc fixes       : {file list or "none"}
  Momus fix summary : {fixed TC list and types or "no fixes"}
  TC files          : {TC-001 ~ TC-NNN, total N}
```

The orchestrator declares the **active Scope** and **TC queue**:

```
▶ PHASE 9 entry — Active Scope
   Modifiable files:
     - {new files + allowed modifications from Scope section}
   Test output path: {from Scope section}
   TC queue        : [TC-001, TC-002, ..., TC-NNN]  ← process in order
   Passed TCs      : []
   Failed TCs      : []
   Excluded TCs    : []
   Retry counters  : {}
```

**Sequential TC execution — process one TC at a time:**

For each TC in the queue (in order), repeat the following cycle:

1. Run **only** `TC-NNN.test.ts` for the current TC.
   - **If it passes**: add to Passed TCs, print `✅ TC-NNN passed`, move to the next TC.
   - **If it fails**: go to step 2.

2. Call `@momus` with:
   > `TC-NNN.test.ts` failed. Compare the failure against the spec in `.sisyphus/tests/{keyword}.md`.
   > Classify as **"feature bug"** or **"test bug"** and state the reason.
   > Error log: {error}

3. Branch on Momus's verdict:
   - **Feature bug** → delegate via `delegate_task(subagent_type="deep")`:
     > **[TROUBLE_SHOOT constraints]**
     > {Insert the numbered prevention checklist extracted in PHASE 0, or omit this block if the list is empty.}
     > Check whether this failure matches any past pattern above before writing a fix.
     >
     > Fix only the **feature code** causing `TC-NNN` to fail. Do not touch `TC-NNN.test.ts`.
     > Error: {message}
     > **[Scope constraint]** Only modify files in the active Scope declaration.
     > If you need an out-of-scope file, stop and report to the orchestrator.
   - **Test bug** → delegate via `delegate_task(subagent_type="deep")`:
     > **[TROUBLE_SHOOT constraints]**
     > {Insert the numbered prevention checklist extracted in PHASE 0, or omit this block if the list is empty.}
     > Check whether this failure matches any past assertion-translation error above before writing a fix.
     >
     > Fix only the bug in `TC-NNN.test.ts`. Do not touch feature code or the scenario spec.
     > Bug: {Momus's reason}
     > After fixing, Momus will re-verify assertion meaning preservation.
     > If meaning preservation fails, count this as a retry and return to step 1.

4. Re-run `TC-NNN.test.ts` only.
   - If it passes: add to Passed TCs, print `✅ TC-NNN passed`, move to the next TC.
   - If it fails: increment retry counter for this TC and return to step 2.

5. **Escape condition**: If the **retry counter for TC-NNN exceeds 3**, mark it as escaped and escalate via `ask_user_input_v0`:

```
⚠️  TC escape — user intervention required
─────────────────────────────────────
TC          : TC-NNN
Last error  : (error message summary)
Momus verdict: (feature bug / test bug)
Retry count : 3
─────────────────────────────────────
```

- **Question**: "TC-NNN auto-correction failed. How would you like to proceed?"
- **Options**: `Skip this TC and continue` / `Abort workflow`
- If skip: add TC-NNN to Excluded TCs and move to the next TC. Record in PHASE 11.
- If abort: terminate the workflow.

**When all TCs in the queue are processed** (passed or excluded), print the summary and proceed to PHASE 10:

```
PHASE 9 summary
─────────────────────────────────────
Passed  : {TC list}
Excluded: {TC list or "none"}
─────────────────────────────────────
```

---

## PHASE 10 — Refactor

Delegate via `delegate_task(subagent_type="deep")` with:

> **[TROUBLE_SHOOT constraints]**
> {Insert the numbered prevention checklist extracted in PHASE 0, or omit this block if the list is empty.}
> Verify the refactored output does not reintroduce any pattern listed above.
>
> Read the `/refactor` skill file first, then refactor following its guidelines.
> Also remove all unused imports and variables across the source.
> **[Scope constraint — strict]** Only refactor files listed in the File Scope section of `.sisyphus/plans/{keyword}.md`.
> **Exclude test files** (the "Test output path" files) from refactoring — Momus's spec↔test validation is complete and modifying them risks corrupting assertion meaning.
> If you find improvements needed in out-of-scope files, do NOT make changes — note them separately.
> If an out-of-scope file modification is truly required, stop and report to the orchestrator. The orchestrator will request user approval via `ask_user_input_v0` before allowing it.
> After refactoring, **re-run each `TC-NNN.test.ts` sequentially** (only if test code exists). Skip excluded TCs from PHASE 9 automatically.
> If not, skip this check.
> **If any TC fails, stop and report the failed TC ID and error log to the orchestrator.**

If test failures are reported after refactoring, **re-enter the PHASE 9 sequential TC loop**.
Before re-entry, the orchestrator re-declares the active Scope and resets the TC queue:

```
PHASE 9 re-entry — Active Scope (refactored files only)
   Modifiable files:
     - {files actually modified during refactoring}
   Out-of-scope: all files not in the list above (never modify)
   TC queue     : [TC-001, TC-002, ..., TC-NNN]  ← full queue, re-run all
   TC counter   : reset to 0 (refactoring failures have different root causes)
   Excluded TCs (cannot re-verify):
     - {TC IDs escaped in PHASE 9 — "none" if none}
     ※ If excluded TCs fail again, skip them automatically without escalation.
```

---

## PHASE 11 — Save context memory

Call `@document-writer` with:

> Call the `memory-manager` skill to permanently store this session's work.
> Record every item below without omission:
>
> - Core requirement summary for this task
> - List of files created/modified with paths
> - API contract summary (endpoint list from `.sisyphus/contracts/{keyword}.md`, or "none")
> - Tech stack applied and key design decisions
> - Reused shared utilities (file path + import path)
> - **Test execution**: if run, record pass/fail results; if skipped, record "tests not run"
> - **Unresolved failures**: if any TCs were skipped via loop escape, record scenario IDs and last errors
> - **Troubleshooting details**: for each issue encountered, record:
>   - What mistake or error occurred and in what situation
>   - Root cause
>   - How it was resolved

---

## PHASE 12 — Record troubleshooting

**[Pre-check]** Start only after PHASE 11 memory save is confirmed complete.

**[Mandatory execution rule]**
This step is **skippable ONLY IF all of the following are true**:
- No TC had retry count ≥ 1 in PHASE 9
- No Metis re-calls occurred in PHASE 3
- No Momus fixes occurred in PHASE 8
- No escaped TCs in PHASE 9
- No out-of-scope file requests in PHASE 5 or PHASE 10
- No Prometheus re-calls occurred in PHASE 3

If even one condition is false, this step is **mandatory and must not be skipped**.

Call `@document-writer` with:

> Extract AI mistakes, wrong approaches, and recurring problems from this entire workflow and record them in `TROUBLE_SHOOT.md` at the project root.
>
> #### What to extract
>
> Include **all** of the following — do not filter or omit categories:
> - Code incorrectly implemented by AI that required fixes (including partial rewrites)
> - Cases where AI chose the wrong architecture or approach and had to backtrack
> - Repeated error patterns in the PHASE 9 self-correction loop (retry count ≥ 2)
> - Gaps or errors flagged by Metis or Momus, even if subsequently fixed
> - Assertion translation errors found in spec↔test validation
> - **Escaped TCs**: test cases that could not be resolved after 3 retries — record as unresolved
> - Any requirement misunderstanding that led to rework
> - Any sub-task that required more than 2 attempts for any reason
>
> Exclude: normal design decisions, user-initiated requirement changes.
>
> #### Format
>
> Write each workflow as one dated entry. Within the entry, separate resolved from unresolved items.
> Detailed problem/cause/fix info is already in memory — do not duplicate it here; one-line rules are sufficient for resolved items.
>
> ```markdown
> ## [YYYY-MM-DD] {task keyword}
>
> ### Resolved
> - **[Rule]** {one-line prevention rule}
>   - What went wrong: {brief description}
>   - Detected by: Metis | Momus | Test failure | Self-correction
>
> ### Unresolved
> - **[Open]** {description of the issue}
>   - Escaped TC: {TC-NNN} — Last error: {one-line summary}
>   - Attempted: {what was tried}
> ```
>
> If Resolved has nothing to record, write `(none)`.
> If Unresolved has nothing to record, write `(none)`.
> **Do not omit either section header.**
>
> #### File handling
>
> - If `TROUBLE_SHOOT.md` already exists, keep existing content and **prepend** the new entry.
> - If it does not exist, create it.

---

## PHASE 13 — Final report

Print the final report and terminate the workflow:

```
✅ /implement workflow complete

keyword          : {keyword}
API contract     : .sisyphus/contracts/{keyword}.md  (n/a if no HTTP endpoints)
plan             : .sisyphus/plans/{keyword}.md
test scenarios   : .sisyphus/tests/{keyword}.md
test code        : {test output path from Scope}  (n/a if skipped)
spec↔test check  : done (n/a if skipped) / Momus fixes: {N or "none"}
test results     : passed {N} / excluded {N} / total {N}  (n/a if skipped)
                   excluded: {TC ID list or "none"}
refactor         : done
memory saved     : done
troubleshooting  : done (skipped if nothing to record)

All tasks completed successfully.
```
