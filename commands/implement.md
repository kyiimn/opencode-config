---
description: End-to-end implementation workflow that prevents AI bias through requirement-based test scenario planning, strict cross-validation by Metis, and separation of coding from test writing. Enforces specs as observable pseudo-assertions and achieves Spec-Driven Verification (SDV) via Momus's spec↔test translation validation.
---

You are the orchestrator. Execute the workflow below **in order, without interruption**.
Print `▶ PHASE N start` at the beginning of each step and `✅ PHASE N done` upon completion.
Never skip or reorder steps.

---

## Pre-flight — Declare keyword

Before PHASE 0, determine the **keyword** by the rules below. Use it consistently in every step.

- If the first word of `$ARGUMENTS` is a valid identifier (letters, digits, hyphens only), use it as the keyword.
- Otherwise summarize `$ARGUMENTS` into a ≤3-word lowercase kebab-case slug.
  - e.g. "Add user login feature" → `user-login`
  - e.g. "Implement payment refund API" → `payment-refund`

Print:
```
📌 keyword: {keyword}
   plan     : .sisyphus/plans/{keyword}.md
   scenarios: .sisyphus/tests/{keyword}.md
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

- If none of the four files exist, skip to PHASE 1.
- After reading, print: `▶ PHASE 0 done — TROUBLE_SHOOT.md: {found/not found}, RULES.md: {found/not found}, DESIGN.md: {found/not found}, ARCHITECTURE.md: {found/not found}`

---

## PHASE 1 — Write implementation plan (Prometheus)

Call `@prometheus` with:

> Write an **implementation plan only** for the requirements below.
> **Do NOT write test scenarios at this step.** A separate agent handles tests independently.
>
> If context files were loaded in PHASE 0, internalize them before planning:
> - `TROUBLE_SHOOT.md` → embed prevention rules throughout the plan.
> - `RULES.md` → follow coding rules and conventions without exception.
> - `ARCHITECTURE.md` → use layer structure and tech-stack conventions as design baseline.
> - `DESIGN.md` → apply UX/UI principles to any frontend-related output.
>
> ### Deliverable — `.sisyphus/plans/{keyword}.md`
>
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
>   - {test directory path, e.g. apps/api/src/tests/scenario/}
>
>   ### Out-of-scope (never modify)
>   - All existing files not listed above
>   ```
>
> Requirements:
> $ARGUMENTS

---

## PHASE 2 — Write test scenarios (Oracle)

**Run in parallel with PHASE 1.** PHASE 1 and PHASE 2 do not depend on each other's output — start both simultaneously. **Wait for both to finish before proceeding to PHASE 3.**

Call `@oracle` with:

> Read the requirements below and write a functional test scenario spec.
>
> **⚠️ Do NOT read `.sisyphus/plans/` or any implementation plan files.**
> This spec must be based on requirements only. Reading the plan contaminates SDV independence.
>
> ※ **Isolation scope**: This is a filesystem-level isolation rule. If `@oracle` runs as a sub-agent with an independent context window in oh-my-opencode, isolation is guaranteed. Otherwise, Prometheus's plan content may remain in the orchestrator context. In either case, Oracle must honor the file access prohibition and reason only from the requirements text.
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

**[Join point]** Start only after both `.sisyphus/plans/{keyword}.md` and `.sisyphus/tests/{keyword}.md` exist.

Call `@metis` with:

> Read `.sisyphus/plans/{keyword}.md` and `.sisyphus/tests/{keyword}.md` and strictly review all items below.
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
>
> #### How to report issues
>
> Report issues to the **orchestrator** as a list. Do not call Prometheus or Oracle directly.
> The orchestrator re-calls the relevant agent with your full report.
> - Plan issues (1–4) → orchestrator re-calls `@prometheus` with the full Metis report.
> - Scenario issues (5–9) → orchestrator re-calls `@oracle` with the **full original PHASE 2 instructions (including isolation rules) plus the Metis report**.
>   The isolation rule ("never read `.sisyphus/plans/`") applies on every re-call without exception.
>
> After each fix, re-review all items.
> **Declare "Cross-validation complete" only when all 9 items pass.**
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

**[Compact context]** Before entering this step, summarize PHASE 0–3 logs in the format below, then immediately run `/compact` to remove the raw logs from context. Pass the summary block as the compaction hint so it remains accessible in later steps.

```
📦 Context summary — PHASE 0–3
  Context files loaded : {found/not found list}
  Prometheus plan core : {2–3 lines of key design decisions}
  Oracle scenario count: Happy Path N / Edge N / Error N / Security N
  Metis issues         : {summary of fix requests or "none"}
  Prometheus re-calls  : {N}, Oracle re-calls: {N}
```

