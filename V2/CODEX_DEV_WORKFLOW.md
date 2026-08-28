# Codex Phase Planning → Slice Implementation Workflow

## Purpose

Use a two-role workflow for normal software development:

1. A **Phase Planning Agent** owns one coherent development phase.
2. A **fresh Slice Implementation Agent** owns each bounded implementation slice.

The workflow is designed to preserve architectural oversight while keeping implementation contexts small and avoiding duplicated discovery and verification.

The Planning Agent owns:

- Scoped repository assessment.
- Architecture and dependency review.
- Risk and data-integrity analysis.
- Phase planning and slice ordering.
- Bounded implementation-prompt creation.
- Direct review of implementation diffs and evidence.
- Corrective diagnosis.

The Implementation Agent owns:

- One approved slice only.
- The code changes required for that slice.
- Focused implementation-time verification.
- Final completion verification for the slice.
- A compact implementation report and recommended review reasoning.

Outside the page-scoped `Design:` exception, the roles remain separate.

---

## Reasoning-Effort Scale

Use four reasoning levels:

### Low

Use for mechanical, isolated, low-risk work with an obvious implementation path.

Examples:

- Copy or text changes.
- Simple styling changes.
- Small test expectation updates where behaviour is already established.
- Mechanical renames of non-public/internal symbols.
- Straightforward configuration or formatting adjustments with no behavioural risk.

### Medium

Use for normal bounded implementation or review using established project patterns and local design choices.

Examples:

- Typical UI or API feature slices.
- Local business-logic changes.
- Small service/controller/component changes.
- Straightforward bug fixes with a clear cause.
- Routine implementation-prompt creation for an already-approved, well-understood slice.

### High

Use when the slice has meaningful complexity, cross-file reasoning, contract implications, difficult debugging, or a larger local blast radius.

Examples:

- Cross-layer changes with established architecture.
- Non-trivial domain rules or lifecycle behaviour.
- Complex query or persistence changes without destructive migration risk.
- Public contract changes that are deliberate and bounded.
- Corrective work where several plausible causes must be distinguished.
- Reviews of substantial multi-file implementations.

### Extra High

Use when correctness depends on broad or deep reasoning, the blast radius is high, or failure could affect security, data, money, production, or architecture.

Use Extra High by default for:

- The initial `Plan:` for every new phase.
- Authentication and authorization.
- Security and tenant/organisation isolation.
- Database migrations or high-risk data-integrity changes.
- Architecture decisions and risky refactors.
- Deployment and production configuration.
- Payments or financial side effects.
- External integrations with writes, irreversible actions, or operational side effects.
- Concurrency, locking, idempotency, retry semantics, or race-sensitive lifecycle work.
- Difficult corrective diagnosis after repeated or ambiguous failures.
- Review of any implementation whose actual changes materially enter one of these areas.

When uncertain between two adjacent levels, choose the higher level. Reasoning recommendations are advisory and may always be escalated by the user.

### Reasoning floors

These are safety floors, not targets:

- If a slice **materially changes** authentication, authorization, security, tenant isolation, a database migration, high-risk data integrity, production/deployment behaviour, payments, an external write/side effect, concurrency, locking, idempotency, or race-sensitive lifecycle behaviour, **Implementation and Review must be Extra High**.
- If a slice materially changes a public contract or spans several layers with non-trivial behaviour, **Implementation and Review should be at least High** unless the change is demonstrably mechanical.
- Repeated ambiguous failures, materially incomplete verification, or a substantial divergence from the approved prompt raises **Review to at least High**.
- A mechanical edit located inside a high-risk subsystem may use a lower level only when it does not alter the subsystem's high-risk semantics and the recommendation states why.

---

## Task Lifecycle

### Planning Agent lifecycle

- Start a **fresh Planning Agent task for each phase**.
- Start the phase with `Plan:` at **Extra High** reasoning.
- Keep that Planning Agent for prompt creation, implementation review, corrections, and all accepted slices in the phase.
- Close the Planning Agent when the phase is complete.
- Start the next phase in a fresh Planning Agent task at Extra High.

A phase should be coherent enough that its slices share architectural context and decisions. Do not keep one Planning Agent indefinitely across unrelated phases.

