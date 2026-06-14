# Project Architecture

> Authoritative source of truth for this project's stack, domain vocabulary,
> path conventions, and architecture decisions.
>
> **Skills read this file in Phase 0.** Do NOT duplicate these values inside
> skill files or task/story files — always reference here so updates propagate
> automatically.

---

## Roles & Boundaries

Single source of truth for the two-agent role separation. CLAUDE.md, skill
files, and `.clinerules/` reference this section — do not restate the rules
elsewhere.

| Role | Tool | Responsibilities | Writes to |
|---|---|---|---|
| **Architect-Planner** | Claude Code | Stories, tasks, architecture decisions, task orchestration | `.ai/**` only |
| **Executor** | Cline | Task implementation, context updates, bounded file writes | Only files in the active task's `Allowed Files` |

**Invariants (non-negotiable):**

- Claude Code NEVER writes production source code — it defines what Cline writes.
- Claude Code writes ONLY under `.ai/**` (stories, tasks, quick-tasks, epics, context).
- Cline writes ONLY to files listed in the active task's `Allowed Files`.
- Cline NEVER makes architecture decisions — it appends them to `context.md`.
- Cline executes one task at a time and stops after each, awaiting a human trigger.
- Context/log files (`context.md`, `execution-log.md`, `quick-log.md`) are append-only.

**Direct Execution Exception:** the only path for Claude Code to write
production code directly is the explicitly invoked `/quick-task` skill (its own
size gate escalates back to `/story-creator` if the change is too large). Every
other path keeps Claude in architect-planner mode — plan only. Claude MUST NOT
infer a request is "small enough" to implement without `/quick-task`.

Cline execution constraints live in [`../.clinerules/`](../.clinerules/).

---

## Stack

<!-- TODO: Fill in your project's stack versions and constraints. -->