**[Pre-check]** Before calling `ask_user_input_v0`, verify:
- Metis declared "Cross-validation complete"?
- Both `.sisyphus/plans/{keyword}.md` and `.sisyphus/tests/{keyword}.md` exist?

Wait until both conditions are met before continuing.

**[4-1] Print plan summary**

Print before calling `ask_user_input_v0`:

```
📋 Plan summary
─────────────────────────────────────
Files to create/modify:
  (list from Scope section of the plan)

Risks and notes:
  (risks flagged by Metis or Prometheus, or "none")

Test scenario count:
  Happy Path N / Edge Case N / Error Case N / Security N / Total N

Shared utilities to reuse:
  (items from reuse plan, or "none")
─────────────────────────────────────
Full plan: .sisyphus/plans/{keyword}.md
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

> Read `.sisyphus/tests/{keyword}.md` and write test code with 1:1 mapping to each TC-NNN.
>
> **⚠️ Implementation isolation**: Do NOT open or read the new/modified files listed in the Scope section of `.sisyphus/plans/{keyword}.md`.
> Write tests based solely on the `Expected results (pseudo-assertions)` in the spec.
> Referencing implementation internals (function signatures, class structure, variable names) ties tests to implementation and violates SDV.
>
> **[Required constraints]**
> - Save test files to the path in the "Test output path" of the Scope section.
> - Add the corresponding scenario ID as a comment at the top of each test case:
>   ```
>   // TC-001: Normal user creation
>   ```
> - Translate `assert A === B` to `expect(A).toBe(B)` or the equivalent framework assertion.
>   Do NOT change the **meaning** of assertions. Translation is purely syntactic.
> - **Do NOT run tests yet.** Writing only.

---

## PHASE 8 — Validate spec↔test translation (Momus)

Immediately after test writing completes, call `@momus` with:

> Read `.sisyphus/tests/{keyword}.md` and the test files at the Scope's test output path side by side.
> Verify that every pseudo-assertion was translated accurately, per TC-NNN.
>
> #### Validation criteria
>
> For each TC, judge:
>
> 1. **1:1 coverage**: Does every TC-NNN have a corresponding test case? List any missing TCs immediately.
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
>   Fix  : Update test code to match spec
>   ```
> - Fix test code directly. **Never modify `.sisyphus/tests/{keyword}.md`.**
> - Re-verify each fixed TC.
> - Declare "Spec↔test translation validated" only when all TCs pass.
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
📋 PHASE 8 Momus fix report
─────────────────────────────────────
{fix details or "No fixes"}
─────────────────────────────────────
```

---

## PHASE 9 — Run tests and self-correction loop

**[Compact context]** Before entering this step, summarize PHASE 5–8 logs in the format below, then immediately run `/compact` to remove the raw logs from context. Pass the summary block as the compaction hint so it remains accessible in later steps.

```
📦 Context summary — PHASE 5–8
  Implemented files : {file list}
  JSDoc fixes       : {file list or "none"}
  Momus fix summary : {fixed TC list and types or "no fixes"}
```

After Momus's validation, the orchestrator declares the **active Scope**:

```
▶ PHASE 9 entry — Active Scope
   Modifiable files:
     - {new files + allowed modifications from Scope section}
   Test output path: {from Scope section}
   Excluded TCs: none
