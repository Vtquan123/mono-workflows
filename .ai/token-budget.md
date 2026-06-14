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

## Reference Files

- [architecture.md](architecture.md) — roles, boundaries, stack
- [intent-verification.md](intent-verification.md) — routing policy
- [planning-tiers.md](planning-tiers.md) — tier thresholds
