# Story Creator — Heuristics

Reference for decomposition signals, naming conventions, and example story lists. Load during Phase 2 when splitting features into stories.

---

## Feature → Story Decomposition Signals

| Feature phrase | Story split |
|---|---|
| "user can create/view/edit/delete a `<entity>`" | `<Entity>` Data Layer + `<Entity>` UI (2 stories) |
| "user can upload/record `<media>`" | `<Media>` Data Layer + `<Media>` UI (2 stories) |
| "process/transform data automatically" | Processing Data Layer + Processing Service (2 stories) |
| "user can tag/label `<entity>`" | Tag Data Layer + `<Entity>`-Tag Association (1–2 stories) |
| "user can search `<entity>`" | Search API + Search UI (2 stories) |
| "user logs in / authenticates" | Auth Setup (1 story, always isolated) |
| "validate input" | Validator utility (part of domain story, not standalone unless shared) |
| "dashboard / home page" | Dashboard UI (1 story, depends on entity layers) |

---

## Story Naming Convention

```
STORY-NNN-<slug>  <Entity> <Concern> [Layer]

Directory name: STORY-NNN-<slug>  (used for .ai/stories/ folder and all cross-references)
Human title:    <Entity> <Concern> [Layer]  (used inside story.md heading after the em dash)

Slug rules:
  - Derived from the human title
  - Lowercase, hyphen-separated
  - 3–5 words; drop articles (a, an, the) and prepositions (for, of, in)

Examples:
  STORY-001-<entity>-data-layer        <Entity> Data Layer Foundation
  STORY-002-<entity>-list-view         <Entity> List View
  STORY-003-<entity>-detail-view       <Entity> Detail View
  STORY-004-<entity>-search-api        <Entity> Search API
  STORY-005-<entity>-search-ui         <Entity> Search UI
  STORY-006-<feature>-service          <Feature> Service
  STORY-007-<entity2>-data-layer       <Entity2> Data Layer
  STORY-008-user-auth-setup            User Auth Setup
```

---

## Example Story List (Phase 2 output)

After decomposing a feature, produce a story list in this format:

```
STORY-001-<entity>-data-layer     <Entity> Data Layer Foundation     (schema + types + repository + service)
STORY-002-<entity>-api            <Entity> CRUD API                  (GET + POST + PUT + DELETE routes)
STORY-003-<entity>-list-view      <Entity> List View                 (hook + list component + page)
STORY-004-<feature>-ui            <Feature> UI                       (feature hook + feature component)
```
