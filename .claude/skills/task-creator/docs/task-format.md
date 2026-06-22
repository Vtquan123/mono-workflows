# Task File Format — Field Reference

Every task file written by `task-creator` MUST use this exact structure.
All sections are mandatory. Use `N/A` only for genuinely inapplicable fields.

---

## File Path

```
.ai/stories/STORY-NNN-<slug>/tasks/TASK-NNN-<slug>.md
```

`NNN` is zero-padded to 3 digits. IDs are assigned in dependency order:
leaf nodes (no deps) get the lowest IDs.
`<slug>` is a 3–5 word kebab-case summary derived from the task title
(e.g. `<entity>-data-model`, `post-<entity>-route`). Drop articles and prepositions.

---

## YAML Frontmatter

Every task file MUST begin with a YAML frontmatter block, before the markdown
title. It is the machine-readable summary of the task: a top-level header that
orchestrators, dashboards, dependency-graph tooling, and future CI/linter checks
read without parsing free-form markdown. It complements — never replaces — the
human-readable sections below.

```yaml
---
id: TASK-NNN                 # required — stable task ID, e.g. TASK-001
story: STORY-NNN             # required — parent story ID, e.g. STORY-001
tier: large                  # required — trivial | medium | large | epic
status: ready                # required — draft | ready | blocked | in_progress | completed | review_failed
validation_mode: scoped      # required — none | review | scoped | full
risk: medium                 # required — low | medium | high
allowed_files_count: 3       # required — MUST equal the count under ## Allowed Files
depends_on:                  # required — task IDs only; [] if none
  - TASK-001
  - TASK-002
parallel_group: api-surface  # optional — canonical group name, or null
critical_path: true          # required — boolean; true if downstream tasks depend on this task
---
```

The frontmatter MUST be valid YAML. The `---` fences are mandatory and the
opening fence must be the first line of the file (no leading blank line).

---

## Canonical Template

```markdown
---
id: TASK-NNN
story: STORY-NNN
tier: <trivial|medium|large|epic>
status: <draft|ready|blocked|in_progress|completed|review_failed>
validation_mode: <none|review|scoped|full>
risk: <low|medium|high>
allowed_files_count: <N>
depends_on: []
parallel_group: <canonical-group|null>
critical_path: <true|false>
---

# TASK-NNN-<slug> — <Layer> <Entity> <Action>

## Objective

One paragraph, 2–5 sentences. State exactly what the execution agent must
produce: file names, exported symbols, function signatures, observable
side-effects. Must be specific enough that a different agent produces the
same output. No optionality. No ambiguity.

## Allowed Files

The execution agent MUST only create or modify these files.
Any write outside this list is a constraint violation.

- `path/to/file.ts` — [create|modify] — brief reason
- `path/to/other.ts` — [create|modify] — brief reason

## Forbidden

Files and directories the execution agent MUST NOT touch.

- `<data-layer-dir>/` — data layer is out of scope for this task
- `<schema-file>` — schema is owned by TASK-NNN-<slug>

Always end with:
- All files not listed in Allowed Files

## Requirements

Numbered list. Each item is a single, actionable, testable requirement.
Use MUST / MUST NOT (RFC 2119). Name exact exports, interface fields,
function signatures. Never defer decisions to the executor.

1. ...
2. ...
3. ...

## Validation Mode

One of: `none` | `review` | `scoped` | `full`.
See `.ai/architecture.md § Validation Convention`.

<none|review|scoped|full>

## Acceptance Criteria

Markdown checklist. Each criterion must be verifiable by running a command
or inspecting a specific, named output, OR (for `review`/`none` modes) by a
named review check. No subjective criteria.

- [ ] `<validation command appropriate to the mode>`
- [ ] `<specific verifiable check>`
- [ ] `<specific verifiable check>`

## Dependencies

Upstream tasks that MUST be completed before this task starts.
List each with the artifact it produces.

- TASK-NNN-<slug> — produces `<artifact>` consumed by this task

If none: `None`

## Dependency Metadata

Structured DAG metadata consumed by orchestrators. All fields MANDATORY.
See `docs/dependency-graph.md` for full schema and rules.

\`\`\`yaml
task_id: TASK-NNN-<slug>
depends_on:           # hard deps only (artifact-consuming upstreams)
  - TASK-NNN-<slug>
soft_deps: []         # sequencing preferences only; not blocking
blocked_by:           # union of depends_on + resource_conflicts keys
  - TASK-NNN-<slug>
parallelizable: true  # false if any hard dep in same group or any resource_conflict
parallel_group: <canonical-group>   # data-foundation | data-access | backend-logic |
                                    # api-surface | frontend-hooks | frontend-ui |
                                    # tests-unit | tests-integration | docs | infra
resource_conflicts:   # tasks touching overlapping files / shared state
  - TASK-NNN-<slug>: <reason e.g. "same file: src/lib/auth.ts">
critical_path: false  # true if task lies on the longest hard-dep chain
relationship_notes: |
  (optional, 1–2 lines on non-obvious coupling)
\`\`\`

## Validation Commands

Commands runnable from the project root, scoped to the task's `validation_mode`.
For `review`/`none`, replace this section with `## Validation Notes` listing the
review checks performed (or why validation is not required).

