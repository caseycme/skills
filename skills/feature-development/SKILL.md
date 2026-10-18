---
name: feature-development
description: >-
  Orchestrates the full feature lifecycle: Refine → Design system → Architecture
  → Build → Verify production readiness. Use when starting a story, implementing
  a feature end-to-end, or closing out work before PR. The parent coordinates
  child skills — it does not duplicate their rules.
---

# Feature Development

**Plan using the standards → build using the standards → verify against the
standards.**

This skill orchestrates only. Each phase delegates to a child skill. Do not
restate child workflows here.

| Phase | Skill         | Path                                               | Mode (child skill) |
| ----- | ------------- | -------------------------------------------------- | ------------------ |
| 1     | refine        | [`refine/SKILL.md`](refine/SKILL.md)               | —                  |
| 2     | design-system | [`design-system/SKILL.md`](design-system/SKILL.md) | **PLAN**           |
| 3     | architecture  | [`architecture/SKILL.md`](architecture/SKILL.md)   | **PLAN**           |
| 4     | build         | [`build/SKILL.md`](build/SKILL.md)                 | **BUILD**          |
| 5     | verify        | [`verify/SKILL.md`](verify/SKILL.md)               | production gate    |

Use [`feasibility`](../product/feasibility/SKILL.md) before phase 1 when the
team needs a ready/not-ready verdict on whether to start.

## Workflow

```
Planning progress:
- [ ] 1. Refine
- [ ] Decisions resolved (if any)
- [ ] 2. Design system (PLAN)
- [ ] 3. Architecture (PLAN)
- [ ] Feature development plan emitted
- [ ] 4. Build
- [ ] 5. Verify production readiness
- [ ] Ready for PR
```

### 1. Refine

**Invoke:** [`refine/SKILL.md`](refine/SKILL.md)

**Pass in:** ticket key or paste, Figma/design links, parent epic, chat context.

**Exit:** what/how clear enough to estimate; open decisions named or resolved;
consolidated refinement brief when decisions land.

**Gate:** stop for **Questions requiring a decision** before phase 2.

### 2. Design system

**Invoke:** [`design-system/SKILL.md`](design-system/SKILL.md) in **PLAN** mode.

**Pass in:** refinement brief, Figma/screenshots, routes/surfaces in scope.

**Exit:** reuse recommendations, prospective PL changes, sequencing (PL PR if
any). Or “not applicable (no UI)” in one line.

### 3. Architecture

**Invoke:** [`architecture/SKILL.md`](architecture/SKILL.md) in **PLAN** mode.

**Pass in:** refinement brief, design system plan (if any), routes/APIs/session
fields in scope.

**Exit:** applicable-theme checklist with repo precedents.

**After 1–3:** emit the [combined plan](#combined-plan-after-phases-13) once.

### 4. Build

**Invoke:** [`build/SKILL.md`](build/SKILL.md).

**Pass in:** refinement brief, design system PLAN output, architecture PLAN
checklist.

**Exit:** implementation on the branch following those standards.

### 5. Verify production readiness

**Invoke:** [`verify/SKILL.md`](verify/SKILL.md) — single pre-PR gate. Do not
run design-system or architecture REVIEW separately; verify owns them.

**Pass in:** refinement brief / AC, design system PLAN output, architecture
PLAN checklist, branch diff.

**Exit:** PASS | PARTIAL | FAIL, PR readiness, and what was checked/fixed.

## Orchestration rules

- **Child skills own behavior.** Read and follow the linked skill for each phase.
- **Pass context forward** — each phase consumes prior outputs.
- **One planning artifact** after phases 1–3; do not dump three full reports.
- **Verify requires code** — skip phases 4–5 during planning-only invocations
  unless implementation already exists.
- **Pre-PR is phase 5 only** — no separate “re-run architecture before PR.”

## When invoked

| Intent             | Run                                         |
| ------------------ | ------------------------------------------- |
| Plan a new story   | Phases 1–3 → combined plan                  |
| Implement feature  | Phase 4 (after 1–3, or with existing brief) |
| Close out / pre-PR | Phase 5                                     |
| Full lifecycle     | 1 → 2 → 3 → plan → 4 → 5                    |

## Combined plan (after phases 1–3)

Synthesize child outputs. Do not reprint full child reports.

```markdown
# Feature development plan: [ticket or title]

## Summary

[3–5 sentences from refine + blockers + estimate readiness]

## Refinement

[Delta findings, AC draft, follow-ups — from refine skill output]

## Design system (PLAN)

[From design-system PLAN mode — or "Not applicable"]

## Architecture (PLAN)

[From architecture PLAN mode checklist]

## Open decisions

[Empty if refine completed]

## Next

→ Phase 4 Build, then phase 5 Verify
```

## After verify (phase 5)

Append or return the verify skill output (see [`verify/SKILL.md`](verify/SKILL.md)).
Parent does not define verify sections — child owns the report.

## Child skills

- [`refine/SKILL.md`](refine/SKILL.md)
- [`design-system/SKILL.md`](design-system/SKILL.md) — PLAN | BUILD | REVIEW
- [`architecture/SKILL.md`](architecture/SKILL.md) — PLAN | BUILD | REVIEW
- [`build/SKILL.md`](build/SKILL.md)
- [`verify/SKILL.md`](verify/SKILL.md)
