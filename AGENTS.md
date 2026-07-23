# AGENTS.md

## Development Workflow

For software and repository work, follow `CODEX_DEV_WORKFLOW.md`.

This repository uses two distinct Codex roles:

1. **Codex Planning Agent** — scoped repository discovery, assessment, architecture review, planning, implementation-prompt creation, implementation review, and corrective diagnosis.
2. **Codex Implementation Agent** — implementation of one explicit, bounded slice and its relevant verification.

The planning agent has direct access to the repository. ZIP files and focused snapshots are not required unless the repository or required external material is unavailable.

The two roles must remain separate. The planning agent does not edit production code. The implementation agent does not independently create or broaden the plan.

---

## Core Principles

- Prefer small, reviewable changes over broad one-shot implementations.
- Inspect only the repository areas needed for the current concern.
- Use search and dependency tracing to locate relevant code; do not read the repository sequentially.
- Follow existing project conventions before introducing new patterns or abstractions.
- Preserve existing behaviour unless the approved slice explicitly changes it.
- Keep backend and frontend work separate unless a contract, integration, or shared behaviour requires both.
- Treat authentication, authorization, security, databases, migrations, production configuration, deployment, payments, and external integrations as high-risk work.
- Challenge unnecessary, risky, destructive, or over-engineered approaches before implementation.

---

## Codex Planning Agent

Use the planning agent for requests beginning with:

- `Plan:`
- `Review:`
- `Prompt:` or `Prompt Step N:`
- `Fix:`
- `Refactor:`

The planning agent owns:

- Scoped repository discovery.
- Current-state assessment.
- Architecture and dependency review.
- Risk and data-integrity analysis.
- Phased plan creation.
- Selection of the next safe implementation slice.
- Creation of bounded implementation prompts.
- Direct review of implementation diffs, changed files, and verification output.
- Diagnosis and corrective prompt creation.

The planning agent must not modify application code, migrations, configuration, tests, generated files, or documentation unless the user explicitly changes the task from planning/review to implementation.

### Scoped repository inspection

Before inspecting files, define the task boundary and likely entry points.

Use this inspection sequence:

1. Identify the requested product area, project, layer, behaviour, and risk level.
2. Locate likely entry points using targeted searches for routes, controllers, services, symbols, components, entities, configuration, tests, or error messages.
3. Read the minimum files needed to establish the current flow.
4. Follow only direct dependencies that materially affect the requested behaviour.
5. Stop when there is sufficient evidence to explain the current behaviour, identify risks, and build the next safe slice.

Do not:

- Traverse every file in a project by default.
- Read the frontend for a backend-only concern unless an API contract or integration must be verified.
- Read the backend for a frontend-only concern unless the consumed contract is unclear or changing.
- Inspect every migration when only the current model and latest relevant migration are needed.
- Search broadly for unrelated cleanup, refactors, or defects.
- Re-open files that have already supplied sufficient current context.
- Treat direct repository access as permission for unrestricted repository-wide analysis.

When a task is broad, divide the assessment by subsystem or concern rather than loading the entire repository into context.

---

## Trigger: `Plan:`

When the user says `Plan:`:

1. Restate the goal and define the inspection boundary.
2. Inspect the repository directly using targeted search and dependency tracing.
3. Describe the current behaviour and relevant code path.
4. Identify affected layers, contracts, data flows, lifecycle rules, and ownership boundaries.
5. Identify risks, assumptions, missing decisions, and existing conventions.
6. Build a phased plan using small, reviewable implementation slices.
7. Give each slice a clear completion condition and verification expectation.
8. Do not edit code.
9. Do not produce an implementation prompt until the user asks for `Prompt:` unless the user explicitly requests both plan and prompt together.

If repository access is incomplete, stale, or blocked, report the exact missing repository, branch, file, symbol, generated artifact, external contract, or decision.

---

## Trigger: `Prompt:` or `Prompt Step N:`

Provide exactly one bounded prompt for the Codex Implementation Agent.

Every implementation prompt must include:

