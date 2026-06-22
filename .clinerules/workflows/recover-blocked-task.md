# Recover Blocked Task

**For**: Cline execution agent
**Triggered by**: A task Cline cannot complete safely
**Role boundary**: Diagnose and report. Do not guess or expand scope to unblock.

When execution cannot proceed safely, Cline **stops** — it does not improvise an
architecture change, add files, or escalate the validation mode to get unstuck.

## Steps

1. **Stop implementation.** Leave the working tree in a consistent state.
2. **Summarize the blocker** in one clear sentence.
3. **Classify the blocker** — pick the closest:
   - missing context
   - missing allowed files
   - unclear acceptance criteria
   - dependency conflict
   - validation failure
   - architecture ambiguity
4. **Suggest the smallest safe next action** (e.g. "add `<file>` to Allowed
   Files", "clarify criterion X", "complete TASK-NNN-1 first"). Recommend; do not
   apply it.
5. **Do not guess or auto-expand scope.** Escalate to human/Claude per
   [`../01-role-boundaries.md`](../01-role-boundaries.md).

## Report

```
TASK BLOCKED: TASK-NNN — <Title>

Phase: <where it stopped>
Blocker: <one sentence>
Type: missing context | missing allowed files | unclear acceptance criteria |
      dependency conflict | validation failure | architecture ambiguity
Smallest safe next action: <recommendation>

Context updated: execution-log.md appended with BLOCKED status
```
