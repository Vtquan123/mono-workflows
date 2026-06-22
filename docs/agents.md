# Agent Policy

Single source of truth for **when agents are mandatory, recommended, optional, or
forbidden** in the mono-workflows two-agent model (Claude plans, Cline executes).
Do not duplicate this matrix or these levels elsewhere — link here.

## Purpose

Agents are specialized **checkpoints** used to shape, verify, or review work.
They are reasoning roles, not executors. They do **not** replace skills,
`.ai/` source-of-truth documents, or Cline executor workflows.

Hierarchy (most authoritative first):

- **`.ai/`** — source of truth. Architecture, intent routing, planning tiers.
- **skills** — procedural workflows for Claude (`story-creator`, `task-creator`,
  `quick-task`, `reviewer`, etc.).
- **agents** — specialized reasoning / checkpoint roles. They read and check; they
  do not own procedure and do not write production code.
- **`.clinerules/`** — execution constraints for Cline.
- **Cline** — scoped executor, not architect. Implements only the approved task
  within its `Allowed Files`.

Agents live in [`.claude/agents/`](../.claude/agents/). Skills live in
[`.claude/skills/`](../.claude/skills/). Cline rules live in
[`.clinerules/`](../.clinerules/). Canonical policy lives in
[`.ai/`](../.ai/).

## Agent Usage Levels

### MUST use

- **`workflow-orchestrator`**
  - When the user request is ambiguous.
  - When the correct workflow mode is unclear.
  - When it is unclear whether the request should become a direct answer, story
    creation, task creation, review, or Cline handoff.

- **`architecture-analyst`**
  - When the request affects architecture, stack choices, path conventions,
    dependencies, contracts, module boundaries, or cross-layer design.
  - When [`.ai/architecture.md`](../.ai/architecture.md) is missing, incomplete,
    or still contains placeholder sections.
  - Before generating stories/tasks that rely on unclear project architecture.

- **`task-reviewer`**
  - Before handing **any** generated task to Cline.
  - Must verify allowed files, scope size, dependencies, acceptance criteria,
    validation mode, and whether the task is executable by Cline without
    architectural guessing.

- **`execution-guardian`**
  - After Cline reports a task as complete.
  - Must check whether Cline respected allowed files, completed acceptance
    criteria, updated required context/log files, and followed the declared
    validation mode.

### SHOULD use

- **`story-designer`** — for large features, multi-step product changes, unclear
  acceptance criteria, or requests that need decomposition into stories.
- **`task-planner`** — when a story needs a dependency graph, task DAG,
  sequencing, parallelization, or careful task slicing.
- **`test-validator`** — when validation mode is unclear, tests are missing, task
  risk is medium/high, or the implementation touches fragile behavior.

### MAY use

- For extra confidence on complex requests.
- When the user explicitly asks for deeper review.
- When the cost of a mistake is higher than the cost of extra reasoning.

### MUST NOT use

Agents must **not** be used when:

- The user asks a simple explanation question.
- The user asks for a small rewrite, translation, or direct answer.
- A skill already provides the complete required procedure.
- The task is low-risk and does not require specialized review.
- Multiple agents would only repeat the same analysis.
- The agent would need to invent project facts not present in `.ai/` or the
  current story/task context.

## Agent Routing Matrix

| Situation | Required agent | Optional agent | Notes |
|---|---|---|---|
| User request is ambiguous | workflow-orchestrator | none | Decide workflow mode first |
| Architecture unclear or impacted | architecture-analyst | workflow-orchestrator | Do not generate stories/tasks from guesses |
| Creating a story | story-designer | architecture-analyst | Use architecture analyst if architecture is unclear |
| Creating tasks from a story | task-planner | test-validator | Use task-reviewer before Cline handoff |
| Before Cline execution | task-reviewer | test-validator | Mandatory checkpoint |
| After Cline completion | execution-guardian | reviewer / test-validator | Mandatory compliance checkpoint |
| Code quality review | reviewer | test-validator | Reviewer checks code quality, guardian checks workflow compliance |
| Simple Q&A | none | none | Answer directly |

## Agent Chain Limits

Reduce token waste:

- Default to zero or one agent.
- Use more than one agent only when each has a **distinct** responsibility.
- Do not chain agents with overlapping roles.
- Do not run `reviewer` and `execution-guardian` for the same purpose.
- Prefer the smallest sufficient agent set.
- Never use agents as a substitute for reading the relevant source-of-truth files.

## Reviewer vs Execution Guardian

- **`execution-guardian`** checks **workflow compliance**: allowed files, task
  scope, acceptance criteria, validation mode, context/log updates, whether the
  next task can start.
- **`reviewer`** checks **code quality**: correctness, edge cases, bugs,
  maintainability, security, performance, typing/API issues.

Rules:

- Always use `execution-guardian` after Cline task completion.
- Use `reviewer` only when code quality review is needed, task risk is
  medium/high, or the user explicitly asks for it.

## Recommended Agent Flow

```text
Simple question
  -> Direct answer (no agent)

Small, well-scoped implementation request
  -> workflow-orchestrator        (if mode unclear)
  -> architecture-analyst         (if architecture impact possible)
  -> quick-task or task-creator skill
  -> task-reviewer                (MUST, before Cline)
  -> Cline executor
  -> execution-guardian           (MUST, after Cline)
  -> reviewer / test-validator    (if risk medium/high or requested)

Feature or multi-step request
  -> workflow-orchestrator
  -> architecture-analyst         (if architecture impact possible)
  -> story-designer -> story-creator skill
  -> task-planner -> task-creator skill
  -> task-reviewer                (MUST, before Cline)
  -> Cline executor
  -> execution-guardian           (MUST, after Cline)
  -> reviewer / test-validator    (if risk medium/high or requested)
```

## Agents vs Skills vs Cline

- **Agents** must not replace skills.
- **Agents** must not write production code. Only `/quick-task` lets Claude write
  code directly — see
  [`.ai/architecture.md` § Roles & Boundaries](../.ai/architecture.md).
- **Cline** must not make architecture decisions or expand task scope beyond its
  `Allowed Files`. See
  [`.clinerules/workflows/executor.md`](../.clinerules/workflows/executor.md).