Context compaction is not a failure, but it is a signal that the task has accumulated substantial history. Prefer to finish the current phase rather than carrying a heavily compacted planner into an unrelated next phase.

### Implementation Agent lifecycle

- Start a **fresh Implementation Agent task for every new slice**.
- A later slice may depend on code created by an earlier slice, but it should learn that state from the live repository and the new bounded prompt, not from the previous slice's conversation history.
- Reuse the same Implementation Agent task only when correcting, completing, or re-verifying the **same unaccepted slice**.
- Once the Planning Agent accepts the slice, close that Implementation Agent task.
- The next slice always gets a fresh Implementation Agent task.

---

## Normal Loop

```text
Fresh Phase Planning Agent — Extra High
    ↓
Plan: <phase goal>
    ↓
Scoped repository inspection
    ↓
Phased plan with bounded slices
Each slice includes recommended reasoning for creating its prompt
    ↓
User selects the next slice and sets the planner to that recommended level
    ↓
Prompt:
    ↓
Planning Agent creates one bounded prompt
Prompt includes recommended Implementation Agent reasoning
    ↓
Fresh Implementation Agent task for this slice
    ↓
Implement + focused iteration + final verification
    ↓
Implementation report
Report includes recommended Review reasoning
    ↓
Same Phase Planning Agent
User sets the planner to the recommended Review reasoning
    ↓
Review:
    ↓
Inspect actual diff and evidence
Accept / Continue / Correct / Revert
    ↓
If Correct:
    Planning Agent creates a bounded corrective prompt
    ↓
    Return to the SAME Implementation Agent task for this slice
    ↓
    Correct + verify + report new Review reasoning
    ↓
If Accepted:
    close this Implementation Agent task
    ↓
Prompt: next slice
    ↓
Fresh Implementation Agent task
    ↓
...
    ↓
Phase complete
    ↓
Close Planning Agent
    ↓
Fresh Planning Agent — Extra High for next phase
```

---

## Planning-Agent Efficiency Rules

The Planning Agent has direct repository access, but repository access must remain scoped.

Before inspecting files:

1. State the phase or review boundary.
2. Identify likely entry points.
3. Search for precise routes, symbols, services, components, entities, tests, configuration keys, or error messages.
4. Read only the strongest matches and direct dependencies.
5. Stop once there is sufficient evidence for the current planning or review decision.

Do not:

- Read the repository sequentially.
- Reopen unchanged files that already supplied sufficient context.
- Re-read `README`, `MEMORY`, workflow, project, or configuration files on every turn unless they changed or a concrete question requires them.
- Re-discover architecture during every `Prompt:` or `Review:` turn when the phase already established it.
- Turn a focused phase or review into a general audit.
- Inspect unrelated projects or layers.
- Carry speculative findings forward as confirmed facts.

Maintain a concise current phase state:

- Accepted architectural decisions.
- Completed slices.
- Current slice.
- Deferred or rejected items.
- Material risks that remain active.

Prefer repository paths and symbols over pasting large file contents into responses.

---

## Trigger: `Plan:`

Start every new phase plan at **Extra High** reasoning.

When the user says `Plan:`:

1. Restate the goal and define the inspection boundary.
2. Perform targeted repository inspection.
3. Describe current behaviour and the relevant execution path.
4. Identify affected layers, contracts, data flows, lifecycle rules, and ownership boundaries.
5. Identify material risks, assumptions, missing decisions, and existing conventions.
6. Build a phased plan using small, reviewable implementation slices.
7. Give each slice a clear completion condition and verification expectation.
8. For each slice, recommend the reasoning effort to use later when asking the Planning Agent to create that slice's `Prompt:`.
9. Do not edit code.
10. Do not create the implementation prompt until the user asks for `Prompt:` unless both were explicitly requested together.

### Plan-time command policy

Planning is repository analysis, not implementation verification.

Do **not** run builds, tests, lint, migrations, `diff --check`, or broad runtime verification during `Plan:` by default.

Run a focused command only when execution is necessary to:

- Establish a disputed or unknown baseline.
- Reproduce behaviour that cannot be established reliably from the code and existing tests.
- Resolve a material uncertainty that changes the plan.
- Confirm an environment or generated-artifact dependency that materially affects slice ordering.

When a plan-time command is necessary, state why it is needed and use the narrowest useful command.

### Required slice format

