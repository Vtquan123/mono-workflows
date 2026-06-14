# Task Creator — Heuristics

Reference for concern taxonomy, decomposition signals, and naming conventions. Load during Phase 1 and Phase 4.

---

## Concern Taxonomy

| Concern | Examples | Task? |
|---|---|---|
| Data model | DB schema field/model | 1 task per model |
| DB migration | schema migration command | 1 task (after schema task) |
| Repository | DB access layer | 1 task per entity |
| Service | Business logic | 1 task per service module |
| Validator | Input validation util | 1 task per validator module |
| Types | Shared interfaces/enums | 1 task (may cover 1–3 files) |
| Server action | Server-side action handler | 1 task per action group |
| API route | REST handler | 1 task per HTTP method |
| Hook | Custom framework hook | 1 task per hook |
| UI component | UI component | 1 task per component |
| UI page | Page/layout | 1 task per page |
| Config | Env vars, config files | 1 task |
| Test suite | Unit/integration tests | 1 task per tested module |

---

## Decomposition Signals

| Story phrase | Generated concerns |
|---|---|
| "user can create an `<entity>`" | data model + types + service + POST route + form component + page |
| "user can view a list" | types + GET route + hook + list component + page |
| "user can edit" | PUT route + service.update method + edit form component |
| "user can delete" | DELETE route + service.delete method (often 1 task) |
| "store/persist/save X" | data model + migration |
| "search/filter" | GET route with query params + filter component (separate) |
| "authenticate" | auth middleware task + token util (always separate tasks) |
| "validate input" | validator util task (1 file, independent) |

---

## Naming Conventions

```
TASK-NNN-<slug> — <Layer> <EntityName> <Action>

File name: TASK-NNN-<slug>.md  (used for the file on disk and all cross-references)
Heading:   TASK-NNN-<slug> — <Layer> <EntityName> <Action>

Slug rules:
  - Derived from the human title
  - Lowercase, hyphen-separated
  - 3–5 words; drop articles (a, an, the) and prepositions (for, of, in)

Examples:
  TASK-001-<entity>-data-model       — <Entity> Data Model
  TASK-002-<entity>-types            — <Entity> Types
  TASK-003-<entity>-repository       — <Entity> Repository
  TASK-004-post-<entity>-route       — POST /api/<entity> Route
  TASK-005-<entity>-card-component   — <Entity>Card Component
  TASK-006-use-<entity>-hook         — use<Entity> Hook
  TASK-007-<entity>-page             — /<entity> Page
  TASK-008-<entity>-repo-tests       — <Entity> Repository Unit Tests
```
