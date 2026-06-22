# Allowed Files

Cline's write boundary is the task's **`Allowed Files`** list.

- Cline may **edit only** files listed in the task's `Allowed Files` section
  (per marker: `[create]`, `[modify]`, `[delete]`).
- Cline may **inspect** only the files needed to complete the current task.
- Cline must **not** modify sibling tasks, unrelated stories, unrelated modules,
  or generated artifacts unless the task explicitly allows it.
- If additional files are required to finish the task, Cline **stops and reports
  the required scope change**. It does not add files to the allowed set itself.

Related: [`01-role-boundaries.md`](01-role-boundaries.md),
[`05-safety-destructive-actions.md`](05-safety-destructive-actions.md).