Each planned slice should contain:

```text
Slice N — <name>
Goal:
- ...

Scope:
- ...

Likely paths/symbols:
- ...

Completion condition:
- ...

Verification expectation:
- ...

Recommended reasoning to create this slice's Prompt: <Low|Medium|High|Extra High>
Reason: <one concise sentence>
```

The prompt-creation recommendation is about the difficulty and risk of translating the approved slice into a safe bounded implementation prompt. It is not yet the Implementation Agent reasoning recommendation.

Finish with `Plan ready.`

---

## Trigger: `Prompt:` or `Prompt Step N:`

Before the user asks for `Prompt:`, they should normally set the Planning Agent to the prompt-creation reasoning recommended for that slice in the phase plan.

The Planning Agent should use the existing phase plan and accepted repository state. Do not repeat broad discovery merely because a new prompt is being created. Inspect only if the repository changed in a way that materially affects the slice or a required detail remains uncertain.

Create exactly one bounded prompt for a **fresh Implementation Agent task**.

Every implementation prompt must include:

```text
Recommended implementation reasoning: <Low|Medium|High|Extra High>
Reason: <one concise sentence>

Implement <one bounded slice> in the live repository.

Goal:
- ...

Scope:
- ...

Relevant paths and symbols:
- ...

Required behaviour:
1. ...
2. ...

Existing conventions to preserve:
- ...

Risks:
- ...

Do not:
- Broaden scope.
- Implement later slices.
- Re-plan the architecture.
- Perform unrelated cleanup.
- ...

Verification:
- Use focused checks while iterating.
- Run the named completion checks once the slice is stable.
- Do not rerun an unchanged passing command unless a later change could invalidate it.
- ...

Report:
- Files and symbols changed.
- Behaviour before and after.
- Behaviour intentionally preserved.
- Verification commands and results.
- Warnings, skipped checks, blockers, or existing failures.
- Confirmation that no later slice or unrelated cleanup was implemented.
- Recommended review reasoning: <Low|Medium|High|Extra High>, based on the actual completed implementation.
- One concise reason for that review recommendation.
- End with: `Please pass this report to the Planning Agent for Review: using <level> reasoning.`

Suggested next slice:
- <name only; not part of this implementation>
```

### Choosing implementation reasoning

Choose the implementation level from the reasoning scale above based on the approved slice itself.

Do not use Extra High merely because the phase was planned at Extra High. A well-bounded implementation can legitimately be Medium even when its parent phase required Extra High architectural planning.

Finish with `Ready to run in a fresh Implementation Agent task.`

---

## Slice Implementation Agent Rules

The Implementation Agent receives one approved prompt and owns only that slice.

It should:

1. Confirm the bounded scope internally.
2. Inspect the named files and only directly necessary dependencies or nearby examples.
3. Follow existing conventions.
4. Implement the smallest useful diff.
5. Preserve behaviour outside the approved change.
6. Use the narrowest relevant verification while iterating.
7. Run the prompt's completion verification once the slice is stable.
8. Return a compact implementation report.
9. Recommend the reasoning level for the Planning Agent's review based on the **actual implementation**, not only the original prompt.

It must not:

- Redefine the plan.
- Select a new architecture independently.
- Broaden scope.
- Implement the next slice early.
- Search for unrelated defects or cleanup.
- Read unrelated projects merely because they are present.
- Rename public contracts, routes, DTOs, files, classes, or symbols unless required.
- Change dependencies, generated files, lock files, migrations, or public contracts unless the approved slice requires it.
- Perform destructive operations without explicit instruction.

If the live repository materially contradicts the prompt, inspect only enough to prove the mismatch and report the exact blocker rather than inventing a replacement plan.

### Implementation verification policy

During implementation:

- Prefer focused tests/checks that directly cover the code being changed.
- After a focused failure, diagnose and fix before broadening verification.
- Do not rerun a passing command after a change that cannot affect its result.
- Avoid repeated full builds or full test suites after every small edit.
- Once the slice is stable, run the required broader completion verification exactly as specified unless a failure requires another correction cycle.
- Report environmental or sandbox failures distinctly from code failures.
- Do not hide existing warnings or failing tests; identify whether they pre-date the slice when that can be established.

### Review-reasoning recommendation