1. `<command appropriate to validation_mode>` — <what it verifies>
2. `<command>` — <what it verifies>

## Context Update

After completing this task, the execution agent MUST append the
following block to `.ai/stories/STORY-NNN-<slug>/context.md` under
"Completed Tasks":

\`\`\`
## TASK-NNN-<slug> — <name> (completed <YYYY-MM-DD>)

### Decisions
- <any deviations from the plan and the reason>

### Files Changed
- `path/to/file.ts` — created
- `path/to/other.ts` — modified

### Notes
- <anything the next task executor should know>
\`\`\`
```

---

## Field Rules

### YAML Frontmatter fields

| Field | Required | Allowed values | Meaning |
|---|---|---|---|
| `id` | yes | `TASK-NNN` | Stable task ID. Matches the `# TASK-NNN-<slug>` title. |
| `story` | yes | `STORY-NNN` | Parent story ID. |
| `tier` | yes | `trivial` \| `medium` \| `large` \| `epic` | Planning tier inherited from the story or task classification (`.ai/planning-tiers.md`). |
| `status` | yes | `draft` \| `ready` \| `blocked` \| `in_progress` \| `completed` \| `review_failed` | Lifecycle state. Newly generated tasks are `ready`, or `blocked` if not safely executable. |
| `validation_mode` | yes | `none` \| `review` \| `scoped` \| `full` | MUST equal the `## Validation Mode` section. |
| `risk` | yes | `low` \| `medium` \| `high` | Based on scope, dependencies, architecture impact, test coverage, and blast radius. |
| `allowed_files_count` | yes | integer ≥ 0 | MUST equal the number of files listed under `## Allowed Files`. |
| `depends_on` | yes | list of `TASK-NNN` IDs (`[]` if none) | Task IDs only. MUST match the upstream IDs in `## Dependencies` and the `depends_on` of `## Dependency Metadata`. |
| `parallel_group` | no | canonical group name (`docs/dependency-graph.md § Group Naming`) or `null` | Group for parallel execution; `null` if not applicable. MUST match `## Dependency Metadata`. |
| `critical_path` | yes | `true` \| `false` | `true` if downstream tasks depend on this task or it blocks the main story path. MUST match `## Dependency Metadata`. |

The frontmatter is a top-level summary. The richer DAG fields (`soft_deps`,
`blocked_by`, `parallelizable`, `resource_conflicts`, `relationship_notes`) stay
in `## Dependency Metadata`, which remains authoritative for those.

#### Frontmatter consistency rules (MUST hold)

