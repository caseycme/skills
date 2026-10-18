---
name: verify
description: >-
  Production-readiness gate for feature work: re-checks implementation against
  acceptance criteria, design-system REVIEW, architecture REVIEW, repository
  conventions, deterministic checks (lint, types, tests, build), and browser
  validation. Phase 5 of feature-development. Use before opening a PR or when
  closing out a feature.
---

# Verify — production readiness

**Would we ship this?** Close the loop on the plan: the same standards used in
phases 2–3 are re-checked against the implementation.

Phase 5 of [`feature-development`](../SKILL.md). This is not “QA only” — it is
the final gate before PR.

## Inputs

- Refinement brief / acceptance criteria
- Design system **PLAN** output (from phase 2)
- Architecture **PLAN** checklist (from phase 3)
- Branch diff vs merge-base
- Build summary (if available)

## Workflow

Run in order. Track in:

```
Verify progress:
- [ ] 1. Refine requirements (AC)
- [ ] 2. Design system REVIEW (branch diff)
- [ ] 3. Architecture REVIEW (branch diff)
- [ ] 4. Deterministic code checks
- [ ] 5. Browser / AC validation (if UI)
- [ ] 6. Fix safe issues (if allowed)
- [ ] 7. Report PASS / PARTIAL / FAIL + PR readiness
```

### 1. Refine requirements check

- Map each in-scope AC to evidence: test, diff, browser check, or **unverified**.
- Flag AC gaps, scope creep, or conflicts with the refinement brief.
- Do not claim PASS on unverified AC.

### 2. Design system REVIEW

Invoke [`design-system`](../design-system/SKILL.md) in **REVIEW** mode on the
branch diff. Compare findings to design system PLAN:

- PLAN said reuse X — did implementation use X?
- PLAN said leave local — is local composition appropriate?
- New DS violations not anticipated in PLAN?

Summarize gaps only; do not duplicate the full DS report.

### 3. Architecture REVIEW

Invoke [`architecture`](../architecture/SKILL.md) in **REVIEW** mode on the
branch diff against the PLAN checklist. For each applicable theme:

- **Met** — precedent followed or check satisfied
- **Gap** — theme violated or unchecked
- **N/A** — unchanged by this branch

Summarize gaps only; do not duplicate the full architecture report.

### 4. Deterministic code checks

Run from repo root on changed scope (full repo if hooks/CI expect it):

```bash
npm run format-fix
npm run format
npm run lint
npm run ts-build
npm run test
npm run build
```

Also scan **changed files** for production violations:

- `TODO` / `FIXME` (outside tests)
- `console.log` / debug-only code
- Mocks or temporary shims outside tests
- Stray `style=` / inline CSS (cross-check DS REVIEW)

Record pass/fail per command. Do not skip format-fix before format.

### 5. Browser / AC validation

When the feature has a UI surface:

- Read [`validate-frontend`](../../validate-frontend/SKILL.md).
- Confirm `npm run dev` is running.
- Map observable AC → URL, actions, expected snapshot state.
- Run Playwright CLI; capture screenshot and console errors.
- Do not claim PASS on login wall, error alt-state, or missing UI.

When **no browser surface**: document unit/manual checks against AC instead.

### 6. Fix safe issues

When the user allows fixes during verify:

- Format/lint auto-fixes, missing imports, obvious convention violations
- Clear production blockers (TODOs, console.log) in changed files
- Do not expand scope, refactor unrelated code, or invent new behavior

Re-run failed deterministic checks after fixes.

### 7. Report

Use the output template below. **PASS** only when AC are verified, deterministic
checks pass, and no unresolved DS/architecture gaps that should block ship.

## Output

```markdown
# Verify: [ticket or title]

## Result

**PASS** | **PARTIAL** | **FAIL**

**PR ready:** yes | no

## Summary

[2–4 sentences: ship verdict and main blockers]

## Acceptance criteria

| AC  | Status              | Evidence              |
| --- | ------------------- | --------------------- |
| …   | met / gap / blocked | test / browser / diff |

## Design system REVIEW

- **vs PLAN:** …
- **Gaps:** … (or none)

## Architecture REVIEW

- **vs PLAN:** …
- **Gaps:** … (or none)

## Deterministic checks

| Check                      | Result    |
| -------------------------- | --------- |
| format                     | pass/fail |
| lint                       | pass/fail |
| ts-build                   | pass/fail |
| test                       | pass/fail |
| build                      | pass/fail |
| TODOs/mock/console in diff | pass/fail |

## Browser validation

- **Result:** PASS / FAIL / N/A
- **URLs:** …
- **Console errors:** …
- **Evidence:** `.playwright/output/…`

## Fixes applied

- [Or "none"]

## Remaining blockers

- [Or "none"]
```

**PARTIAL:** some AC verified or checks pass, but blockers remain (auth, flag,
missing test env, unresolved gap).

**FAIL:** deterministic checks fail, critical AC unmet, or ship-stopping
DS/architecture violation.
