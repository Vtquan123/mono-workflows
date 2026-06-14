---
name: task-creator
description: >
  Transform a feature or story description into a set of highly constrained,
  file-scoped implementation tasks for execution agents (Cline). Triggers on
  /task-creator command or when the user asks to "create tasks for STORY-XXX",
  "generate tasks from story", or "break down this feature into tasks". Produces
  deterministic, bounded tasks with explicit file scope, acceptance criteria,
  and dependency chains.
version: 1.2.0
---

# task-creator — Implementation Task Generator

Transforms a story or feature description into deterministic, file-scoped tasks
for execution by Cline or any other constrained agent.

**You are operating in architect-planner mode. You write task files only. You
MUST NOT write or modify production source code outside `.ai/`.**

---

## Stack Reference

Read `.ai/architecture.md § Stack` and `.ai/architecture.md § Stack Rules`.

Every task involving UI files MUST include the applicable rules from
`.ai/architecture.md § Stack Rules` in the task's `## Requirements` section.

---

## Domain Vocabulary

Read `.ai/architecture.md § Domain`. Use entity names exactly as defined there
in task titles and file paths.

---

## Invocation Patterns

```
/task-creator STORY-001-<slug>
/task-creator STORY-001-<slug> --dry-run
/task-creator "Add search to the entity list"
/task-creator STORY-001-<slug> --split TASK-003
/task-creator STORY-001-<slug> --append
```

| Flag | Behavior |
|---|---|
| `STORY-NNN-<slug>` | Read story from `.ai/stories/STORY-NNN-<slug>/story.md` |
| `"<description>"` | Use inline description; auto-create story directory |
| `--dry-run` | Print tasks to conversation; do NOT write files |
| `--split <TASK-ID>` | Re-split an oversized task into sub-tasks |
| `--append` | Add tasks to existing story without overwriting |

---

## Workflow

### Phase 0 — Locate Input

1. **Read `.ai/architecture.md`** — load stack constraints, domain vocabulary,
   path conventions, and open architecture decisions. Stop if missing.
2. If given `STORY-NNN-<slug>`, read `.ai/stories/STORY-NNN-<slug>/story.md`.
   - If missing, stop and tell the user to create it first (or use story-creator).
3. If given an inline description:
   - Find the highest `STORY-NNN` under `.ai/stories/` and increment.
   - Generate a kebab-case slug (3–5 words, lowercase, drop articles/prepositions).
   - Create `.ai/stories/STORY-NNN-<slug>/story.md` with the description.
   - Create `.ai/stories/STORY-NNN-<slug>/context.md` (from `../story-creator/templates/context.md`).
4. Read `.ai/stories/STORY-NNN-<slug>/context.md` — constrains the task set
   (do not re-create already-completed work).

### Phase 1 — Domain Decomposition

Identify every distinct concern in the story. Each concern becomes exactly one
task (or is split if it fails the size gate in Phase 3).

Concern taxonomy and decomposition signals: see `docs/heuristics.md`.

Assign IDs `TASK-001`, `TASK-002`, … in topological order. Lower ID =
earlier wave.

### Phase 2 — Dependency Graph (DAG)

Build an explicit **DAG** over proposed tasks. The graph is the authoritative
source for `depends_on`, `parallel_group`, `blocked_by`, `critical_path`, and
`resource_conflicts` metadata emitted in each task file.

Full schema, edge types, group naming, and conflict detection: see `docs/dependency-graph.md`.

**DAG structure:**

```
Data model
  └─ Migration
  └─ Types
       └─ Repository
            └─ Service
                 └─ API route / Server action
                      └─ Hook
                           └─ UI component
                                └─ Page
                                     └─ Tests
```

**Default ordering rules:**

- Data model tasks have NO dependencies (run first, lowest IDs).
- Type tasks depend on data model tasks.
- Repository tasks depend on type tasks.
- Service tasks depend on repository tasks.
- API/action tasks depend on service tasks.
- Hook tasks depend on type tasks and may depend on API tasks.
- Component tasks depend on type tasks and hooks.
- Page tasks depend on component tasks.
- Test tasks have the highest IDs and depend on the module under test.

**Edge labeling** — `hard` edges (artifact import) create `depends_on`; `soft` edges are advisory only. See `docs/dependency-graph.md § Relationship Types`.

**Parallel groups** — use canonical names from `docs/dependency-graph.md § Group Naming`. Tasks in the same group with no hard edge and no resource conflict are parallel-safe.

**Critical path** — flag tasks on the longest hard-dependency chain with `critical_path: true`.

### Phase 2.5 — Conflict Detection

Before writing any task file, scan all proposed tasks for resource conflicts. A conflict exists if ANY hold:

1. Two tasks share an Allowed Files path (`[modify]`/`[modify]` or overlapping `[create]`).
2. Two tasks produce DB migrations on the same schema or table.
3. Two tasks mutate the same store/reducer, DI registration, env loader, or global config.
4. Two tasks touch tightly coupled domain modules (auth + session, router + route registry, theme provider + tokens).
5. Two tasks modify the same `package.json`, lockfile, Dockerfile, or CI workflow.

