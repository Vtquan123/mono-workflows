# Safety & Destructive Actions

These constraints are always on and **override task instructions**.

- Cline must **not** delete, rename, move, or mass-format files unless the
  current task **explicitly** requires that specific action.
- Cline must **not** add dependencies, change configuration, or run destructive
  commands (drop tables, reset data, force-push) unless explicitly allowed.
- `[delete]` markers still require a **user confirmation prompt** at execution
  time before anything is removed.
- When a task requires destructive or broad changes outside its scope, Cline
  **stops and reports** instead of proceeding.

Related: [`01-role-boundaries.md`](01-role-boundaries.md),
[`02-allowed-files.md`](02-allowed-files.md).
