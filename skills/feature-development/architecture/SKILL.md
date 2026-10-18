---
name: architecture
description: >-
  Architectural guidance from historical lead PR comments in three modes: PLAN
  (checklist and precedents before build), BUILD (apply while implementing),
  REVIEW (evaluate branch diff against the checklist). Use in feature-development
  phases 3, 4, and 5. Not a replacement for design-system or deterministic
  verify checks.
---

# Architecture

Lead review themes distilled from
[`.cursor/app-pro-ui-pr-lead-comments-2024-2026.md`](../../../app-pro-ui-pr-lead-comments-2024-2026.md)
(1,226 comments, 2024–2026). Grep the dump only for feature-specific examples.

Same expertise at three stages. The orchestrator passes **mode** explicitly.

| Mode       | When                        | Question                                       |
| ---------- | --------------------------- | ---------------------------------------------- |
| **PLAN**   | feature-development phase 3 | What constraints and precedents apply?         |
| **BUILD**  | feature-development phase 4 | Satisfy the PLAN checklist while implementing. |
| **REVIEW** | feature-development phase 5 | Does the diff actually meet those constraints? |

Default to **PLAN** when mode is unspecified and planning; **REVIEW** when a
branch diff is being verified.

## PLAN mode

Inputs: refinement brief, design system PLAN (if any), routes/APIs/session in
scope.

1. Select applicable themes from **Review themes** below.
2. Grep comment dump for 0–3 feature-specific signals (optional).
3. Find **one repo precedent per applicable theme**.
4. Emit checklist ordered by review risk (access/gating first).

**Output:** `# Architecture (PLAN): [feature]` — use template under
**PLAN output** below.

## BUILD mode

Inputs: architecture PLAN checklist, files being edited.

1. For each applicable theme in PLAN, follow the cited precedent in new code.
2. When touching session, data hooks, or shared utilities, re-read the matching
   theme checks.
3. Document deviations in the build summary; do not silently drift.

**Output:** none required unless deviation.

## REVIEW mode

Inputs: architecture PLAN checklist, branch diff.

1. For each applicable theme from PLAN, mark **Met**, **Gap**, or **N/A** with
   diff evidence.
2. Optional: grep comment dump for 0–3 signals if gaps need lead-context
   examples.
3. Do not invent new themes — evaluate against PLAN + **Review themes** below.

**Output:** `# Architecture (REVIEW): [feature]` — use template under
**REVIEW output** below. Invoked by [`verify`](../verify/SKILL.md), not as a
standalone pre-PR step.

## Rules

- Bias toward **checklists**, not essays. One line per check unless an example
  is necessary.
- Do not invent themes not grounded in **Review themes** below or the comment
  dump.
- Do not duplicate the full refine brief or a file-by-file implementation plan.
- Pre-PR evaluation is **REVIEW mode inside verify** — do not run REVIEW as a
  separate gate before verify.

## Theme selection guide

| Feature touches…         | Prioritize themes                                                  |
| ------------------------ | ------------------------------------------------------------------ |
| New route or `_module/`  | `hooks-and-module-structure`, `reuse-and-precedent`                |
| Session, flags, quotas   | `app-api-and-session`, `access-gating`, `testing-and-verification` |
| UI components            | `pattern-library-ds`, `mui-layout`                                 |
| API hooks / lists        | `data-fetching`, `types-and-boundaries`, `edge-cases`              |
| Tables                   | `datagrid-tables`, `types-and-boundaries`                          |
| Forms                    | `forms`, `edge-cases`                                              |
| Metrics / dates / locale | `i18n-formatting`, `edge-cases`                                    |
| Events                   | `analytics`                                                        |
| Large or mixed PR        | `scope-and-pr-hygiene`                                             |

## Review themes

Distilled from lead PR comments. Counts are comment matches, not unique PRs.

### Types and DTO boundaries (`types-and-boundaries`) — 88

- Prefer shared types and aliases over repeating inline unions.
- Validate API shapes at boundaries; avoid sprinkling optional fields without a plan.
- Name types for reuse when the same shape appears in multiple files.

