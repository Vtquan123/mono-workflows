# Review Workflow — Diff-Aware Playbook

This document expands the five-phase workflow defined in `SKILL.md` with
concrete commands, decision points, and stop conditions. Read-only throughout.

---

## Phase 1 — Scope discovery

Goal: know exactly what to review before reading a single line of code.

Commands (read-only):

```bash
git status                          # uncommitted state
git diff --stat main...HEAD         # branch changes vs main
git log --oneline main..HEAD        # commits on this branch
git diff --stat <base>...<head>     # user-specified range
```

Decision points:

| Signal                                  | Action                                          |
|-----------------------------------------|-------------------------------------------------|
| Branch tracks a known story (`STORY-NNN`) | Load the story doc read-only.                  |
| Commit references `TASK-NNN` / `QUICK-NNN` | Load that task/quick-task doc.                |
| Diff > 20 files OR > 1500 changed lines | Pause. Ask user: full review or scoped subset. |
| No diff vs `main`                       | Fall back to staged + unstaged working tree.   |
| Reviewing a PR by number                | `gh pr view N --json files,title,body`.        |

Stop condition: you have a finite, named list of files in scope and a known
acceptance criteria source (or explicit acknowledgement there is none).

---

## Phase 2 — Diff-aware reading

Read only what is necessary to judge the change.

Priority order:

1. The changed hunks themselves.
2. The 10–30 lines surrounding each hunk (existing context).
3. The function signature(s) the hunk depends on, if not visible in the hunk.
4. Call sites of changed exported symbols (only when regression risk is in
   question).
5. The matching test file for the changed module.

Avoid:

- Reading entire files when the hunk is self-contained.
- Following imports transitively beyond depth 1.
- Reading vendored / generated / lockfile changes line-by-line.

Parallel reads are encouraged when investigating independent files in the
same phase. Sequential reads when each read informs the next.

---

## Phase 3 — Analysis

Walk the axes from `SKILL.md § Phase 3`. For each finding, record:

- `path:line` (post-change line number)
- One-sentence problem statement
- One-sentence "why it matters" (impact / blast radius)
- One-sentence recommended direction (not code)
- Tentative severity (apply rubric)

If a finding requires reading outside the diff to confirm, do that read now,
then return. Do not let confirmation reads cascade into a full audit.

---

## Phase 4 — Acceptance criteria check

Source of acceptance criteria, in preference order:

1. The linked story/task document's "Acceptance Criteria" section.
2. The PR description's checklist.
3. Inferred from the task title + commit messages (mark as inferred).

For each criterion, one of:

- **Passed** — verified directly from the diff.
- **Failed** — directly contradicted by the diff or by a finding above.
- **Not verifiable from diff** — would need runtime, manual QA, or files
  outside the diff. State what would verify it.

Do not run the code. Do not execute tests. Verification is static, from the
diff.

---

## Phase 5 — Report

Emit the report exactly per `SKILL.md § Output Template`. Then stop.

Stop means:

- No "next steps" section.
- No offer to apply fixes.
- No follow-up tool calls that mutate state.
- Wait for the user's next instruction.

---

## Phase 3 — Analysis Axes

Walk each changed region against these axes. Skip axes that do not apply to the change.

- **Correctness** — does it do what the task says? Off-by-one, wrong operator, swapped args, wrong return type, wrong branch taken.
- **Regression risk** — does it break existing call sites, contracts, serialization, public APIs, migrations, or persisted state?
- **Edge cases** — empty input, null/undefined, zero, negative, very large, unicode, concurrent access, timezone, locale, partial failure.
- **Architecture & conventions** — layering violations, leaking concerns, duplicate abstractions, ignoring an existing helper, naming drift.
- **Typing & null-safety** — `any`, unchecked casts, optional chaining swallowing errors, lost generics, narrowed-then-widened types.
- **Async / concurrency** — missing `await`, unhandled promise rejection, race conditions, lost cancellation, double-fire effects.
- **State management** — stale closure, mutated props/state, missing dependency in effect, derived state stored instead of computed.
- **Performance** — N+1 queries, accidental O(n²), unnecessary re-renders, blocking I/O on hot path, missing memoization where it matters.
- **Security** — injection (SQL/shell/HTML), unsanitized user input, secrets in logs/commits, weak auth/authz checks, broken CSRF/SSRF assumptions.
- **Error handling & observability** — swallowed errors, missing logs, unhelpful messages, wrong error type, no telemetry on critical paths.
- **Testing** — missing tests for the new behavior, tests asserting implementation details instead of behavior, snapshot-only coverage, flaky patterns.
- **Maintainability** — dead code, unclear names, comment lying about code, helper introduced for a single caller, hidden coupling.

---

## Token-Efficient Review Strategy

1. **Lead with `git diff --stat`** to plan, not with `git diff` of the full patch. Decide which files justify a deep read.
2. **Read changed hunks first.** Only widen to surrounding context when the hunk alone is insufficient to judge correctness.
3. **Open referenced symbols on demand.** Do not preemptively read every file a changed function imports.
4. **Prefer `grep`/`Glob`** to confirm a single fact (e.g. "is this helper used elsewhere?") rather than reading multiple files.
5. **Batch independent reads in parallel** (multiple `Read`/`Grep` calls in one tool-use block) when investigating different files.
6. **Stop reading once a finding is established.** Do not keep digging to "be thorough" — note the finding and move on.
7. **Skip noise.** Lockfiles, generated files, snapshots, dist/, vendored code: list them in scope but do not deep-read unless the change is non-mechanical.
8. **Quote the smallest excerpt** that supports a finding. Never paste whole functions when one line proves the point.
9. **One review pass.** Plan in Phase 1, execute in Phase 2, report in Phase 5.

Target: a typical small/medium task review fits in a single response.

---

## Best Practices

- **High signal, low volume.** A short report with five real findings beats a long report padded with style nits.
- **Prioritize the headline risk.** Surface the dominant issue in the summary paragraph before the bullet list.
- **Be specific.** "This is risky" is not a finding. "`foo.ts:42` reads `user.id` before the null-check on line 39" is a finding.
- **Flag uncertainty explicitly.** If you cannot verify something from the diff alone, say so under the relevant severity and explain what would resolve it.
- **Avoid speculation.** Don't report "this *could* be slow" without a concrete reason. Performance findings need a path argument.
- **Respect the author.** Critique the code, not the person.
- **Stay inside the diff.** If something outside the diff is genuinely load-bearing, mention it once under Suggestions — don't pivot into a repo-wide audit.
- **Do not praise.** Neither `NEEDS_CHANGES` nor `APPROVED` reviews need a compliment paragraph.