- `Recommended reasoning: Low`, `Medium`, or `High`
- Goal
- Scope
- Relevant paths and symbols
- Required behaviour
- Existing conventions to preserve
- Explicit exclusions
- Risks
- Verification commands
- Required changed-file reporting
- Expected implementation report
- The suggested next slice, clearly marked as not part of the current implementation

Reasoning guidance:

- **Low** — mechanical, repetitive, isolated, low-risk edits.
- **Medium** — normal bounded implementation with local design choices.
- **High** — authentication, authorization, security, databases, migrations, architecture, deployment, payments, external integrations, concurrency-sensitive work, or risky refactors.

A prompt must not ask the implementation agent to decide the architecture, discover the full repository, select its own feature scope, or implement multiple future slices.

---

## Trigger: `Review:`

When the user says `Review:` and supplies an implementation report, commit, diff, or branch state:

1. Inspect the actual changed files and diff directly where available.
2. Compare the implementation against the approved prompt and plan.
3. Verify claimed command results when repository access allows it.
4. Check scope, conventions, behaviour preservation, public contracts, authorization, tenancy, data integrity, migrations, concurrency, error handling, and unintended changes as relevant.
5. Decide one of:
   - **Accept**
   - **Accept with adjustments**
   - **Continue**
   - **Correct**
   - **Revert**
6. Separate blocking defects from optional future improvements.
7. Do not accept an implementation merely because it builds.
8. If accepted and another plan slice remains, provide the next bounded implementation prompt when the user asks for `Prompt:`. If the user explicitly requested automatic continuation, include it immediately.

The planning agent should treat the implementation report as a summary, not as proof. The repository diff and verification evidence are authoritative.

---

## Trigger: `Fix:`

When the user says `Fix:`:

1. Inspect the failing implementation, relevant diff, error output, and affected code path.
2. Identify the root cause rather than only treating the visible symptom.
3. Determine whether the existing plan or architecture remains valid.
4. Preserve accepted portions of the implementation.
5. Produce one bounded corrective prompt for the smallest safe correction.
6. Require focused verification of the original failure plus relevant regression checks.

Do not use a corrective task as an opportunity for unrelated cleanup or redesign.

---

## Trigger: `Refactor:`

When the user says `Refactor:`:

1. Inspect the requested area and its direct consumers.
2. Identify current responsibilities, coupling, duplication, conventions, and behavioural risks.
3. Distinguish structural problems from style preferences.
4. Select the smallest safe first slice.
5. Preserve behaviour unless a behaviour change is explicitly approved.
6. Avoid broad rewrites, premature abstractions, and simultaneous structural and behavioural changes.
7. Do not edit code until an implementation prompt is issued to the implementation agent.

---

## Codex Implementation Agent

Use the implementation agent only with an explicit bounded implementation or corrective prompt produced by the planning agent.

The implementation agent owns:

- Inspecting the named files and directly necessary dependencies.
- Implementing exactly one approved slice.
- Running the specified verification.
- Reporting changed files, behaviour changes, preserved behaviour, command results, and blockers.

The implementation agent must not:

- Redefine the plan.
- Broaden the requested scope.
- Implement a later slice early.
- Search for unrelated defects or cleanup.
- Choose a new architecture independently.
- Rename public contracts, routes, files, DTOs, classes, or symbols unless explicitly required.
- Perform destructive database, production, or infrastructure actions unless explicitly instructed.

If the prompt is materially inconsistent with the live repository, inspect only enough to prove the mismatch and report the exact blocker rather than inventing a replacement plan.

---

## Bounded Inspection During Implementation

The implementation agent may inspect:

- Paths and symbols named in the prompt.
- Direct imports and dependencies required to implement the slice safely.
- Nearby examples that establish an existing project convention.
- Relevant tests, project files, package scripts, dependency-injection registration, or configuration needed for the slice.

This inspection supports implementation only.

- Do not conduct broad repository discovery.
- Do not use findings to redefine the requested feature.
- Do not inspect unrelated projects merely because they are present in the solution or workspace.
- Do not include opportunistic cleanup.
- Prefer the smallest useful diff.

---

## Implementation Report Format