_Example (PR #554):_ “Thoughts are either a new type, or just changing
TargetObjectInput…”

### Hooks, naming, and `_module` structure (`hooks-and-module-structure`) — 86

- Place feature logic under `app/(default)/[feature]/_module/` with camelCase hooks.
- Name hooks and helpers for what they gate or fetch, not implementation detail.
- Use barrel exports where the feature already does.

_Example (PR #498):_ “if we just name this something like `hasSiteSleuthAccess`
then it's obvious what its for…”

### pattern-library and design system (`pattern-library-ds`) — 73

- Use pattern-library components for reusable product patterns; no inline CSS or ad-hoc cards.
- Check pattern-library for truncation, typography, and layout before adding local workarounds.
- If pattern-library must change, coordinate version bump and consuming PR.

_Example (PR #1386):_ “should both actually be using `Typography` with `noWrap`
and a max width…”

### Testing and manual verification (`testing-and-verification`) — 67

- Add or update unit tests when behavior is non-trivial or easy to regress.
- Manually verify session-sensitive paths with the account types in AC.
- Document how you tested in the PR when setup is non-obvious.

_Example (PR #498):_ “make sure you've tested on a non admin account…”

### Edge cases and undefined data (`edge-cases`) — 66

- Define behavior when nested fields are undefined or API returns partial data.
- Avoid assumptions that finish before request completes (target filled async).
- Call out date/timezone and default-value edge cases explicitly.

_Example (PR #554):_ “`<data area>.target` is undefined on request, and is
filled in when the request finishes…”

### app-api helpers and session truth (`app-api-and-session`) — 65

- Use app-api/client helpers for flags, access, and quotas — do not re-derive in the UI.
- Verify behavior on admin vs non-admin and community vs Pro accounts.
- Confirm feature-flag checks match `util/auth/feature-flags.ts` patterns.

_Example (PR #498):_ “maybe we use the `hasAdminRestrictedAccess` from
`app-api/client`?”

### Access, tiers, and gating (`access-gating`) — 49

- Test all relevant account types called out in AC (admin, community, Pro, quota-limited).
- Do not treat `isPro` or `isMozProActive` as a substitute for feature-specific access when quotas apply.
- Call out intentional behavior changes for lower tiers before merge.

### MUI layout and sx (`mui-layout`) — 48

- Prefer Stack/Grid/sx over inline styles; avoid deprecated Box style props.
- Let layout primitives (PageSection, Stack) own spacing — do not double-wrap.
- Use Typography from pattern-library/MUI instead of raw span/div for text.

_Example (PR #498):_ “Doesn't the section handle the spacing between the
components…”

### DataGrid and tables (`datagrid-tables`) — 23

- Follow existing DataGrid patterns for sort, pagination, and column defs.
- Align column alignment and pinning with Figma and sibling tables.
- Keep view-model mapping separate from render when the feature already does.

### Scope and PR hygiene (`scope-and-pr-hygiene`) — 21

- Keep the PR to the ticket slice; move unrelated refactors to follow-ups.
- Remove TODOs, debug flags, and temporary mocks before merge.
- Revert or flip feature flags and env toggles called out in review.

### Reuse existing patterns (`reuse-and-precedent`) — 20

- Search the repo for a similar feature before adding new abstractions.
- Cite the file or hook you are following in the PR description.
- Prefer extending an existing helper over a parallel implementation.

_Example (PR #498):_ “this already exists as part of our feature flag stuff in
app-api…”

### Analytics and tracking (`analytics`) — 19

- Use `util/tracking` constants and shared event helpers.
- Confirm new events with product/analytics naming conventions.
- Decide whether tracking belongs in this ticket or a follow-up.

### React Query and data fetching (`data-fetching`) — 17

- Follow established query-key and hook patterns from the feature or `util/react-query`.
- Handle loading, empty, error, and partial-data states explicitly.
- Check downstream consumers when changing shared query hook return shapes.

_Example (PR #758):_ “look at usages of `util/quota/useQueryWithQuota.ts` to
make sure they are getting the right things returned…”

### Formatting and locale (`i18n-formatting`) — 14

- Use shared number/date formatting utilities.
- Pass scope and locale through when creating or fetching campaign/domain data.
- Do not hardcode market or engine assumptions that vary by account.

### Performance and render stability (`performance`) — 13

- Avoid unstable deps that force hooks to run every render.
- Question full-page refresh or refetch on local state changes.
- Do not over-memoize; follow precedent in the touched feature.

### Forms and validation (`forms`) — 8

- Use React Hook Form patterns from the feature or `components/`.
- Validate on the right trigger (field vs submit) per product rules.
- Keep form state out of React Query unless precedent exists.

## PLAN output

```markdown
# Architecture (PLAN): [feature or ticket]

## Feature context

[2–3 sentences]

## Applicable themes

### [theme id] — Applicable | Not applicable

- [ ] Check from review themes
- Precedent: `path`

## Feature-specific signals

- [0–3 bullets from comment dump, optional]
```

Skip empty sections. **Not applicable** themes: one line each.

## REVIEW output

```markdown
# Architecture (REVIEW): [feature or ticket]

## vs PLAN checklist

| Theme | Status          | Evidence     |
| ----- | --------------- | ------------ |
| …     | Met / Gap / N/A | path or note |

## Gaps

- … (or none)
```

## Refreshing themes

When the comment dump is updated, re-extract themes into **Review themes**
above from
[`.cursor/app-pro-ui-pr-lead-comments-2024-2026.md`](../../../app-pro-ui-pr-lead-comments-2024-2026.md).
Keep the same theme ids and check-list shape so checklists stay comparable.
