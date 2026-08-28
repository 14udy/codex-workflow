# Codex Planning → Codex Implementation Workflow

## Purpose

For normal software development outside the page-scoped design workflow, use two separate Codex tasks:

1. A long-lived **Planning Agent** task with direct repository access.
2. A separate **Implementation Agent** task for each bounded implementation slice.

The planning task owns repository assessment, architecture review, planning, prompt creation, implementation review, and corrective diagnosis.

The implementation task owns only the approved code change and its verification.

Requests beginning with `Design:` use the same-task exception described below. The design task keeps visual exploration, user feedback, approval, and implementation together so the approved direction is not handed to a separate implementation task.

This workflow removes the normal need to upload repository ZIP files or focused snapshots. Direct repository access must still be used selectively and efficiently.

---

## Normal Loop

```text
User opens or returns to the Codex Planning Agent
    ↓
User sends Plan: <goal>
    ↓
Planning Agent performs scoped repository inspection
    ↓
Planning Agent explains current behaviour and creates a phased plan
    ↓
User approves the plan and sends Prompt:
    ↓
Planning Agent creates one bounded Implementation Agent prompt
    ↓
User opens a separate Codex Implementation Agent task and runs the prompt
    ↓
Implementation Agent changes code, verifies, and reports
    ↓
User pastes the report into the Planning Agent with Review:
    ↓
Planning Agent inspects the actual repository diff and verification evidence
    ↓
Planning Agent accepts, corrects, reverts, or continues
    ↓
User sends Prompt: for the next approved slice
```

Keep the Planning Agent task as the durable source of architectural context and plan state. Use a fresh Implementation Agent task when isolation would reduce context contamination or scope drift.

---

## Page-Scoped Design Workflow Exception

Use this workflow only when the user begins a request with `Design:` or explicitly asks to enter the page-design workflow.

```text
User sends Design: <page or component>
    ->
Design task inspects the current page and existing theme
    ->
Design task asks only the missing high-impact questions
    ->
Design task proposes a visual direction without editing files
    ->
User reviews, revises, and explicitly approves the direction
    ->
The same design task implements the approved direction
    ->
If needed, the task asks the user to open the page in the Codex in-app browser
    ->
The design task inspects the rendered page at representative breakpoints
    ->
The design task fixes glaring page-scoped issues and repeats the relevant checks
    ->
User reviews the implemented page
    ->
User may explicitly promote shared decisions into the project design system
```

The design workflow must:

- Stay in one Codex task through design exploration, approval, and implementation.
- Not create an implementation-agent prompt, delegate the approved direction, or hand implementation to another agent.
- Work on one named page or component at a time.
- Use the installed `frontend-design` skill when available.
- Inspect the current page, directly relevant components, and existing shared theme before proposing changes.
- Invite the user to answer any or all of the design-intake questions defined in `AGENTS.md`, including optional inspiration images, screenshots, links, brand materials, and disliked examples.
- Infer low-risk details from the repository and state material assumptions instead of forcing a complete questionnaire.
- Produce a compact design proposal before editing, including palette, typography, layout, information hierarchy, and one appropriate signature element.
- Wait for explicit approval or an explicit request to implement.
- Preserve existing functionality and keep implementation limited to the approved page plus directly necessary shared styling primitives.
- Record functional changes suggested by the design as separate future development slices unless the user explicitly adds them to scope.

### Rendered-page quality gate

An approved `Design:` implementation is not ready for user review until the design task has had the opportunity to inspect the actual page in the Codex in-app browser.

1. Run the relevant source-level checks first, but do not treat a successful check or build as visual verification.
2. If the target URL is not already available in the in-app browser, ask the user to start the local site and open the named page there. Continue in the same task once it is ready.
3. Inspect representative widths that exercise the real responsive breakpoints. The normal baseline is approximately 390px, 768px, 1024px, and 1440px, adjusted when the project's breakpoints or composition call for different widths.
4. Audit the rendered hierarchy, text wrapping, spacing, alignment, overflow, clipping, overlap, breakpoint transitions, action visibility and contrast, images, icons, and directly relevant interactive states.
5. Use computed styles and measured layout when needed to catch issues hidden by source inspection, including global CSS overriding utilities, elements that should be hidden, and collisions at exact breakpoint boundaries.
6. Compare the result with the approved direction and self-critique whether it feels intentional and subject-specific rather than generic, templated, or over-explained.
7. Correct glaring issues that stay within the approved page, visual direction, and preserved behaviour. Reinspect the affected viewports and states after every material correction.
8. Request approval before changing the visual direction materially, altering behaviour, widening scope, or redesigning another page or component.
9. Report the viewports and states inspected, the corrections made during the audit, and anything that remains unverified. If browser access is unavailable, say so and do not claim that the page was visually verified.

This quality gate is the narrow browser-testing exception for an approved page-scoped `Design:` task. It does not authorize a general smoke test or exploration of unrelated routes, forms, workflows, or integrations unless the user separately asks for one.

### Project design-system promotion

An accepted page is a candidate reference, not an automatic global template.