```

Use this declared Scope for all Momus calls and `delegate_task` instructions within this step.

Run the tests.

- **If all TCs pass, proceed to PHASE 10 immediately.**
- If failures occur, follow the self-correction loop below.

**Self-correction loop:**

1. Collect failed TC list and error logs.
2. Call `@momus` with:
   > These TCs failed during test execution. Compare against `.sisyphus/tests/{keyword}.md` spec and classify each as **"feature bug"** or **"test bug"**. State the reason for each judgment.
   > Failed TCs and error logs: {list}
3. Branch on Momus's verdict:
   - **Feature bug** → delegate via `delegate_task(subagent_type="deep")`:
     > Fix only the **feature code** causing the test failure. Never touch test code.
     > Target TCs: {TC ID list}, error logs: {messages}
     > **[Scope constraint]** Only modify files in the active Scope declaration.
     > If you need to modify an out-of-scope file, stop and report to the orchestrator.
   - **Test bug** → delegate via `delegate_task(subagent_type="deep")`:
     > Fix only the **test code bug**. Never touch feature code or the scenario spec.
     > Target TC: {TC ID}, bug: {Momus's reason}
     > After fixing, have Momus re-verify assertion meaning preservation for this TC.
     > If meaning preservation fails, count this as a retry and return to step 1 of this loop.
4. Re-run tests after each fix.
   - **If all TCs pass, exit the loop and proceed to PHASE 10.**
5. **Escape condition**: If the **same TC ID has been retried more than 3 times**, mark it as an escape target and escalate via `ask_user_input_v0`:

```
⚠️  Self-correction loop escape — user intervention required
─────────────────────────────────────
Escaped TCs     : (TC IDs exceeding 3 retries)
Last error      : (error message summary)
Momus verdict   : (feature bug / test bug)
Retry count     : (per TC)
─────────────────────────────────────
```

- **Question**: "Auto-correction failed. How would you like to proceed?"
- **Options**: `Continue excluding escaped TCs` / `Abort workflow`
- If continue: add escaped TCs to the active Scope's exclude list and **resume the loop**. If no remaining failures, move to PHASE 10. Record unresolved failures explicitly in PHASE 11.
- If abort: terminate the workflow.

---

## PHASE 10 — Refactor

Delegate via `delegate_task(subagent_type="deep")` with:

> Read the `/refactor` skill file first, then refactor following its guidelines.
> Also remove all unused imports and variables across the source.
> **[Scope constraint — strict]** Only refactor files listed in the File Scope section of `.sisyphus/plans/{keyword}.md`.
> **Exclude test files** (the "Test output path" files) from refactoring — Momus's spec↔test validation is complete and modifying them risks corrupting assertion meaning.
> If you find improvements needed in out-of-scope files, do NOT make changes — note them separately.
> If an out-of-scope file modification is truly required, stop and report to the orchestrator. The orchestrator will request user approval via `ask_user_input_v0` before allowing it.
> After refactoring, **re-run tests only if test code exists**. If not, skip this check.
> **If any test fails, stop and report the failed TC list and error logs to the orchestrator.**

If test failures are reported after refactoring, **re-enter the PHASE 9 self-correction loop**.
Before re-entry, the orchestrator re-declares the active Scope:

```
🔁 PHASE 9 re-entry — Active Scope (refactored files only)
   Modifiable files:
     - {files actually modified during refactoring}
   Out-of-scope: all files not in the list above (never modify)
   TC counter  : reset to 0 (refactoring failures have different root causes)
   Excluded TCs (cannot re-verify):
     - {TC IDs escaped in PHASE 9 — "none" if none}
     ※ If excluded TCs fail again, skip them and handle only the remaining TCs.
```

---

## PHASE 11 — Save context memory

Call `@document-writer` with:

> Call the `memory-manager` skill to permanently store this session's work.
> Record every item below without omission:
>
> - Core requirement summary for this task
> - List of files created/modified with paths
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

**[Pre-check]** Start only after PHASE 11 memory save is confirmed complete. PHASE 12 references PHASE 11's troubleshooting details to extract prevention rules — order must be guaranteed.

Call `@document-writer` with:

> Extract AI mistakes and incorrect implementations from this entire workflow and record them in `TROUBLE_SHOOT.md` at the project root.
>
> #### What to extract
>
> Record only these types — exclude normal design decisions or user requirement changes:
> - Code incorrectly implemented by AI that required fixes
> - Repeated error patterns in the self-correction loop
> - Gaps or errors flagged by Metis or Momus
> - Assertion translation errors found in spec↔test validation
>
> #### Format
>
> Write each item as below. Detailed problem/cause/fix info is in memory — do not duplicate it here.
>
> ```markdown
> ## [YYYY-MM-DD] {task keyword}
>
> - {one-line rule or checkpoint to prevent recurrence}
> - {add lines if multiple items}
> ```
>
> #### File handling
>
> - If `TROUBLE_SHOOT.md` already exists, keep existing content and **prepend** the new entry.
> - If it does not exist, create it.
> - If there are no troubleshooting items from this workflow, skip this step.

---

## PHASE 13 — Final report

Print the final report and terminate the workflow:

```
✅ /implement workflow complete

📌 keyword          : {keyword}
📋 plan             : .sisyphus/plans/{keyword}.md
🧪 test scenarios   : .sisyphus/tests/{keyword}.md
🧾 test code        : {test output path from Scope}  (n/a if skipped)
🔍 spec↔test check  : done (n/a if skipped) / Momus fixes: {N or "none"}
🔧 refactor         : done
💾 memory saved     : done
📝 troubleshooting  : done (skipped if nothing to record)

All tasks completed successfully.
```
