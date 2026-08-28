# AGENTS.md

## Development Workflow

For normal software and repository work, follow `CODEX_DEV_WORKFLOW.md`.

The repository uses two separate Codex roles:

1. **Phase Planning Agent** — repository assessment, architecture and risk review, phase planning, bounded prompt creation, implementation review, and corrective diagnosis.
2. **Slice Implementation Agent** — implementation and verification of one explicit bounded slice.

Keep the roles separate for normal `Plan:`, `Prompt:`, `Review:`, `Fix:`, and `Refactor:` work.

- Start a **fresh Planning Agent task for each phase**. Begin the phase plan at **Extra High** reasoning.
- Start a **fresh Implementation Agent task for each new implementation slice**.
- Reuse the same Implementation Agent task only for corrections or follow-up work that remains part of the same unaccepted slice.
- Once a slice is accepted, close that Implementation Agent task. The next slice gets a fresh task.
- The planning agent does not edit production code.
- The implementation agent does not redefine the plan or broaden scope.

`Design:` requests use the page-scoped same-task exception defined in `CODEX_DEV_WORKFLOW.md`.

---

## Core Repository Principles

- Prefer small, reviewable changes over broad one-shot implementations.
- Inspect only the repository areas needed for the current concern.
- Search first; follow direct dependencies only as far as necessary.
- Stop inspecting once there is enough evidence to act safely.
- Do not repeatedly reopen unchanged files that have already supplied sufficient context.
- Follow existing project conventions before introducing new patterns or abstractions.
- Preserve existing behaviour unless the approved slice explicitly changes it.
- Keep backend and frontend work separate unless a contract, integration, or shared behaviour requires both.
- Prefer the smallest useful diff.
- Avoid opportunistic cleanup, unrelated refactors, dependency upgrades, formatting churn, or speculative abstractions.
- Challenge unnecessary, risky, destructive, or over-engineered approaches before implementation.
- Do not run a general smoke test unless the user asks. The rendered-page audit for an approved `Design:` task is the narrow exception described in `CODEX_DEV_WORKFLOW.md`.

### High-risk areas

Treat the following as high-risk and apply the reasoning guidance in `CODEX_DEV_WORKFLOW.md`:

- Authentication and authorization.
- Security and tenant or organisation isolation.
- Databases, migrations, data integrity, destructive operations, and production data.
- Architecture and high-blast-radius refactors.
- Deployment and production configuration.
- Payments and financial side effects.
- External integrations with writes or irreversible side effects.
- Concurrency, idempotency, locking, retries, and lifecycle transitions.

---

## Scoped Repository Inspection

Before reading files, define the task boundary and likely entry points.

Use this sequence:

1. Identify the product area, project, layer, behaviour, and risk level.
2. Locate likely entry points using targeted searches for routes, controllers, services, symbols, components, entities, configuration, tests, or error messages.
3. Read the minimum files needed to establish the current flow.
4. Follow only direct dependencies that materially affect the requested behaviour.
5. Stop when there is sufficient evidence to explain the current behaviour, identify risks, and perform the assigned role.

Do not:

- Traverse every file in a project by default.
- Read the frontend for a backend-only concern unless an API contract or integration must be verified.
- Read the backend for a frontend-only concern unless the consumed contract is unclear or changing.
- Read an entire migration history when the current model and a specifically relevant migration are sufficient.
- Search broadly for unrelated cleanup, refactors, or defects.
- Re-read `README`, workflow, memory, project, or configuration files repeatedly unless they changed or a concrete question requires them.
- Treat repository access as permission for unrestricted repository-wide analysis.

When a task is broad, divide inspection by subsystem or concern instead of loading the entire repository into context.

---

## General Coding Preferences

- Prefer small, focused diffs.
- Prefer boring, maintainable solutions.
- Avoid cleverness where a simple solution is clearer.
- Follow existing naming, folder, architectural, test, and dependency-injection conventions.
- Keep behaviour changes explicit and public contracts stable unless the approved slice changes them.
- Avoid changing generated files or lock files unless required by the slice.
- If tests exist, update only the smallest relevant tests.
- If tests are missing, report the gap rather than creating broad test infrastructure without approval.
- Do not rename public routes, DTOs, files, classes, or symbols unless the slice requires it.
- Do not perform destructive database, production, or infrastructure actions without explicit instruction.

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
- Do not introduce MediatR, CQRS, Clean Architecture, or new architectural frameworks without explicit approval.