1. `allowed_files_count` equals the number of entries under `## Allowed Files`.
2. `depends_on` equals the upstream task IDs in `## Dependencies` (and `## Dependency Metadata`'s `depends_on`).
3. `validation_mode` equals the `## Validation Mode` section.
4. `risk` is consistent with `validation_mode` and scope — `full` validation or
   high blast radius implies `risk: medium`/`high`; a localized `review`/`none`
   task is normally `risk: low`.
5. `status: ready` MUST NOT be used if required dependencies are incomplete or
   architecture context is missing — use `status: blocked` instead.
6. `critical_path: true` whenever another task lists this task in its `depends_on`.
7. `parallel_group` and `critical_path` match `## Dependency Metadata`.
8. The frontmatter parses as valid YAML.

#### Blocked tasks

If a task cannot safely be executed by Cline, set `status: blocked` in the
frontmatter and add a `## Blocker` section (immediately after `## Objective`)
explaining the cause — one or more of: missing context, unclear acceptance
criteria, missing allowed files, dependency conflict, architecture ambiguity,
validation uncertainty. Do not emit `status: ready` for such a task.

### `# TASK-NNN-<slug> — <title>`

- `NNN` is zero-padded (001, 002, … 099, 100).
- `<slug>`: 3–5 words, lowercase, hyphen-separated, derived from the title. Drop articles/prepositions.
- Title: `<Layer> <EntityName> <Action>` in title case.
- Layer vocabulary: `Data Model`, `Migration`, `Types`, `Repository`, `Service`,
  `Validator`, `API Route`, `Server Action`, `Hook`, `Component`, `Page`,
  `Tests`, `Config`.
- Keep under 60 characters.

### `## Objective`

- 2–5 sentences. Single paragraph.
- MUST name: exact output file(s), exported symbol names, key method signatures.
- MUST NOT use: "refactor", "improve", "handle", "manage", "deal with",
  "etc.", "as needed", "properly", "correctly".
- MUST NOT defer architectural decisions to the executor.

### `## Allowed Files`

- One entry per file, with exact path from project root.
- Mark each `[create]`, `[modify]`, or `[delete]`.
  - `[create]` — file does not yet exist; executor must create it. If the file
    exists at run time, executor treats it as `[modify]` and preserves exports.
  - `[modify]` — file already exists; executor edits only the relevant section.
  - `[delete]` — file must be removed. **Only present after explicit user confirmation during task planning.** Executor MUST ask the user again before deleting.
- If a new file lives in a directory that does not yet exist, add a note in
  the Objective: "Executor MUST create parent directory `<dir>/` before writing
  the file." Do not list directories as separate entries.
- If a path cannot be determined until a dependency runs, write:
  `[TBD: path determined by TASK-NNN-<slug> output]`
  and add TASK-NNN-<slug> to the Dependencies section.
- NEVER write: "any file in X", "relevant files", "src/**", "as needed".

### `## Forbidden`

- At minimum, list directories that would tempt an executor to drift.
- Always include: `All files not listed in Allowed Files`.
- For UI tasks, explicitly forbid data layer directories and API route directories.
- For data model tasks, explicitly forbid UI component and page directories.

### `## Requirements`

- 3–10 items. One actionable requirement per item.
- Use MUST / MUST NOT for hard constraints.
- Name exact: class names, function names, parameter types, return types,
  interface field names, error types thrown.
- For UI components: MUST specify the class merging utility and design token
  names per `.ai/architecture.md § Stack Rules` — never raw colors or values.
- Do NOT say "use appropriate error handling" — say "wrap in try/catch;
  throw `new AppError('DB_ERROR', message)` on failure".

### `## Validation Mode`

- Exactly one of: `none` | `review` | `scoped` | `full`.
- Definitions: `none` = metadata-only/non-functional, no command validation;
  `review` = manual/review checks only (docs, copy, prompt, rule, markdown);
  `scoped` = targeted validation (lint, typecheck, unit/package test);
  `full` = full build or project-level validation.
- The task creator selects the mode from files affected, task type, risk level,
  dependency position, and whether the task is an integration/final task or touches
  shared contracts or build/runtime config. Default mapping:

  | Task Type | Default validation_mode |
  |---|---|
  | docs / markdown / prompt / rules only | review |
  | formatting / comments only | review |
  | type-only localized code change | scoped |
  | localized feature code | scoped |
  | tests-only change | scoped |
  | shared interface / API contract | full |
  | package config / build config | full |
  | migration / deployment / runtime config | full |
  | final integration task | full |

- High-risk changes (shared interfaces/contracts, build/package/dependency config,
  migrations, auth flow, routing/API boundaries, deploy/runtime config, cross-package
  integration) MUST use `full`. Final integration tasks MUST use `full`.

### `## Acceptance Criteria`

- Markdown checklist (`- [ ] …`).
- Minimum 2 criteria; aim for 3–5.
- MUST include validation appropriate to the task's `validation_mode`
  (see `## Validation Mode` below and `.ai/architecture.md § Validation Convention`):
  - `full` → include the full build / project-level command (from `.ai/architecture.md § Commands`).
  - `scoped` → include targeted commands (typecheck, lint, unit/package test) or explain why none available.
  - `review` → include review checks instead of build commands.
  - `none` → explain why validation is not required.
- For API tasks: include a `curl` command with expected status code.
- For repository/service tasks: include a named export check or unit test.
- No subjective criteria ("looks correct", "works", "is implemented").

### `## Dependencies`

- Every upstream TASK-ID whose output this task imports or modifies.
- Include what artifact the upstream task produces.
- If none: write `None` — do not omit the section.
- Implicit imports (types, repositories) MUST be listed here.

### `## Dependency Metadata`

- MANDATORY block; verbatim YAML keys; preserve key order.
- `depends_on` is `hard`-edge only. Implicit imports → add the producing task.
- `soft_deps` carries `soft` and `sequencing_preference` edges. Empty list if none.
- `blocked_by` = `depends_on` ∪ keys of `resource_conflicts`. Scheduler reads this.
- `parallelizable` is `false` if ANY of: `depends_on` non-empty AND target shares group; `resource_conflicts` non-empty.
- `parallel_group` MUST be one of the canonical names in `docs/dependency-graph.md § Group Naming`. Invent only if no canonical fits.
- `resource_conflicts` lists task IDs that touch overlapping files / shared global state, each with a one-phrase reason.
- `critical_path` `true` iff the task lies on the longest hard-dependency chain through the story. At least one task per story MUST be flagged.
- `relationship_notes` optional. Use only when coupling is non-obvious.

### `## Validation Commands`

- Commands scoped to the task's `validation_mode`.
- Runnable from project root without modification. No placeholder commands.
- `full` → include the build/project-level command from `.ai/architecture.md § Commands`.
- `scoped` → targeted commands (typecheck, lint, unit/package test).
- `review`/`none` → replace with `## Validation Notes`: list review checks performed,
  or state why validation is not required.

### `## Context Update`

- Contains the verbatim block the execution agent copies into `context.md`.
- MUST reference the correct `STORY-NNN-<slug>` in the append path.
- Context Update heading MUST use the full `TASK-NNN-<slug>` identifier.
- MUST include `### Decisions`, `### Files Changed`, `### Notes` subsections.
- This section must NEVER be empty.

---

## File Creation Rules

### When to mark `[create]`

| Scenario | Files to mark |
|---|---|
| New data model added to schema | schema file — `[modify]`; migration file auto-generated, note it in Objective |
| New route segment | page file — `[create]`; add layout only if segment needs its own layout |
| New server action module | actions file — `[create]` |
| New repository module | repository file — `[create]` |
| New UI component | component file — `[create]` |
| New shared types file | types file — `[create]` |
| New custom hook | hook file — `[create]` |

**Directory scaffolding rule**: If a `[create]` file lives in a directory that does not yet exist, include a note in the task Objective: "The executor MUST create the parent directory `<dir>/` before writing the file." Do NOT list directories as separate Allowed Files entries — only list files.

**`[create]` means**: the file does not exist yet; the executor must create it from scratch. If the file already exists when the executor runs, the executor MUST treat it as `[modify]` and preserve existing exports.

---

## File Deletion Rules

If story analysis implies a file should be deleted (e.g., replacing a module, removing a deprecated route), **do NOT silently add it to Allowed Files**. Stop and ask the user:

```
The story implies deleting `<exact/path/to/file.ts>`.
Reason: <one sentence why the deletion is needed>.
Should I include this deletion in the task? (yes / no / rename instead)
```

- **yes**: write the task with `[delete]` marker in Allowed Files and include a "Delete Steps" subsection in Requirements listing the exact shell command and any import cleanup.
- **no**: document the file as `[modify]` or exclude it.
- **rename**: use `[create]` for the new path and `[delete]` for the old path — two separate entries.

**Never infer that a deletion is safe.** Always surface it.

---

## Context Initialization Template

If `.ai/stories/STORY-NNN-<slug>/context.md` is empty or missing, initialize it
from `../story-creator/templates/context.md`. That template is the canonical
source of truth for context.md structure and append schema.
