# Intent-Verification — Examples

Routing examples and anti-patterns. Load when need routing rationale or deciding how the gate behaves.

---

## Example Conversations

**Ambiguous → clarify, then route light**

```
User: "optimize this workflow"
Gate: signals fire — open-ended verb, no scope, no mode.
Ask:  "Want me to (a) tweak it directly, (b) discuss what 'optimized' should
       mean first, or (c) plan it as a tracked workflow?"
User: "just discuss it"
→ Mode 2. No files created.
```

**Clear command → silent route, no question**

```
User: "/quick-task rename `tmp` to `count` in utils.ts"
Gate: fast path — explicit command. → Mode 1. No question.
```

**Question phrasing → Direct Answer**

```
User: "what does the debounce helper do?"
Gate: fast path — question phrasing. → Direct Answer. No question.
```

**Vague feature → clarify, do NOT auto-orchestrate**

```
User: "add a search feature"
Gate: "feature" + no detail = ambiguous.
Ask:  "Plan this as a full workflow (stories + tasks for Cline), or do you
       want a lightweight version I build directly?"
User: "full workflow"
→ Mode 3 → /story-creator → tier-classifier (likely `large`).
```

---

## Positive Examples (gate working correctly)

```
"fix this"            → ambiguous → ask → route to the answer
"/story-creator …"    → fast path → Mode 3, no question
"explain this regex"  → fast path → Direct Answer, no question
"refactor this file"  → ambiguous (scope unknown) → ask → route
"plan the auth flow"  → fast path ("plan") → Mode 2, no question
```

---

## Anti-Pattern Examples (what the gate prevents)

```
ANTI — orchestrating a vague prompt
  User: "refactor this"
  WRONG: immediately run /story-creator, generate STORY-005 + 6 tasks.
  RIGHT: ask one question — scope and mode are unknown.

ANTI — asking when intent is explicit
  User: "/story-creator add CSV export"
  WRONG: "Do you want a workflow or a quick fix?" — they already said.
  RIGHT: route straight to Mode 3.

ANTI — defaulting upward
  User: "make this better"  (no answer to the clarification)
  WRONG: assume Mode 3, build stories.
  RIGHT: default to the lightest plausible outcome (Direct Answer / Mode 1).

ANTI — multi-round interrogation
  WRONG: ask 4 questions about requirements before routing.
  RIGHT: one question, route, gather detail inside the chosen mode.

ANTI — assuming Cline
  User: "add this feature"
  WRONG: assume full workflow + Cline delegation.
  RIGHT: ask — the user may want a lightweight direct build.
```
