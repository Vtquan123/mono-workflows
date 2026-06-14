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

Validation convention: every task's Acceptance Criteria MUST include
the build command as the first item.

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
