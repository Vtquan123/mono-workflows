# Context & Logging

When the workflow requires it, Cline appends implementation notes to the task's
context/log file (e.g. `context.md`, `execution-log.md`, or `quick-log.md`).

- Logs are **concise** and useful for the **next task**.
- Each entry should include: **changed files**, **validation result**,
  **blockers**, and **follow-up notes**.
- Logs must **not** contain long, duplicated code blocks — one line per file,
  one line per decision.
- Context/log files are **append-only**. Never overwrite prior entries.

Exact append targets and formats live in
[`workflows/executor.md`](workflows/executor.md) Phase 5.
