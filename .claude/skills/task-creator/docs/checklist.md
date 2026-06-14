# Task Creator — Pre-Flight Checklist

Run ALL items before writing any task file. One failure = fix it first.

- [ ] Task touches a single concern type
- [ ] Task has ≤ 5 allowed files
- [ ] Objective names exact exports and file paths (no ambiguity)
- [ ] All file paths are exact (or `[TBD: dep TASK-NNN]` with explicit dep)
- [ ] All implicit upstream dependencies are listed in Dependencies
- [ ] ≥ 2 acceptance criteria that are mechanically verifiable
- [ ] Context Update section contains the verbatim append block
- [ ] Tasks are ordered so dependencies come first (lower ID = runs sooner)
- [ ] No banned phrases (see `docs/anti-patterns.md` § Red-Flag Phrase Blocklist)
- [ ] UI tasks include stack-approved token and class merging constraints
- [ ] Task file includes `## Dependency Metadata` block with all fields
- [ ] `depends_on` lists only `hard` edges (artifact-consuming upstreams)
- [ ] `parallel_group` uses a canonical group name (see `docs/dependency-graph.md`)
- [ ] `resource_conflicts` populated if any same-file / shared-state collision
- [ ] `parallelizable: false` whenever `resource_conflicts` is non-empty
- [ ] At least one task in the story flagged `critical_path: true`
- [ ] Phase 2.5 conflict detection ran with zero unresolved conflicts
