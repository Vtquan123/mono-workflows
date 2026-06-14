# Review Report Template

Use this exact structure for every reviewer output. No preamble. No closing pep-talk.

---

```markdown
# Review Summary

**Scope:** <branch | PR# | STORY-NNN | files…>
**Files reviewed:** <n>
**Diff size:** ~<lines> changed

<one-paragraph high-signal summary: what was changed, what the headline risk is>

## Critical Issues
- `path/to/file.ext:LINE` — <problem>. <why it matters>. <recommended direction>.

## Major Issues
- `path/to/file.ext:LINE` — <problem>. <why it matters>. <recommended direction>.

## Minor Issues
- `path/to/file.ext:LINE` — <problem>. <recommended direction>.

## Suggestions
- `path/to/file.ext:LINE` — <suggestion>.

## Acceptance Criteria Check
- [x] AC-1: <criterion> — Passed.
- [ ] AC-2: <criterion> — Failed. <one-line reason>.
- [ ] AC-3: <criterion> — Not verifiable from diff. <what would verify it>.

## Final Status
APPROVED
<or>
NEEDS_CHANGES — <one-line summary of blockers>
```

---

## Template Rules

- Every finding cites `path:line` (use the post-change line number).
- Each bullet is one line. Put longer reasoning in a trailing parenthetical, still on one line.
- Recommendations describe **direction**, not code (e.g. "guard for empty array before indexing").
- Omit any severity section that has zero items (drop the heading too).
- If no significant issues exist, replace all severity sections with:
  `No significant issues found in reviewed changes.`
  but still emit Acceptance Criteria Check and Final Status.

## Final Status Decision

- `NEEDS_CHANGES` if there is **any** Critical, **any** Major, or **any** failed acceptance criterion.
- `APPROVED` otherwise (Minor + Suggestions alone do not block).

See [`examples/example-review.md`](../examples/example-review.md) for a canonical filled example.
See [`examples/example-approved.md`](../examples/example-approved.md) for an approved-with-no-issues example.
