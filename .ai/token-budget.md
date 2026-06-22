# Token Budget — Shared Policy

Canonical token-economy rules for every skill under `.claude/skills/**`. Each
`SKILL.md` carries its own tailored `## Token Budget`; this file holds the
shared principles so they are not duplicated. Skills reference this file.

## Principles

1. Default to concise output.
2. Do not paste long examples unless explicitly requested.
3. Do not paste full templates into chat — reference the `templates/` file.
4. Load supporting docs (`docs/**`) only when the step needs them.
5. Prefer references to files over repeating long content.
6. Keep routing/classification outputs extremely short.
7. Keep generated artifacts bounded and scannable.
8. Use verbose output **only** when:
   - the user explicitly asks for detailed output;
   - the skill is generating a required artifact;
   - the task is blocked and needs explanation;
   - there is a safety or correctness issue that must be explained.
9. Do not repeat global policy from `.ai/*.md` — reference it instead.
10. Keep final summaries concise.

## Context Loading Matrix

Load the minimum context for each situation. Stop reading files once the next
action is safe and unambiguous. Prefer references to other files over copying
their content into context.

| Situation | Load | Do not load | Stop when |
|---|---|---|---|
| Direct answer / simple Q&A | `README.md`, `CLAUDE.md` only if needed | Skills, examples, story/task templates | The question can be answered directly |
| Intent routing | `.ai/intent-verification.md`, minimal `CLAUDE.md` routing notes | Story/task templates, full skill docs | Workflow mode is clear |
| Tier classification | `.ai/planning-tiers.md`, relevant `.ai/architecture.md` summary | Story examples, unrelated docs | Tier and rationale are clear |
| Story creation | `.ai/architecture.md`, `.ai/planning-tiers.md`, story-creator skill | Task-creator docs until task phase | Story boundaries and acceptance criteria are clear |
| Task creation | Current story, current context, task-creator skill, relevant architecture references | Sibling stories, unrelated tasks, full repo scan | Tasks have allowed files, dependencies, validation mode |
| Cline execution | Current task, current story context, allowed files only, `.clinerules/workflows/executor.md` | Whole repo, sibling tasks, unrelated docs | Task can be implemented without guessing |
| Task review | Task, diff, acceptance criteria, validation result | Full repo, unrelated stories/tasks | Compliance and quality risks are known |
| Architecture review | `.ai/architecture.md`, relevant files only | All skills, all agents, all workflows | Architecture decision or gap is identified |
| Agent routing | `docs/agents.md`, current request summary | Full skill docs unless selected | Required/optional agents are known |

### Context Loading Rules

- Start with the smallest sufficient context.
- Load source-of-truth files before generated artifacts.
- Load generated artifacts before implementation files.
- Do not load examples unless the format is unclear.
- Do not load sibling tasks unless dependency or conflict analysis requires them.
- Do not load the whole repository for scoped task execution.
- Prefer file references and section references over copied content.
- Stop loading context once the next action is safe and unambiguous.
- If additional context would only confirm what is already clear, do not load it.

### Cline Context Rules

- Cline should receive only the current task, current story context, allowed files, and executor workflow.
- Cline must not inspect unrelated files unless the task explicitly allows it.
- Cline must not use broader repository context to redesign the task.
- Cline must stop after completing one task and reporting the result.

### Agent Context Rules

- Agents should receive only the files needed for their checkpoint role.
- Do not give every agent every skill document.
- Do not chain agents just to repeat the same analysis.
- Prefer the smallest sufficient agent context.

## Reference Files

- [architecture.md](architecture.md) — roles, boundaries, stack
- [intent-verification.md](intent-verification.md) — routing policy
- [planning-tiers.md](planning-tiers.md) — tier thresholds