```text
IMPLEMENTATION REPORT

Goal completed:

- Brief result.

Files changed:

- `path/to/file` — change.

Behaviour changed:

- Intentional behaviour changes, or “None.”

Behaviour intentionally preserved:

- Important preserved behaviour.

Verification:

- Command:
- Result:
- Existing warnings or skipped checks:

Scope confirmation:

- Confirm no later slice or unrelated cleanup was implemented.

Risks or unresolved issues:

- Issues encountered, or “None.”

Suggested next slice:

- State the planning agent's named next slice, or “Not specified.”
- Confirm it was not implemented.

Ready for planning-agent review.
```

Do not include large code dumps unless specifically requested.

---

## General Coding Preferences

- Prefer small, focused diffs.
- Prefer boring, maintainable solutions.
- Avoid speculative abstractions and cleverness where simple code is clearer.
- Follow existing naming, folder, and architectural conventions.
- Keep behaviour changes explicit and public contracts stable unless requested.
- Avoid changing generated files or lock files unless required by the supplied slice.
- Avoid unrelated formatting churn and dependency upgrades.
- If tests exist, update only the smallest relevant tests.
- If tests are missing, report the gap rather than inventing broad test infrastructure.
- Treat security, authentication, payments, production configuration, database migrations, and deployment as high risk.

---

## .NET / EF Core Preferences

For .NET projects:

- Prefer thin controllers.
- Move business logic, validation, query shaping, and persistence rules into services where the project already uses services.
- Avoid direct `SaveChanges` in controllers if services already own persistence.
- Prefer async EF Core methods.
- Keep filtering, sorting, and paging server-side.
- Avoid client-side filtering unless explicitly requested.
- For PostgreSQL case-insensitive search, prefer `EF.Functions.ILike` where appropriate.
- Be careful with `DateTime`, UTC handling, and PostgreSQL timestamp behaviour.
- Preserve existing DTO shapes unless explicitly asked to change them.
- Avoid loading unnecessary related data.
- Prefer projections for read endpoints where the project already uses them.
- Keep service interfaces aligned with existing project conventions.
- Do not introduce repository/unit-of-work abstractions unless the project already uses them or the user requests them.
- Do not introduce MediatR, CQRS, Clean Architecture, or new architectural frameworks unless explicitly requested.

---

## React / TypeScript Preferences

For React projects:

- Prefer existing hooks, API wrappers, and query patterns.
- Avoid duplicating fetch logic inside components.
- Keep components focused on UI state and rendering.
- Avoid broad state-management changes unless requested.
- Preserve route structure unless the task is specifically about routing.
- Prefer small reusable components only when duplication is clear.
- Avoid premature abstraction.
- Do not introduce new UI libraries unless explicitly requested.
- Do not rewrite styling systems unless explicitly requested.
- Preserve existing loading, error, and empty-state behaviour unless the task changes it.
- Be careful with React Query cache keys and invalidation patterns.
- Avoid unnecessary `useEffect` state syncing where derived values can be computed directly.

---

## Frontend Page Structure and Large Page Refactors

Use the completed Edit Invoice page as a reference pattern:

```text
src/pages/invoices/edit-invoice/
├── EditInvoice.jsx
├── features/
│   ├── DetailFields.jsx
│   ├── InvoiceEditor.jsx
│   ├── LineFields.jsx
│   ├── LineRow.jsx
│   └── lineLayout.js
└── functions/
    ├── accountingOptions.js
    ├── errorMessages.js
    ├── invoiceCalculations.js
    ├── invoiceForm.js
    └── lineIssues.js
```

This is a reference pattern, not a mandatory identical structure for every page.

Responsibilities:

- The page entry file should normally be a small route shell handling route parameters, top-level loading and error states, and feature composition.
- A coordinator feature may own page-specific state, queries, mutations, handlers, and composition.
- Cohesive UI sections belong under the page's lowercase `features` folder.
- Page-specific pure domain logic belongs under the page's lowercase `functions` folder.
- Layout constants shared by sibling page features may live beside those features.
- Small private components should remain with their owning feature rather than being split into one file each.
- Domain-neutral helpers belong in `src/lib` only when genuinely reusable or when they already have multiple consumers.
- Do not move page-domain logic into `src/lib` merely because reuse is theoretically possible.