---

## React / TypeScript Preferences

For React projects:

- Prefer existing hooks, API wrappers, query patterns, and shared components.
- Avoid duplicating fetch logic inside components.
- Keep components focused on UI state and rendering.
- Avoid broad state-management changes unless requested.
- Preserve route structure unless the task is specifically about routing.
- Prefer small reusable components only when duplication is demonstrated.
- Avoid premature abstraction.
- Do not introduce new UI libraries without explicit approval.
- Do not rewrite styling systems unless explicitly requested.
- Preserve existing loading, error, empty-state, and accessibility behaviour unless the slice changes them.
- Be careful with React Query cache keys and invalidation patterns.
- Avoid unnecessary `useEffect` state syncing when values can be derived directly.

---

## Frontend Page Structure and Large Page Refactors

Use the completed Edit Invoice page as a reference pattern, not a mandatory template:

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

Responsibilities:

- The page entry file should normally be a small route shell handling route parameters, top-level loading/error states, and feature composition.
- A coordinator feature may own page-specific state, queries, mutations, handlers, and composition.
- Cohesive UI sections belong under the page's lowercase `features` folder.
- Page-specific pure domain logic belongs under the page's lowercase `functions` folder.
- Layout constants shared by sibling page features may live beside those features.
- Small private components should remain with their owning feature rather than being split mechanically.
- Domain-neutral helpers belong in `src/lib` only when genuinely reusable or already used by multiple consumers.
- Do not move page-domain logic into `src/lib` merely because reuse is theoretically possible.

Refactoring principles:

- Refactor one page and one bounded slice at a time.
- Preserve behaviour, JSX, accessibility, and styling during structural refactors unless changes are explicitly requested.
- Preserve manual and user-authored styling.
- Do not introduce custom hooks solely to shorten a file.
- Do not introduce context or reducers solely to reduce prop counts or line counts.
- Do not use arbitrary maximum file lengths; cohesion and discoverability determine boundaries.
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

- Treat migrations and production-data changes as high risk.
- Inspect only affected entities, configurations, services, queries, tests, current schema assumptions, and relevant migrations.
- Do not read the entire migration history unless sequencing, historical data shape, rollback, or compatibility requires it.
- Identify whether the requested change needs a migration before implementation.
- Preserve existing data.
- Do not drop columns, tables, indexes, constraints, or data unless explicitly requested.
- Report backward-compatibility, deployment-order, rollback, locking, and data-migration risks.
- For PostgreSQL, be careful with casing, quoted identifiers, timestamp kinds, indexes, and provider-specific functions.
- Do not run destructive database commands without explicit instruction.

---

## Deployment / Infrastructure Preferences

For deployment, production, and infrastructure work:

- Treat production changes as high risk.
- Do not change, print, or overwrite secrets or environment files unless explicitly requested.
- Be careful with CORS, cookies, authentication, HTTPS, reverse proxies, systemd, Caddy, Cloudflare, and database connection strings.
- Prefer minimal changes that can be verified with focused health checks or build commands.
- Clearly report commands that were run and their results.
- If a command could be destructive, stop and request explicit approval before running it.

---

## Node / JavaScript / TypeScript Preferences

For Node, JavaScript, and TypeScript projects:

- Identify the package manager before making changes.
- Prefer existing scripts from `package.json`.
- Do not change dependencies unless explicitly needed.
- Avoid broad dependency upgrades.
- Preserve the existing module style, such as ESM or CommonJS.
- Follow existing linting, formatting, and folder conventions.
- Run the smallest relevant verification command available.
- Do not introduce new frameworks or libraries unless explicitly requested.

---

## Astro Preferences

For Astro projects:

- Preserve existing content structure, routing, and component conventions.
- Prefer Astro components for static content.
- Use client-side JavaScript only when needed.
- Avoid unnecessary hydration.
- Preserve SEO metadata, canonical tags, sitemap setup, robots configuration, and favicon/link tags unless the slice explicitly changes them.
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
- Short reasoning-effort recommendations using the scale in `CODEX_DEV_WORKFLOW.md`.

Avoid:

- Long tutorials.
- Large code dumps.
- Repeating the full request or implementation prompt.
- Abstract architecture essays disconnected from repository evidence.
- Recommendations outside the requested scope.
