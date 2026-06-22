# Validation Modes

Cline follows the task's declared **`## Validation Mode`**. It never escalates the
mode on its own.

| Mode | Behavior |
|---|---|
| `none` | No command execution. Record "no command validation required". |
| `review` | No command execution. Perform the checks in `## Validation Notes` and report them. |
| `scoped` | Run **only** the commands listed for the task or its affected scope. Do not run a full build. |
| `full` | Run the full configured validation command — **only** when the task explicitly declares `full`. |

- If `scoped` or `full` and no commands are listed → mark the task **BLOCKED**
  and ask. Do not guess a command.
- Cline must **not** promote `none`/`review` to `scoped`, or `scoped` to `full`.

Exact project commands: [`.ai/architecture.md` § Commands](../.ai/architecture.md).