| Layer | Version / Constraint |
|---|---|
| <!-- Framework --> | <!-- e.g. Next.js 16.2.6 — App Router, RSC. No pages/ directory. --> |
| <!-- Runtime/Lang --> | <!-- e.g. TypeScript strict mode. Path alias @/* → src/*. --> |
| <!-- Styling --> | <!-- e.g. Tailwind CSS v4 — config in src/app/globals.css. --> |
| <!-- UI Layer --> | <!-- e.g. ShadCN base-nova. Primitives: @base-ui/react (NOT Radix UI). --> |
| <!-- ORM/DB --> | <!-- e.g. Prisma 5.x — PostgreSQL. --> |
| <!-- Utilities --> | <!-- e.g. src/lib/utils.ts exports cn(). --> |
| <!-- Icons --> | <!-- e.g. lucide-react v1.x --> |

## Stack Rules

<!-- TODO: Write the stack rules that executors MUST follow in every task.
     Copy this block verbatim into every story's ## Constraints section. -->

Executor MUST follow these in every task. Copy this block verbatim into
every story's `## Constraints` section.

- <!-- Rule 1: e.g. Next.js 16.2.6 — App Router only, no pages/ directory -->
- <!-- Rule 2: e.g. Tailwind CSS v4 — token names only, no raw colors or hex values -->
- <!-- Rule 3: e.g. TypeScript strict mode — no any, no non-null assertion without comment -->
- <!-- Rule 4: e.g. cn() from @/lib/utils — required for all conditional class merging -->
- <!-- Rule 5: Add any additional stack constraints -->

---

## Domain

### Entities

<!-- TODO: List your project's primary domain entities.
     Use these names exactly in story titles, task titles, file paths, and constraint blocks. -->

Use these names exactly in story titles, task titles, file paths, and constraint blocks.

| Entity | Description |
|---|---|
| `<!-- Entity1 -->` | <!-- e.g. Note — primary content unit with title, body, tags --> |
| `<!-- Entity2 -->` | <!-- e.g. User — auth identity --> |
| `<!-- Entity3 -->` | <!-- add more as needed --> |

### Feature → Entity Mapping

<!-- TODO: Map common feature phrases to their primary entity.
     Helps story-creator assign the correct domain to a requirement. -->

| Feature phrase | Primary entity |
|---|---|
| `<!-- "create / view / edit a <thing>" -->` | <!-- Entity1 --> |
| `<!-- "log in / authenticate" -->` | <!-- User --> |
| `<!-- add more rows --> ` | |

---

## Path Conventions

<!-- TODO: Fill in your project's file path patterns.
     Skills use these to populate Allowed Files in tasks. -->

All paths are relative to the project root.

| Concern | Path Pattern |
|---|---|
| <!-- Types (entity) --> | <!-- e.g. src/types/<entity>.ts --> |
| <!-- Repository --> | <!-- e.g. src/lib/<entity>.repository.ts --> |
| <!-- Service --> | <!-- e.g. src/lib/<entity>.service.ts --> |
| <!-- Validator --> | <!-- e.g. src/lib/<entity>.validator.ts --> |
| <!-- API route (collection) --> | <!-- e.g. src/app/api/<entity>/route.ts --> |
| <!-- API route (by ID) --> | <!-- e.g. src/app/api/<entity>/[id]/route.ts --> |
| <!-- Page --> | <!-- e.g. src/app/<route>/page.tsx --> |
| <!-- UI component --> | <!-- e.g. src/components/<EntityName>.tsx --> |
| <!-- Custom hook --> | <!-- e.g. src/hooks/use<Entity>.ts --> |
| <!-- Utility --> | <!-- e.g. src/lib/<name>.ts --> |
| <!-- Unit test --> | <!-- e.g. src/lib/__tests__/<name>.test.ts --> |

---

## Commands

```bash
# TODO: Replace with your project's actual commands
# npm run dev      # start dev server
# npm run build    # production build — verifies compilation
# npm run lint     # linter
```

## Validation Convention

Validation is risk-based and task-appropriate. Tasks are NOT required to
run a full build. Each task declares a `validation_mode` and includes only
the validation appropriate to it.

Modes:

- `none`: no command validation required. Metadata-only or non-functional changes.
- `review`: manual/review validation only. Docs, copy, prompt, rule, or markdown-only changes.
- `scoped`: targeted validation for the affected area — lint, typecheck, unit test, or package-level test.
- `full`: full build or project-level validation required.

Policy by tier and risk:

- **Trivial / documentation-only**: no build required unless changed files affect
  executable code. May use review-only validation.
- **Small / localized code**: prefer scoped validation (typecheck, lint, targeted
  tests, affected-package). Full build optional unless change affects integration behavior.
- **Medium / micro-story**: scoped validation per task. Full build (or equivalent
  integration validation) only on final integration tasks or when the task changes shared contracts.
- **Large / full-story**: task-appropriate validation per task. Final integration
  task includes full build or equivalent project-level validation.
- **Epic / multi-phase**: full build (or equivalent) at phase boundaries and final
  integration. Do not force every leaf task to run full build.
- **High-risk tasks**: full build or stronger validation is required when changing
  shared interfaces/contracts, build configuration, package/dependency configuration,
  migrations, auth/authorization flow, routing or API boundaries, deployment/runtime
  configuration, or cross-package integration.

Acceptance criteria must include validation appropriate to the task's `validation_mode`:
`full` → full build/project-level command; `scoped` → targeted commands (or explain why
none available); `review` → review checks instead of build commands; `none` → explain
why validation is not required. Full validation for final integration and high-risk
tasks is never optional.

---

## Architecture Decisions

Append decisions here as they are made. Execution agents record decisions
after completing relevant tasks.

```
Format for each decision:
### <Decision Title> (<YYYY-MM-DD>)
- Status: Decided | Undecided | Superseded by <decision>
- Decision: <what was decided>
- Reason: <why>
- Affects: <list of stories or modules>
```

<!-- TODO: Add your first architecture decisions here. Example:

### Auth Provider (<YYYY-MM-DD>)
- Status: Undecided
- Options: Clerk, next-auth
- Constraint: Must be locked before any story touching the User entity
- Affects: User auth setup story, all protected API routes

-->