For each conflict, apply the lightest fix: split files, merge tasks, or serialize.
See `docs/dependency-graph.md § Conflict Detection Rules` for resolution patterns.

Never let a detected conflict survive into the emitted task files.

### Phase 3 — Task Sizing Check

For each proposed task, run ALL four gates. Fail any gate → split the task.

```
FILE COUNT  : ≤ 5 files
DOMAIN COUNT: = 1 concern type
LOC ESTIMATE: ≤ 200 lines changed
UPSTREAM DEPS: ≤ 3 tasks
```

See `docs/sizing-guide.md` for splitting patterns.

### Phase 4 — Task File Generation

Write each task as `.ai/stories/STORY-NNN-<slug>/tasks/TASK-NNN-<slug>.md`.

The task slug uses the same rules as story slugs: 3–5 words, lowercase, hyphen-separated, drop articles and prepositions.

Every field in the canonical template (`docs/task-format.md`) is MANDATORY.
`N/A` is allowed only for genuinely inapplicable sections.

File path inference: read `.ai/architecture.md § Path Conventions` and use those patterns in `## Allowed Files`.

File creation and deletion rules: see `docs/task-format.md § File Creation Rules` and `§ File Deletion Rules`.

### Phase 5 — Context Initialization

If `.ai/stories/STORY-NNN-<slug>/context.md` is empty or missing, initialize it
from `../story-creator/templates/context.md` — that file is the canonical source
of truth for context.md structure and append schema.

### Phase 6 — Summary

Print to the conversation. Output has two blocks: the task list and the
parallel execution plan derived from the DAG.

```
STORY-NNN-<slug>: <title>
──────────────────────────────────────────────────────
TASK-001-<entity>-data-model      [no deps]        group: data-foundation   <Entity> Data Model
TASK-002-<entity>-types           [deps: 001]      group: data-foundation   <Entity> Types
TASK-003-<entity>-repository      [deps: 002]      group: data-access       <Entity> Repository
TASK-004-post-<entity>-route      [deps: 003]      group: api-surface       POST /api/<entity> Route
TASK-005-<entity>-card-component  [deps: 002]      group: frontend-ui       <Entity>Card Component
TASK-006-<entity>-page            [deps: 004,005]  group: frontend-ui       /<entity> Page
TASK-007-<entity>-repo-tests      [deps: 003]      group: tests-unit        <Entity> Repository Unit Tests
──────────────────────────────────────────────────────
7 tasks written to .ai/stories/STORY-NNN-<slug>/tasks/

Parallel Execution Plan
──────────────────────────────────────────────────────
Wave 1 (no deps):       TASK-001
Wave 2 (after Wave 1):  TASK-002
Wave 3 (after Wave 2):  TASK-003 ‖ TASK-005
Wave 4 (after Wave 3):  TASK-004 ‖ TASK-007
Wave 5 (after Wave 4):  TASK-006

Critical path:          TASK-001 → TASK-002 → TASK-003 → TASK-004 → TASK-006
Resource conflicts:     none
Bottlenecks:            TASK-002 (3 downstream consumers)
```

`‖` denotes tasks that may run in parallel within the same wave.

---

## Anti-Pattern Prevention

Full catalogue: [docs/anti-patterns.md](docs/anti-patterns.md) — single source of truth.

Critical subset (highest severity — full list in docs/anti-patterns.md):
- **ANTI-001** — Cross-domain task: files from ≥2 concern types in one task → split by concern
- **ANTI-002** — God task: >5 allowed files → split by method group or component
- **ANTI-005** — Implicit dependency: upstream import not listed in Dependencies → audit every import
- **ANTI-008** — Missing context update: Context Update section empty → every task needs verbatim append block
- **ANTI-DAG-001** — Missing dependency metadata: task file lacks `## Dependency Metadata` block → emit per docs/task-format.md

---

## Pre-Flight Checklist

Load `docs/checklist.md` and verify all items before writing any task file.

---

## Token Budget

Shared rules: [.ai/token-budget.md](../../../.ai/token-budget.md).

- Default chat summary: ≤ 250 words.
- Target task file length: ≤ 120 lines per task.
- Target task requirement section: ≤ 250 words per task.
- Keep each task focused on one concern.
- Do not paste the full DAG explanation into chat; output the task index and parallel waves concisely.
- Load `docs/dependency-graph.md` only when generating or validating task dependencies.
- Load anti-pattern / checklist docs only during final validation.

---

## Reference Files

- [docs/task-format.md](docs/task-format.md) — canonical template + field rules + file creation/deletion rules
- [docs/sizing-guide.md](docs/sizing-guide.md) — four gates + splitting patterns
- [docs/anti-patterns.md](docs/anti-patterns.md) — anti-pattern catalogue
- [docs/dependency-graph.md](docs/dependency-graph.md) — DAG schema, edge types, parallel groups, conflict detection
- [docs/heuristics.md](docs/heuristics.md) — concern taxonomy, decomposition signals, naming conventions
- [docs/checklist.md](docs/checklist.md) — mandatory pre-flight checklist
- [examples/](examples/) — add your own project-specific task examples here
