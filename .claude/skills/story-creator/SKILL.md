---
name: story-creator
description: >
  Transform a product requirement or feature request into bounded implementation
  stories, then orchestrate task generation via task-creator. Triggers on
  /story-creator command or when the user asks to "create a story for X",
  "break this feature into stories", or "plan implementation for Y". Produces
  architecture-safe, independently executable stories with full task chains.
version: 1.0.0
---

# story-creator — Implementation Story Generator

Transforms a product requirement into bounded, architecture-safe implementation
stories, then invokes `task-creator` to produce executable task chains for Cline.

**You are operating in architect-planner mode. You write story files and task
files only. You MUST NOT write or modify production source code outside `.ai/`.**

---

## Roles & Boundaries

This skill operates within the strict two-role model (Architect-Planner writes
`.ai/**`; Cline executes within `Allowed Files`). Canonical definition:
[`.ai/architecture.md` § Roles & Boundaries](../../../.ai/architecture.md).

---

## Stack Reference

Read `.ai/architecture.md § Stack` and `.ai/architecture.md § Stack Rules`.

Translate those rules into per-story constraints. Do not leave them implicit.
Every story's `## Constraints` section MUST copy the Stack Rules block verbatim
from `.ai/architecture.md`.

---

## Domain Vocabulary

Read `.ai/architecture.md § Domain`. Use entity names exactly as defined there
in story titles, constraint blocks, and affected areas.

---

## Invocation Patterns

```
/story-creator "Add search to the notes list"
/story-creator --prd path/to/requirements.md
/story-creator --prd path/to/requirements.md --dry-run
/story-creator --tier=medium "Add archived filter"          # force a planning tier
/story-creator STORY-001-<slug> --tasks                     # generate tasks for an existing story
/story-creator STORY-001-<slug> --retask                    # regenerate tasks (preserves story.md)
```

| Flag | Behavior |
|---|---|
| `"<description>"` | Inline requirement; auto-assign STORY-NNN-<slug> |
| `--prd <path>` | Read requirement from a markdown file |
| `--tier=<name>` | Force planning tier (trivial / medium / large / epic). Bypasses tier-classifier. |
| `--dry-run` | Print story plan to conversation; do NOT write files |
| `STORY-NNN-<slug> --tasks` | Skip story creation; invoke task-creator only |
| `STORY-NNN-<slug> --retask` | Re-run task-creator; existing tasks are overwritten |

---

## Workflow

### Phase 0 — Locate and Parse Input

0. **Confirm intent (Mode 3).** An explicit `/story-creator` invocation already
   confirms Mode 3 — proceed. If you arrived here by auto-trigger on a vague phrase,
   run the `intent-verification` gate first and continue only if it routes to Mode 3.
1. **Read `.ai/architecture.md`** — load stack constraints, domain vocabulary,
   path conventions, and open architecture decisions. Stop if missing.
2. Identify input source: inline description, PRD file, or existing STORY-XXX.
3. If `--prd <path>`: read the file; extract feature name, goals, user personas,
   constraints, out-of-scope items. Stop if file is missing.
4. If inline description: treat as a one-sentence feature statement.
5. Scan `.ai/stories/` for highest `STORY-NNN` and assign the next ID. Generate a
   kebab-case slug from the story title (3–5 words, lowercase, drop articles and
   prepositions). Full directory: `STORY-NNN-<slug>`.
6. Honor any open architecture decisions in `.ai/architecture.md § Architecture Decisions`.

### Phase 0.5 — Planning Tier Classification

1. Read `.ai/planning-tiers.md` (single source of truth for tier behavior).
2. If the user passed `--tier=<name>`, use that tier and skip to the tier-branch.
3. Otherwise invoke the `tier-classifier` skill on the input. It returns a tier + rationale block.
4. Record `Tier: <name>` and `Tier Rationale: <one line>` at the top of every artifact.

**Tier branch** (full behaviors in `docs/planning-tiers.md`):

| Tier      | What this run produces                                                    | Phases below that run                                            |
|-----------|---------------------------------------------------------------------------|------------------------------------------------------------------|
| `trivial` | One quick-task file in `.ai/quick-tasks/QUICK-NNN-<slug>.md`              | Skip Phases 1–6. Use the Quick Task path in `docs/planning-tiers.md`. |
| `medium`  | One slim story (4 required sections) with ≤ 3 tasks                       | Run Phase 1 lite, skip Phase 2 split, run a slim Phase 4, Phase 5, Phase 6. |
| `large`   | Default. One full story (9 sections) with ≤ 7 tasks                       | Run Phases 1–6 as written below.                                 |
| `epic`    | Epic dir `.ai/epics/EPIC-NNN-<slug>/` with phased roadmap + Phase-1 stories | Run Phase 1, then the Epic path in `docs/planning-tiers.md`, then Phase 5/6 per generated story. |

Do **not** duplicate signal weights, thresholds, or behaviors here —
they live in `.ai/planning-tiers.md` and `docs/planning-tiers.md`.

### Phase 1 — Feature Analysis

Decompose the input into structured dimensions:

```
DOMAIN ANALYSIS
  ├── Primary domain (primary entity from .ai/architecture.md § Domain)
  ├── Secondary domains touched
  ├── Layer span (data → service → API → UI)
  └── Cross-cutting concerns (auth, validation, error handling)

DELIVERY DIMENSIONS
  ├── User-visible outcomes (what changes for the user)
  ├── Data changes (new models, modified fields)
  ├── API surface changes (new routes, modified contracts)
  └── UI changes (new components, modified pages)
```