After the user accepts the implemented page, ask whether its shared visual direction should become the project design system. Promote it only after explicit approval.

For this repository, use the established root `DESIGN_SYSTEM.md`. Record approved design tokens, typography, component treatments, motion and accessibility principles, interface voice, source-of-truth implementation paths, and the accepted reference page. Clearly separate reusable primitives from page-specific layout or signature ideas.

Later pages should consume shared CSS variables, theme primitives, and components rather than copying page-local CSS. Adopt the approved visual language one page at a time; do not automatically redesign existing pages or mechanically repeat the reference page's composition.

If design work requires material behaviour, data-flow, route, backend, or contract changes, stop and move those changes through the normal Planning Agent to Implementation Agent workflow unless the user explicitly expands the design scope.

---

## Planning Agent Rules

The Planning Agent may read the live repository but should remain read-only unless the user explicitly requests direct implementation.

It is responsible for:

- Targeted repository discovery.
- Current-state and code-flow assessment.
- Architecture review.
- Risk analysis.
- Phased planning.
- Implementation-slice selection.
- Codex prompt creation.
- Review of reports, diffs, files, and verification output.
- Corrective diagnosis.

It must:

- Define scope before inspection.
- Use targeted search before opening files.
- Follow direct dependencies only as far as required.
- Stop inspecting once the current behaviour and safe next slice are understood.
- Separate confirmed repository facts from assumptions.
- Preserve project conventions unless a deliberate change is justified.
- Keep backend, frontend, database, infrastructure, and external integration analysis separated unless the task crosses those boundaries.

It must not:

- Read the entire repository by default.
- inspect unrelated projects or layers.
- Turn a focused task into a general audit.
- Edit code during planning or review.
- Accept an implementation solely from its report or successful build.
- Create several implementation prompts at once unless the user explicitly asks for a complete prompt sequence.

---

## Inspection Efficiency

Direct repository access replaces snapshot handling; it does not remove scope control.

### Search first

Start from precise terms such as:

- Endpoint route
- Controller or handler name
- Service or interface
- Entity or DTO
- React page or component
- Error text
- Configuration key
- Database table or migration symbol
- Test name

Use repository search to identify candidate files, then open only the strongest matches.

### Follow the execution path

For a backend request, usually inspect only the relevant path through:

```text
route/controller
    ↓
service/business rules
    ↓
entity/query/persistence
    ↓
external integration or background job, when applicable
    ↓
focused tests
```

For a frontend concern, usually inspect only:

```text
route/page
    ↓
feature/component state
    ↓
query/mutation/API wrapper
    ↓
consumed backend contract, when required
    ↓
focused tests or build scripts
```

### Database work

For database-related analysis, prefer:

- Affected entities.
- EF configurations.
- Current service/query behaviour.
- Relevant constraints and indexes.
- The latest or specifically related migrations.
- Focused tests.

Do not load the full migration history unless sequencing, historical data shape, rollback, or compatibility requires it.

### Stop conditions

Stop repository inspection when there is enough evidence to:

- Explain current behaviour.
- Identify the root concern.
- Define risks and decisions.
- Select the next bounded slice.
- Name the files and symbols the implementation agent will likely touch.
- Specify meaningful verification.

---

## Trigger: `Plan:`

Example:

```text
Plan: Audit the backend API and make a plan for any corrective slices needed.
```

The Planning Agent should first narrow broad wording into explicit audit domains. For example, a backend audit may be organised as:

1. Application structure and dependency boundaries.
2. Authorization and tenant isolation.
3. Persistence, transactions, and concurrency.
4. Background jobs and external integrations.
5. Error handling, validation, and observability.
6. Performance and unnecessary data loading.
7. Focused test coverage gaps.

It should inspect one domain at a time, prioritised by risk, rather than reading every backend file.

For any `Plan:` request, the Planning Agent should:

1. State the interpreted scope and exclusions.
2. Identify likely entry points with targeted searches.
3. Inspect only the relevant code paths.
4. Describe current behaviour with file and symbol references.
5. Identify defects, risks, assumptions, and missing decisions.
6. Separate confirmed issues from optional improvements.
7. Produce phased corrective slices ordered by safety and dependency.
8. Define verification and completion criteria for each slice.
9. Finish without editing code.

Broad audits should produce an evidence-based issue register before implementation prompts are created.

---

## Trigger: `Prompt:` or `Prompt Step N:`

The Planning Agent creates one prompt for a separate Implementation Agent.

Prompt template:

```text
Recommended reasoning: <Low|Medium|High>

Implement <one bounded slice> in the live repository.

Goal:
- ...

Scope:
- ...

Relevant paths and symbols:
- ...

Required changes:
1. ...
2. ...

Existing conventions:
- ...

Risks:
- ...

Do not:
- ...
- ...

Verification:
- ...

Report:
- Files and symbols changed.
- Behaviour before and after.
- Behaviour intentionally preserved.
- Verification command results.
- Warnings, skipped checks, or blockers.
- Confirmation that no later slice or unrelated cleanup was implemented.
- The named next slice, with confirmation that it was not implemented.
```