At the end of implementation, recommend the Planning Agent review level using the actual diff, failures encountered, and verification evidence.

Escalate the review recommendation when the implementation:

- Touched more layers or public contracts than expected.
- Required architecture-sensitive judgement.
- Changed auth, authorization, security, tenancy, migration, data-integrity, production, payment, integration-side-effect, concurrency, or lifecycle behaviour.
- Encountered repeated or ambiguous failures.
- Required a materially different implementation from the prompt.
- Left skipped, partial, or uncertain verification.

Do not recommend a high level merely because many files changed mechanically. Base the recommendation on reasoning difficulty and risk.

---

## Implementation Report Format

```text
IMPLEMENTATION REPORT

Goal completed:
- Brief result.

Files changed:
- `path/to/file` — change.

Behaviour changed:
- Intentional behaviour changes, or `None`.

Behaviour intentionally preserved:
- Important preserved behaviour.

Verification:
- Command:
- Result:
- Existing warnings or skipped checks:

Scope confirmation:
- Confirm no later slice or unrelated cleanup was implemented.

Risks or unresolved issues:
- Issues encountered, or `None`.

Recommended review reasoning: <Low|Medium|High|Extra High>
Reason:
- One concise sentence based on the actual implementation, risk, and verification history.

Suggested next slice:
- State the Planning Agent's named next slice, or `Not specified`.
- Confirm it was not implemented.

Please pass this report to the Planning Agent for Review: using <level> reasoning.
```

Do not include large code dumps unless specifically requested.

---

## Trigger: `Review:`

The user should normally set the Phase Planning Agent to the review reasoning recommended by the Implementation Agent before sending `Review:`.

The recommendation is advisory. The user or Planning Agent may escalate if the diff reveals higher risk than the Implementation Agent recognised.

When the user says `Review:` and supplies the implementation report:

1. Inspect `git status` and the actual diff.
2. Inspect all changed files relevant to the slice.
3. Inspect directly affected contracts, tests, and dependencies where needed.
4. Compare the implementation against the approved prompt and phase plan.
5. Check scope, conventions, behaviour preservation, public contracts, authorization, tenancy, data integrity, migrations, concurrency, lifecycle behaviour, error handling, and deployment implications as relevant.
6. Assess whether the Implementation Agent's reported verification is appropriate and sufficient.
7. Decide one of:
   - **Accept**
   - **Accept with adjustments**
   - **Continue**
   - **Correct**
   - **Revert**
8. Separate blocking defects from optional future improvements.
9. Do not accept an implementation merely because it builds.
10. Do not edit code.

### Review verification policy

The Planning Agent performs an **independent code review**, but it must not duplicate implementation verification without a concrete reason.

Do **not** rerun passing build, test, lint, migration, or `diff --check` commands by default.

Treat the implementation report as a summary and the repository diff as authoritative. The reported verification is evidence that must be assessed, not automatically repeated.

Rerun the narrowest relevant verification only when one or more of these conditions applies:

- A required command failed, was skipped, or was not run.
- The reported command does not actually cover the changed behaviour.
- The diff reveals a likely defect or an untested branch.
- Tests were weakened, removed, or changed in a way that makes their evidence questionable.
- The implementation materially diverged from the approved prompt.
- High-risk auth, security, migration, data-integrity, concurrency, production, payment, or integration behaviour warrants independent confirmation.
- A corrective change was made after the reported passing result and could invalidate it.
- Verification output is ambiguous or inconsistent with the repository state.

When rerunning verification, state the concrete reason and use the smallest command that resolves the concern.

### Review output

Report:

- Decision.
- Blocking findings, if any.
- Important non-blocking observations only when useful.
- Whether implementation verification was accepted as sufficient or independently rerun, and why.
- If correction is required, the smallest safe corrective scope.

If accepted, finish with `Review complete: accepted.`

If correction is required, finish with `Review complete: correction required.`

---

## Trigger: `Fix:`

Use `Fix:` when the implementation, verification, or planning review identifies a blocking defect in the current slice.

The Phase Planning Agent should:

1. Inspect the failing implementation, relevant diff, error output, and affected execution path.
2. Identify the root cause rather than only treating the visible symptom.
3. Decide whether the original slice and architecture remain valid.
4. Preserve accepted portions of the implementation.
5. Create one bounded corrective prompt for the smallest safe correction.
6. Require focused verification of the original failure plus only the relevant regression checks.
7. Recommend the Implementation Agent reasoning for the correction.