Output: a plain-language feature summary (3–6 bullets). This becomes the
Business Context in the story.

### Phase 2 — Story Boundary Definition

Apply the Story Boundary Rules to carve the feature into stories:

**One story = one coherent business/domain concern.**

Boundary rules:

- A story MUST NOT span more than one primary entity's data layer.
- A story MUST NOT mix API implementation with UI implementation.
- A story CAN include types, repository, service, and API for the same entity.
- A story CAN include a hook and its corresponding UI component if they are
  tightly coupled and the total task count stays ≤ 7.
- Auth, validation, and error utilities MUST each be their own story if they
  introduce new shared modules.
- Split by layer when a story would produce > 7 tasks after decomposition.

See `docs/heuristics.md` for decomposition signals, naming conventions, and example story lists.

### Phase 3 — Story Sizing Check

For each story, run all five gates. Fail any gate → split the story.

```
TASK COUNT      : ≤ 7 tasks after decomposition
DOMAIN COUNT    : = 1 primary entity
LAYER SPAN      : ≤ full stack (data → UI) for one entity
EXECUTOR RISK   : no decision requires real-time architecture judgment
DEPENDENCY DEPTH: ≤ 2 upstream stories
```

See `docs/sizing-guide.md` for split patterns and merge conditions.

### Phase 4 — Story File Generation

For each story, write `.ai/stories/STORY-NNN-<slug>/story.md` using the canonical
template (`docs/story-format.md`). All nine sections are MANDATORY.

Initialize `.ai/stories/STORY-NNN-<slug>/context.md` from `templates/context.md`.
Create `.ai/stories/STORY-NNN-<slug>/tasks/` (empty — task-creator fills it).
Create `.ai/stories/STORY-NNN-<slug>/logs/` (empty — executor fills it).

### Phase 5 — Task Orchestration via task-creator

For each story, invoke `task-creator`:

1. Call `/task-creator STORY-NNN-<slug>`.
2. Verify: sequential task IDs, no constraint violations, acyclic dependency chain.
3. Record task index in `context.md` under `## Task Index`.

**Do NOT generate tasks manually.** Always delegate to task-creator.

See `docs/orchestration-protocol.md` for the full integration spec.

### Phase 6 — Summary

Print to the conversation:

```
STORY PLAN
══════════════════════════════════════════════════════════════
STORY-001-<entity>-data-layer  <Entity> Data Layer Foundation
  Tasks: TASK-001 … TASK-005  (5 tasks)
  Domains: <Entity>
  Entry point: TASK-001 (no deps)

STORY-002-<entity>-list-view  <Entity> List View
  Tasks: TASK-006 … TASK-010  (5 tasks)
  Domains: <Entity>
  Entry point: TASK-006 (deps: STORY-001 complete)
══════════════════════════════════════════════════════════════
2 stories written to .ai/stories/
Next: run /task-creator STORY-001-<entity>-data-layer to generate implementation tasks.
```

---

## Anti-Pattern Prevention

Full catalogue: [docs/anti-patterns.md](docs/anti-patterns.md) — single source of truth.

Critical subset (highest severity — full list in docs/anti-patterns.md):
- **SANTI-001** — Giant story: >7 tasks → split by layer (data story + UI story)
- **SANTI-002** — Vague goal: uses "improve/refactor/enhance" → rewrite as user-observable outcome
- **SANTI-005** — Architecture drift: deferred choices in Technical Decisions → planner decides all patterns
- **SANTI-008** — Missing acceptance criteria: no story-level criteria → ≥3 mechanically verifiable criteria
- **SANTI-009** — Premature UI: UI tasks before data layer story complete → add data layer story as dependency

---

## Pre-Flight Checklist

Load `docs/checklist.md` and verify all items before writing any story file.

---

## Token Budget

Shared rules: [.ai/token-budget.md](../../../.ai/token-budget.md).

- Default chat summary: ≤ 250 words.
- Story artifact should be concise and structured.
- Target slim story: ≤ 80 lines.
- Target full story: ≤ 180 lines unless explicitly complex.
- Do not paste long examples into chat.
- Use templates from `templates/` instead of embedding full templates in `SKILL.md` or chat.
- Load `docs/**` only when needed for the current artifact step.
- Put details in story files, not in conversational output.

---

## Reference Files

- [docs/planning-tiers.md](docs/planning-tiers.md) — tier branches: Quick Task, Slim Story, Full Story, Epic
- [docs/story-format.md](docs/story-format.md) — canonical 9-section story template
- [docs/sizing-guide.md](docs/sizing-guide.md) — story sizing gates and split patterns
- [docs/anti-patterns.md](docs/anti-patterns.md) — 10 story anti-patterns + banned phrases
- [docs/heuristics.md](docs/heuristics.md) — decomposition signals, naming conventions, example story lists
- [docs/checklist.md](docs/checklist.md) — mandatory pre-flight checklist
- [docs/orchestration-protocol.md](docs/orchestration-protocol.md) — task-creator integration spec
- [.ai/planning-tiers.md](../../../.ai/planning-tiers.md) — central tier configuration (signals, thresholds, behaviors)
- [templates/story.md](templates/story.md) — blank story template
- [templates/context.md](templates/context.md) — blank context template
- [examples/](examples/) — add your own project-specific story examples here
