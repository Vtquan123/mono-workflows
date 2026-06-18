# Agents

How Claude Code decides between a direct answer, an agent, a skill, and Cline in
the mono-workflows two-agent model (Claude plans, Cline executes).

Agents live in [`.claude/agents/`](../.claude/agents/). Skills live in
[`.claude/skills/`](../.claude/skills/). Cline rules live in
[`.clinerules/`](../.clinerules/). This file is the single source of truth for
agent routing — do not duplicate the full matrix elsewhere.

## Agent Routing Matrix

| Situation | Use | Purpose | Next Step |
|---|---|---|---|
| User asks a simple explanation or asks about the repo | No agent | Answer directly without workflow overhead | Respond directly |
| User request needs workflow mode selection | `workflow-orchestrator` | Classify as direct answer, quick task, planning-only, or full workflow | Route to the right next step |
| Request may affect architecture, stack, path conventions, dependencies, commands, or cross-cutting design | `architecture-analyst` | Detect architecture impact and missing context | Update architecture context or proceed |
| Feature request needs story boundaries | `story-designer` | Define story scope, business goal, user outcome, and planning tier | Run `story-creator` skill |
| Story is approved and needs task breakdown | `task-planner` | Propose safe, file-scoped, dependency-aware task sequence | Run `task-creator` skill |
| Story artifact needs to be created | `story-creator` skill | Create story files using canonical repo format | Proceed to task planning |
| Task artifact needs to be created | `task-creator` skill | Create task files using canonical repo format | Review task before execution |
| Task file is ready but not yet executed | `task-reviewer` | Check scope, allowed files, dependencies, acceptance criteria, and Cline readiness | Hand to Cline only if PASS |
| Task is ready for implementation | Cline executor | Implement only the approved task within Allowed Files | Produce execution report |
| Validation strategy is unclear or task risk needs validation review | `test-validator` | Recommend minimal meaningful validation or review validation results | Proceed to execution review |
| Cline has completed a task | `execution-guardian` | Verify acceptance criteria, scope compliance, validation status, and context/log updates | Continue, fix, or stop |

## Recommended Agent Flow

```text
Simple question
  -> Direct answer

Small, well-scoped implementation request
  -> workflow-orchestrator
  -> architecture-analyst, if architecture impact is possible
  -> quick task or task-creator skill
  -> task-reviewer
  -> Cline executor
  -> test-validator, if validation is needed
  -> execution-guardian

Feature or multi-step request
  -> workflow-orchestrator
  -> architecture-analyst, if architecture impact is possible
  -> story-designer
  -> story-creator skill
  -> task-planner
  -> task-creator skill
  -> task-reviewer
  -> Cline executor
  -> test-validator, if validation is needed
  -> execution-guardian
```

## Agents vs Skills vs Cline

- **Agents** — focused specialists for routing, analysis, planning, review, and
  validation. They shape and check work; they read, they do not own procedure.
- **Skills** — the canonical procedural workflow logic and artifact format
  (`story-creator`, `task-creator`, `quick-task`, etc.). Agents do not replace them.
- **Cline** — the constrained executor. Implements only the approved task within
  its `Allowed Files`. See [`.clinerules/workflows/executor.md`](../.clinerules/workflows/executor.md).

Rules:

- Agents must not replace skills.
- Agents must not write production code unless explicitly intended and safe
  (only `/quick-task` lets Claude write code directly — see
  [`.ai/architecture.md` § Roles & Boundaries](../.ai/architecture.md)).
- Cline must not make architecture decisions or expand task scope.

## Agent Usage Principles

1. Use the fewest agents necessary.
2. Do not use agents for simple direct answers.
3. Always use `workflow-orchestrator` when the correct workflow mode is unclear.
4. Use `architecture-analyst` before planning when architecture impact is possible.
5. Use `story-designer` before `story-creator` for feature-level requests.
6. Use `task-planner` before `task-creator` for multi-task stories.
7. Always use `task-reviewer` before handing a generated task to Cline.
8. Use `test-validator` only when validation strategy or validation results need focused review.
9. Always use `execution-guardian` after Cline completes a task.
10. Do not let Cline modify files outside `Allowed Files`.
11. Do not let Cline make architecture decisions.
12. Do not duplicate long rules across agents, skills, and docs.
