# Review Completed Task

**For**: Cline execution agent
**Triggered by**: A request to review a task Cline just finished, before the next task starts
**Role boundary**: Review only. Do not plan, redesign, or expand scope.

This is an on-demand compliance check. It verifies the completed task obeyed the
persistent rules in [`../01-role-boundaries.md`](../01-role-boundaries.md) …
[`../05-safety-destructive-actions.md`](../05-safety-destructive-actions.md).

## Checklist

| Check | Pass condition | Rule |
|---|---|---|
| Allowed files only | Every changed file is in the task's `Allowed Files` list | [`02-allowed-files.md`](../02-allowed-files.md) |
| Acceptance criteria met | Each acceptance criterion is satisfied by the diff | — |
| Validation mode followed | The declared `Validation Mode` was honored, not escalated | [`03-validation-modes.md`](../03-validation-modes.md) |
| Context/log updated | If the workflow required it, `context.md` / log was appended | [`04-context-logging.md`](../04-context-logging.md) |
| No destructive overreach | No unrequested delete/rename/move/format/dependency change | [`05-safety-destructive-actions.md`](../05-safety-destructive-actions.md) |
| Blockers surfaced | Any blocker before the next task is recorded and reported | — |

## Report

```
TASK REVIEW: TASK-NNN — <Title>

- Allowed files only: pass / FAIL (<offending path>)
- Acceptance criteria: met / NOT met (<which>)
- Validation mode: followed (<mode>) / VIOLATED (<how>)
- Context/log updated: yes / no / not required
- Destructive overreach: none / FOUND (<what>)
- Blockers before next task: none / <description>

Verdict: READY for next task / NOT READY (<reason>)
```

Stop after the report. Do not start the next task.