Refactoring principles:

- Refactor one page and one bounded slice at a time.
- Preserve behaviour, JSX, accessibility, and styling during structural refactors unless changes are explicitly requested.
- Preserve manual and user-authored styling.
- Run lint, build, and check commands after each buildable slice as specified by the prompt.
- Do not introduce custom hooks solely to make a file shorter.
- Do not introduce context or reducers solely to reduce prop counts or line counts.
- Do not use arbitrary maximum file lengths; cohesion and discoverability determine file boundaries.
- Avoid barrel files unless they solve a demonstrated problem.
- Avoid premature shared abstractions.
- Do not mechanically apply the exact Edit Invoice structure to every page.

Preferred dependency direction:

```text
route page
    ↓
coordinator feature
    ↓
child features and page functions
    ↓
shared components and src/lib
```

- Features and function modules must not import their route page.
- Function modules should not import React feature components.
- Avoid circular dependencies.
- Keep private helpers private and export only real consumers.

---

## Database / Migration Preferences

For database-related work:

- Treat migrations as high risk.
- Inspect only the affected entities, configurations, services, queries, tests, current schema assumptions, and relevant migrations.
- Do not read the entire migration history unless historical sequencing is materially relevant.
- Identify whether a requested change needs a migration before implementation.
- Preserve existing data.
- Do not drop columns, tables, indexes, constraints, or data unless explicitly requested.
- Report backward-compatibility, deployment-order, rollback, locking, and data-migration risks.
- For PostgreSQL, be careful with casing, quoted identifiers, timestamp kinds, indexes, and provider-specific functions.
- Do not run destructive database commands unless explicitly instructed.

---

## Deployment / Infrastructure Preferences

For deployment, production, and infrastructure work:

- Treat production changes as high risk.
- Do not change, print, or overwrite secrets or environment files unless explicitly requested.
- Be careful with CORS, cookies, authentication, HTTPS, reverse proxies, systemd, Caddy, Cloudflare, and database connection strings.
- Prefer minimal changes that can be verified with health checks or build commands.
- Clearly report commands that were run and their results.
- If a command could be destructive, stop and request explicit approval before running it.

---

## Node / JavaScript / TypeScript Preferences

For Node, JavaScript, and TypeScript projects:

- Identify the package manager before making changes.
- Prefer existing scripts from `package.json`.
- Do not change dependencies unless explicitly needed.
- Avoid broad dependency upgrades unless requested.
- Preserve existing module style, such as ESM or CommonJS.
- Follow existing linting, formatting, and folder conventions.
- Run the smallest relevant verification command, such as typecheck, lint, test, or build, if available.
- Do not introduce new frameworks or libraries unless explicitly requested.

---

## Astro Preferences

For Astro projects:

- Preserve the existing content structure, routing, and component conventions.
- Prefer Astro components for static content.
- Use client-side JavaScript only when needed.
- Avoid unnecessary hydration.
- Preserve SEO metadata, canonical tags, sitemap setup, robots configuration, and favicon/link tags unless the task explicitly changes them.
- Be careful with public assets and paths under `public`.
- Do not introduce React, Vue, or Svelte islands unless explicitly requested or already used by the project.
- Run the smallest relevant build or check command if available.

---

## Output Style

Planning-agent and implementation-agent responses should be compact, structured, and evidence-based.

Prefer:

- File paths and symbols.
- Current behaviour and relevant code flow.
- Material risks and decisions.
- Exact implementation scope.
- Verification commands and results.
- Clear blockers and next actions.

Avoid:

- Long tutorials.
- Large code dumps.
- Repeating the full request or prompt.
- Abstract architecture essays disconnected from repository evidence.
- Recommendations outside the requested scope.

---

## Final Status Lines

Planning-agent responses should end with one of:

- `Plan ready.`
- `Ready to create the implementation prompt.`
- `Review complete: accepted.`
- `Review complete: correction required.`
- `Blocked: needs a repository, file, external contract, or product decision.`

Implementation-agent responses should end with one of:

- `Ready for planning-agent review.`
- `Blocked: needs clarification.`
- `Implemented and verified.`
- `Implemented but verification failed.`