Send the corrective prompt back to the **same Implementation Agent task** because the slice is not yet accepted.

The Implementation Agent should correct, verify, and emit a new implementation report with a fresh recommended review reasoning level.

Do not use a corrective task as an opportunity for unrelated cleanup or redesign.

---

## Trigger: `Refactor:`

Use the Phase Planning Agent to assess refactors before implementation.

The Planning Agent should:

1. Inspect the requested area and direct consumers.
2. Map current responsibilities, coupling, duplication, and dependency direction.
3. Distinguish demonstrated structural problems from style preferences.
4. Separate structural changes from behaviour changes.
5. Select the smallest safe buildable first slice.
6. Define behaviour-preservation checks.
7. Recommend prompt-creation reasoning for that slice.
8. Wait for `Prompt:` before creating the implementation prompt.

Avoid arbitrary file-length targets, broad rewrites, premature abstractions, and simultaneous structural and behavioural changes.

---

## Broad Audit Guidance

Broad audits are valid, but they should remain evidence-driven and risk-ranked.

Recommended approach:

1. Inspect solution/project structure without opening all source files.
2. Identify relevant entry points, dependency registration, authentication, persistence, background processing, integrations, and tests.
3. Use risk-ranked targeted searches to sample and trace representative paths.
4. Expand only when evidence suggests a systemic issue.
5. Produce an issue register grouped by severity, confidence, affected paths, and proposed slice.
6. Avoid cosmetic cleanup unless it represents a demonstrated maintenance or correctness problem.

The goal is not token-level coverage of every file. The goal is enough repository evidence to identify material risks and safe slices.

---

## Page-Scoped `Design:` Exception

`Design:` is the deliberate exception to the separate Planning Agent → Implementation Agent handoff.

For one named page or component:

- Keep design exploration, user feedback, approval, implementation, and rendered-page correction in the same Codex task.
- Inspect only the target page/component, directly relevant components, and existing theme/design primitives.
- Ask only missing high-impact intake questions: audience, page purpose, what feels wrong, what must remain unchanged, useful inspiration/brand material, and styles the user dislikes.
- Infer low-risk details from the repository rather than forcing a full questionnaire.
- Preserve functionality unless the user explicitly approves behaviour changes.
- Propose a compact visual direction before editing: palette, typography, layout, hierarchy, and one appropriate signature element.
- Wait for explicit approval or an explicit implementation request.
- Implement only the approved page/component and directly necessary shared styling primitives.
- Use the installed frontend-design skill when available.

### Rendered-page quality gate

After implementation and relevant source checks:

1. Inspect the actual page in the Codex in-app browser when available.
2. Check representative widths appropriate to the project's breakpoints; normally around 390px, 768px, 1024px, and 1440px.
3. Inspect hierarchy, wrapping, alignment, spacing, overflow, clipping, overlap, contrast, images/icons, and directly relevant interactive states.
4. Use computed styles or measured layout when source inspection is insufficient.
5. Correct glaring page-scoped issues that remain within the approved direction.
6. Reinspect affected states after material corrections.
7. Ask before materially changing visual direction, behaviour, scope, or another page/component.
8. If browser access is unavailable, report visual verification as incomplete rather than treating a successful build as visual proof.

After acceptance, promote visual decisions into `DESIGN_SYSTEM.md` only with explicit user approval. Distinguish shared primitives from page-specific layout or signature elements.

If design work requires material behaviour, data-flow, route, backend, or contract changes, move those changes through the normal Phase Planning Agent → Slice Implementation Agent workflow unless the user explicitly expands the design scope.

---

## Completion Standard

A slice is complete only when:

- The approved behaviour is implemented.
- Unrelated behaviour is preserved.
- The diff remains within scope.
- Required verification passes, or failures are explicitly reported.
- High-risk work includes the relevant safety and compatibility checks.
- The Phase Planning Agent has reviewed and accepted it.

A successful build alone is not sufficient evidence of correctness.

A phase is complete when all accepted slices required for the phase goal are complete, deferred items are recorded, and the Planning Agent can be closed without carrying unresolved phase state into the next phase.