### Reasoning level

- **Low** — mechanical and low risk.
- **Medium** — normal bounded implementation.
- **High** — authentication, authorization, security, databases, migrations, architecture, deployment, payments, integrations, concurrency, data integrity, or risky refactors.

The prompt should reference known paths and symbols, but the implementation agent may inspect direct dependencies needed to implement safely.

---

## Implementation Agent Rules

The Implementation Agent receives one approved prompt.

It should:

1. Confirm the bounded scope internally.
2. Inspect named files and directly necessary dependencies.
3. Follow existing conventions.
4. Implement the smallest useful diff.
5. Preserve behaviour outside the approved change.
6. Run the specified verification.
7. Return a compact implementation report.

It should not:

- Re-plan the feature.
- Select a different architecture.
- Expand into adjacent cleanup.
- Implement the named next slice.
- Read unrelated projects.
- Change dependencies, generated files, lock files, public contracts, or migrations unless required by the prompt.
- Perform destructive operations without explicit instruction.

When the live repository differs materially from the prompt, it should stop after identifying the exact mismatch.

---

## Trigger: `Review:`

Paste the Implementation Agent report into the Planning Agent:

```text
Review:
<implementation report>
```

The Planning Agent should not review only the prose report. It should inspect:

- `git status`
- The actual diff
- All changed files
- Directly affected contracts and tests
- Verification output or rerun focused verification when appropriate

Review decision:

- **Accept** — implementation matches the prompt and is sufficiently verified.
- **Accept with adjustments** — implementation is safe, but a small follow-up is required.
- **Continue** — accepted and ready for the next planned slice.
- **Correct** — a bounded corrective prompt is required.
- **Revert** — implementation is unsafe or based on a wrong approach.

Review checks should be selected according to scope:

- Prompt compliance and unintended changes.
- Existing conventions.
- Security and authorization.
- Tenant or organisation isolation.
- Data integrity and migration safety.
- Transactions, concurrency, idempotency, and retries.
- Error handling and lifecycle transitions.
- Frontend/backend contracts.
- Performance and unnecessary data loading.
- Verification quality.
- Deployment and backward compatibility.

Do not block progress for unrelated improvements. Record those as possible later slices.

---

## Trigger: `Fix:`

Use `Fix:` when an implementation, build, test, or manual verification fails.

The Planning Agent should:

1. Inspect the error, diff, and affected execution path.
2. Identify the root cause.
3. Decide whether the original slice remains valid.
4. Preserve accepted work.
5. Produce one corrective prompt with focused regression checks.

The corrective Implementation Agent should make the smallest safe correction and report both the original failure and the correction.

---

## Trigger: `Refactor:`

The Planning Agent should:

1. Inspect the requested file or subsystem and its direct consumers.
2. Map current responsibilities and dependency direction.
3. Identify demonstrated problems such as duplication, mixed responsibilities, testability issues, or unsafe coupling.
4. Separate structural refactoring from behaviour changes.
5. Select the smallest buildable first slice.
6. Define behaviour-preservation checks.
7. Create the implementation prompt only after the user requests `Prompt:`.

Avoid arbitrary file-length targets, broad rewrites, premature shared abstractions, and new frameworks.

---

## Broad Audit Guidance

A request such as:

```text
Plan: Audit the backend API and make a plan for any corrective slices needed.
```

is valid for the Planning Agent, but it should remain efficient.

Recommended approach:

1. Inspect solution and project structure without opening all source files.
2. Identify application entry points, dependency registration, authentication, persistence, background processing, integrations, and test projects.
3. Use risk-ranked targeted searches to sample and trace representative code paths.
4. Expand inspection only when evidence suggests a systemic issue.
5. Produce findings grouped by severity, confidence, affected paths, and recommended slice.
6. Avoid proposing cosmetic cleanup unless it creates a demonstrated maintenance or correctness problem.

The goal is not complete token-level coverage of every file. The goal is sufficient repository evidence to identify material risks and safe corrective slices.

---

## Context and Task Management

The Planning Agent task may be long-lived, but it should keep its working context compact:

- Maintain a concise current plan and decision record.
- Refer to repository paths and symbols instead of pasting large files.
- Re-read changed files during review rather than relying on stale memory.
- Reconfirm the active branch and worktree before major reviews.
- Mark completed, deferred, rejected, and superseded slices clearly.
- Avoid carrying speculative findings forward as confirmed facts.

Outside the page-scoped `Design:` workflow, use separate Implementation Agent tasks when:

- A clean context reduces the chance of scope drift.
- The previous implementation task contains unrelated history.
- The next slice requires a different risk/reasoning level.
- The implementation should be independently reviewable.

---

## Completion Standard

A slice is complete only when:

- The approved behaviour is implemented.
- Unrelated behaviour is preserved.
- The diff remains within scope.
- Required verification passes, or failures are explicitly reported.
- High-risk work includes relevant safety and compatibility checks.
- The Planning Agent has reviewed and accepted it.

A successful build alone is not sufficient evidence of correctness.
