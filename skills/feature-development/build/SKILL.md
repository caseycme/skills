---
name: build
description: >-
  Implements feature work using the refinement brief plus design-system BUILD,
  architecture BUILD, and convention-first guidance. Phase 4 of
  feature-development. Use during implementation after planning, or when the user
  asks to build a refined feature.
---

# Build

Implement the feature using the **same standards that shaped the plan**.

This skill is phase 4 of [`feature-development`](../SKILL.md). It does not
replace child skills — it applies them while coding.

## Inputs

Required from prior phases (or equivalent in chat):

- **Refinement brief** — what, AC, out of scope, account types
- **Design system PLAN** — reuse targets, PL sequencing, leave-local calls
- **Architecture PLAN** — applicable themes and precedents

## Workflow

1. **Read** [`convention-first`](../../engineering/build/convention-first/SKILL.md)
   and follow it for all code changes.
2. **Design system — BUILD mode:** read [`design-system`](../design-system/SKILL.md)
   BUILD section. Apply PL components, tokens, and leave-local decisions from
   the PLAN output while implementing.
3. **Architecture — BUILD mode:** read [`architecture`](../architecture/SKILL.md)
   BUILD section. Satisfy applicable checklist items; cite precedents named in
   PLAN.
4. **Implement** the smallest valuable slice that meets AC. No TODOs, mocks
   (outside tests), `console.log`, or inline CSS.
5. **Stop** when AC in scope are met or a blocker needs a product decision —
   escalate to refine, do not guess.

## Exit criteria

- Code on branch matches refinement brief and in-scope AC.
- Design system PLAN decisions applied (or deviations documented).
- Architecture PLAN checklist items addressed in touched files.
- convention-first pipeline satisfied for changed files.

## Output

When build completes (or pauses), summarize:

```markdown
# Build summary: [ticket or title]

## Implemented

- [Bullet: what shipped vs AC]

## Standards applied

- DS: [PL components/patterns used — cite paths]
- Architecture: [themes satisfied — cite precedents followed]

## Deviations from plan

- [None, or explicit call with reason]

## Blockers

- [Product/dependency blockers only — empty if none]

→ Ready for phase 5 Verify
```

Do not run verify here — that is phase 5.
