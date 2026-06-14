# Tier Classifier — Output Examples

Reference output blocks for each tier. Load when verifying output format or checking score calibration.

---

## Trivial

```
Request: "Fix typo in the homepage hero title."
TIER: trivial
SCORE: -1
SIGNALS:
  estimated_files: 1 (+0)
  scope_keywords: typo (-2)
  layer_span: 1 (+0)
  modules_involved: 1 (+0)
  migration_or_infra: no (+0)
  ui_plus_backend: no (+0)
  test_impact: none (+0)
  architectural_decision_required: no (+0)
TIE_BREAKERS_APPLIED: ["1-file rename/typo → force trivial"]
RATIONALE: Single-file copy edit, no layers, no logic.
```

---

## Medium

```
Request: "Add an `archived` boolean filter to the notes list query."
TIER: medium
SCORE: 4
SIGNALS:
  estimated_files: 2-3 (+1)
  scope_keywords: add field (+1)
  layer_span: 2 (+2)
  modules_involved: 1 (+0)
  migration_or_infra: no (+0)
  ui_plus_backend: no (+0)
  test_impact: scoped (+0)
  architectural_decision_required: no (+0)
TIE_BREAKERS_APPLIED: none
RATIONALE: Two-layer change, one entity, ~3 files.
```

---

## Large

```
Request: "Add a notes search feature (API + UI)."
TIER: large
SCORE: 9
SIGNALS:
  estimated_files: 4-7 (+3)
  scope_keywords: feature, endpoint, component (+3)
  layer_span: 3 (+4)
  modules_involved: 1 (+0)
  migration_or_infra: no (+0)
  ui_plus_backend: yes (+3)
  test_impact: scoped (+0)
  architectural_decision_required: no (+0)
TIE_BREAKERS_APPLIED: none
RATIONALE: Full-stack feature on a single entity.
```

---

## Epic

```
Request: "Migrate users from MySQL to Postgres, update auth service and admin UI."
TIER: epic
SCORE: 21
SIGNALS:
  estimated_files: 16+ (+10)
  scope_keywords: migration, multi-service (+6)
  layer_span: 4 (+6)
  modules_involved: 3+ (+5)
  migration_or_infra: yes (+4)
  ui_plus_backend: yes (+3)
  test_impact: broad (+0)
  architectural_decision_required: yes (+5)
TIE_BREAKERS_APPLIED: ["migration_or_infra + modules>=2 → force epic"]
RATIONALE: Cross-cutting migration spanning multiple modules.
```
