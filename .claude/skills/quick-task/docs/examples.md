# Quick-Task — Examples

Reference for good fits and anti-patterns. Load when unsure if a request qualifies.

---

## Good Fits (gate passes)

```
/quick-task Fix the typo "recieve" → "receive" in src/components/Inbox.tsx
→ 1 file, 1 word. Phase 0 pass → edit → re-read → "Fixed typo at Inbox.tsx:42."

/quick-task Rename the local var `tmp` to `pendingCount` in calcTotals()
→ 1 file, ~4 lines, no contract change. Direct edit, lint, done.

/quick-task Bump the retry limit from 3 to 5 in config/http.ts
→ Config tweak, 1 line. Direct edit, done.

"quick fix — the empty-state text should say 'No notes yet'"
→ Signal-activated, 1 file, copy change. Gate passes → direct edit.

/quick-task Add a JSDoc comment to the exported formatDate() helper
→ 1 file, doc-only. Direct edit, done.

"what does the debounce() in src/lib/utils.ts actually do?"
→ Lightweight analysis. Read one file, answer. No artifacts.
```

---

## Anti-Patterns (escalate instead)

```
/quick-task Add user authentication
→ ANTI: risk class = auth, multi-file, architectural. Escalate → story-creator.

/quick-task Refactor the API layer to use the repository pattern
→ ANTI: "refactor (system|architecture)", cross-file, new pattern. Escalate.

/quick-task Add a search feature to the notes list
→ ANTI: feature, UI + backend, multi-layer. Escalate → large/epic tier.

/quick-task Fix the bug where the app crashes sometimes on save
→ ANTI: unknown root cause, investigation chain needed (check 9 fails). Escalate.

/quick-task Just quickly migrate the DB schema, it's a small change
→ ANTI: "migration" risk class, irreversible. Escalate regardless of size.

/quick-task Rename the User type everywhere
→ ANTI: shared type / public contract, unbounded file count. Escalate.
```

**The recurring trap:** a request *described* as small ("just", "quick", "simple") that *is not* small. Trust the size gate, not the adjective.
