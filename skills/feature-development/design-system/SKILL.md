---
name: design-system
description: >-
  Design-system guidance in three modes: PLAN (prospective reuse and
  abstractions before build), BUILD (apply those standards while implementing),
  REVIEW (audit branch diff against them). Use in feature-development phases 2,
  4, and 5; or standalone for inline styles, sx, custom CSS, one-off cards, or
  pattern-library scope. Formerly pattern-library-refactor.
---

# Design System

Same expertise at three stages. The orchestrator passes **mode** explicitly.

| Mode       | When                        | Question                                            |
| ---------- | --------------------------- | --------------------------------------------------- |
| **PLAN**   | feature-development phase 2 | What patterns/abstractions should this feature use? |
| **BUILD**  | feature-development phase 4 | Apply PLAN decisions while implementing.            |
| **REVIEW** | feature-development phase 5 | Does the implementation actually follow them?       |

Default to **REVIEW** when mode is unspecified and a branch diff exists.
Default to **PLAN** when mode is unspecified and there is no diff yet.

## PLAN mode

Before code exists. Inputs: refinement brief, Figma node or screenshot, routes
in scope.

1. Search **pattern-library** (`src/index.ts`, `src/theme.ts`, `src/core/`) and
   app-pro-ui for similar features.
2. Run the **abstraction test** (below) on each major UI element from design.
3. For each element: leave local | existing PL API | token/variant | PL PR
   needed — with cited paths.
4. Flag PL sequencing (PL PR before/parallel with app-pro-ui) and missing
   copy/assets.

If no UI surface: return **Not applicable (no UI)** in one line.

**Output:** `# Design system (PLAN): [feature]` — reuse list, prospective
changes, sequencing. No branch diff required.

## BUILD mode

During implementation. Inputs: design system PLAN output, files being edited.

1. Apply PLAN decisions: use named PL components/tokens; respect leave-local
   calls.
2. Follow the **reasoning layer** below — do not recreate reusable looks in
   feature code.
3. If PLAN is wrong given what you learn while coding, document the deviation
   in the build summary; do not silently drift.

**Output:** none required unless deviation — then note in build summary.

## REVIEW mode

After implementation. Scan **this branch** (app-pro-ui or pattern-library) for
UI decisions that should live in the design system. Do not only say “delete the style” or “use a shared component.” Prefer the smallest **global** pattern-library change that lets any feature do the same thing—or recommend leaving the local implementation when abstraction is not justified.

This is not “all styles belong in pattern-library.” Repeated product/UX decisions should increasingly be encoded into the system so individual builders do not reinvent them. Feature code may still compose locally.

Architecture rule: Reusable visual decisions belong in pattern-library. Feature code may use `sx` for genuinely local layout/composition, but should not recreate reusable component styling, tokens, states, or interaction patterns. Avoid `style`, custom CSS, and legacy MUI system props when an established PL/MUI API exists.

This skill is the DS layer. For app-pro-ui hook/util/grid structure and generic code hygiene, use the `refactoring` skill.

## Reasoning layer

Judge every finding with these rules. They come from years of pattern-library and app-pro-ui review comments. Prefer an existing API over a new one. Moving code into pattern-library is not automatically an improvement.

### Abstraction test

Run this on every candidate recipe, token, or new API. It is the primary judgment—detection only produces candidates.

1. Is this local composition?
2. Does the same recipe appear elsewhere?
3. Is the component already supposed to own this?
4. Would moving it to pattern-library reduce meaningful future decisions?
5. Would adding an API actually make pattern-library worse?

Similarity alone does not justify abstraction. Two snippets that look alike may represent different product concepts. Prefer shared abstractions when the same product/UX decision should evolve together.

`sx` is not the problem. The issue is where the design decision lives. Theme-token `sx` is better than hex. A repeated recipe may belong in pattern-library. A one-off `<Stack sx={{ mt: 2 }}>` usually does not. If the test says the local solution is correct, omit it from findings—do not list it as Low.

### 1. Use the design system — do not restyle reusable decisions in the feature

- Prefer established pattern-library components for reusable product patterns (`CardWithAction` even without an action, `CardCTAFeatured` as a compound card, `DetailPanelToggle`). Do not rebuild those from raw `Card`.
- Prefer MUI composition primitives such as `Stack`/`Grid` over custom layout CSS when the layout is local.
- Use established layout primitives and defaults before inventing local geometry.
- Put reusable color, shadow, and marketing/AI treatments in `pattern-library/src/theme.ts` as palette-shaped tokens (`primary.main`, `ai.gradient.start`). Prefer a token or `theme.shadows` over hardcoded hex, fills, or shadows when the value is a product decision.
- Use `sx`, not deprecated MUI style-as-props. Do not add style-named props on shared components to work around that. Local `sx` is fine when it is composition, not a reusable look.
- If a feature needs a new look that other surfaces will share (featured media, expand overlay, inverted/marketing surfaces), add a pattern or prop in pattern-library.

### 2. Keep shared components generic

- Do not land feature-specific padding, sidenav, or chart changes on shared `Card`, `PageSection`, or public `index.ts` exports. Scope prototypes; revert unrelated files. Change the item, not the chrome, when only the item is wrong.
- Name props for reuse (`footerDescription`, not a one-feature label). Extra behavior is optional (line numbers, commas on a textarea).
- Prefer theme variants and Storybook `args` over forked JSX (`variant === "marketing"` checks).
- Do not export charts or other internals from pattern-library if apps do not consume them.
- Longer term, wrap a surface in a theme (e.g. marketing) instead of threading one-off visual props through every page.

### 3. Visual QA on the edges

- Check rounded corners, equal-height cards, orphaned wrapped text, mobile FAB collapse, z-index vs drawers/modals, and whether a control still works on a white background.
- Ask whether special spacing is required or default styles already match other modals/pages. Do not pad a shared header to line up with a misaligned neighbor—fix the neighbor.

### 4. Question the design requirement

Before adding a DS capability to support an unusual design, determine whether the design itself should conform to an established system behavior. Do not expand the design system merely to encode a questionable one-off requirement.

### 5. Resist unnecessary API surface

Every proposed DS abstraction must justify the API surface it adds. Moving code into pattern-library is not automatically an improvement. Configuration that does not improve the experience is maintenance cost.

Keep DS-architecture hygiene only: unnecessary wrappers that interfere with composition, duplicated styling, component API complexity, and feature-specific logic in shared primitives. Flatten nested `Box`es when they get in the way of `Stack`/`sx` composition. Use theme shorthand (`background.main`) instead of `useTheme` for simple tokens.

### 6. Ship with the related surface

- A pattern-library change that apps must consume needs a matching app-pro-ui or moz-ui PR and a way to verify it (amplify/domain or equivalent).
- Confirm the current package/version convention from the consuming repo (`package.json`, lockfile, or install docs) before recommending how to bump. Do not assume a pin style.
- Do not merge a shared primitive until the consuming app PR is ready.

## When invoked (REVIEW mode)

Copy and track:

```
Review Progress:
- [ ] Diff this branch against the merge-base
- [ ] Collect candidates (not conclusions)
- [ ] Cluster by intent (same decision, not same code)
- [ ] Run the abstraction test
- [ ] Search pattern-library for an existing home
- [ ] Recommend the smallest change — including leave local / no DS change
- [ ] Report findings; do not implement unless asked
```

Finding work is not required. If the diff is appropriately local, say so and stop.

## Step 1: Scan the branch

Inspect **this branch vs its merge-base**. Detect the default branch (`git symbolic-ref refs/remotes/origin/HEAD` or `origin/main`, then `main`). Run from the repo that has the changes (`app-pro-ui` or `pattern-library`).

```bash
DEFAULT=$(git symbolic-ref refs/remotes/origin/HEAD 2>/dev/null | sed 's@^refs/remotes/@@')
DEFAULT=${DEFAULT:-origin/main}
git merge-base "$DEFAULT" HEAD
git diff $(git merge-base "$DEFAULT" HEAD)...HEAD -- '*.tsx' '*.ts' '*.jsx' '*.js' '*.css' '*.scss'
git diff --name-only $(git merge-base "$DEFAULT" HEAD)...HEAD
```

Search added lines for **candidates**:

**Styling**

- `style=`
- `sx=`
- `styled(`
- ` css\`` /  `css(`
- `makeStyles` / `createStyles` / `withStyles`
- hardcoded `#hex`, `rgb(`, `rgba(`, `px` (investigate; do not auto-classify)
- legacy Box props: `display=`, `alignItems=`, `justifyContent=`, `flexDirection=`

**Reinvented or drifted UI**

- Raw `Card` where an established card pattern (`CardWithAction`, `CardCTAFeatured`) fits
- Custom expand controls instead of `DetailPanelToggle`
- Custom layout CSS where `Stack`/`Grid` would compose locally
- Invented page geometry where layout primitives or defaults already set width
- Hardcoded shadows instead of `theme.shadows` when the shadow is a shared look
- New public exports or theme edits that only serve one feature

Then open each hit and read surrounding JSX for intent. Also catch `sx` objects assigned before JSX, `<style>` tags, and imported stylesheets.

Ignore `.cursor/`, `node_modules/`, `.next/`. Include feature code, shared `components/`, Storybook, and `src/theme.ts`.

## Step 2: Detect, then judge

Detection is a heuristic. Classification happens after the abstraction test.

| Detector                                                                                                                                              | Treat as                           | Examples                                                       |
| ----------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------- | -------------------------------------------------------------- |
| `style=` / `styled(` / `css` / new `.css`                                                                                                             | Candidate leak                     | `<Card style={{ flexBasis: "50%" }}>`                          |
| Hardcoded visual values that appear to encode design-system decisions, especially color, spacing, typography, radius, shadow, and repeated dimensions | Candidate token                    | `color: "#002F50"`; investigate `px` before treating as a leak |
| `sx={{ ... }}` using theme keys                                                                                                                       | Candidate recipe                   | `sx={{ mt: 3, width: "100%" }}`                                |
| Legacy Box system props                                                                                                                               | Candidate legacy                   | `<Box display="flex" alignItems="center">`                     |
| Existing PL component unused                                                                                                                          | Candidate existing system solution | table not wrapped in `CardWithAction`                          |
| Feature-specific edit on a shared component                                                                                                           | Candidate scope leak               | padding on `CardHeader` to match one story                     |
| One-feature prop or export                                                                                                                            | Candidate non-generic API          | feature-named footer prop; exporting unused charts             |

Investigate before classifying hardcoded values. Legitimate local cases include calculated dimensions, chart geometry, icon/canvas internals, borders, virtualized layouts, and intrinsic component constraints.

Last-resort exceptions (document why; do not invent a primitive):

- Third-party / canvas / chart internals that cannot use pattern-library props
- Print/PDF layout with no existing report primitive _and_ truly one-off
- Prototype-only files that are not exported from pattern-library

Candidates that pass the abstraction test as local composition are not findings. Put them under **Do not promote** only if they would otherwise be mistaken for gaps. Otherwise omit them.

## Step 3: Cluster by intent

Group hits that express the same **decision**, not the same CSS or similar-looking markup.

Typical intents: color/emphasis, spacing/alignment recipe, typography, component chrome, interaction state, feature pattern (banner, page header, card CTA, status), generic vs one-feature API, prototype scope, questionable design requirement.

One cluster → one recommendation. Prefer one strong recommendation over several minor variants of the same finding.

## Step 4: Search pattern-library first

Search the sibling `pattern-library` repo before proposing anything new. Cite paths.

1. Public exports — `pattern-library/src/index.ts`
2. Tokens and global defaults — `pattern-library/src/theme.ts`
3. Component props / variants under `pattern-library/src/core/`
4. Existing app-pro-ui usages of the same component that already avoid the leak

Precedent for **absorbing UI into the DS** (cite these; do not copy internals into app-pro-ui):

- Semantic prop: `Typography` `lineClamp` — `pattern-library/src/core/Typography/Typography.tsx`
- Semantic prop: `Button` `aiGlow` — `pattern-library/src/core/Button/Button.tsx`
- Semantic prop: `PageHeader` `gradientBackground` — `pattern-library/src/core/PageHeader/PageHeader.tsx`
- Established cards: `CardWithAction` and compound `CardCTAFeatured` — `pattern-library/src/core/cards/`
- Primitive: `LineClamp` — `pattern-library/src/core/LineClamp/LineClamp.tsx`
- Theme variant: `Card` `inverted` / `marketing` — `pattern-library/src/theme.ts`
- Theme default: `MuiStack` `spacing: 3` — `pattern-library/src/theme.ts`

If the API already exists, the finding is an **existing system solution**. Do not recommend a new pattern-library change.

## Step 5: Recommend the smallest change

Pick the first option that fits. Leave local is a first-class outcome.

1. **Leave local** — implementation is appropriately feature-specific, sufficiently simple, or not worth expanding the shared API
2. **Existing system solution** — pattern-library already has the component, variant, or prop
3. **Theme token** — palette, typography, spacing, or `theme.shadows` in `src/theme.ts`
4. **Variant or semantic prop** on an existing component (preferred over a new file)
5. **Theme component default / styleOverride** when every instance should look this way
6. **New primitive** only when the behavior represents a coherent reusable concept with no appropriate existing home. Existing duplication strengthens the case but is not required
7. **Revert / narrow scope** when the change does not belong on the shared component (prototype-only, or fix the neighbor)

Do **not** recommend:

- Exporting a shared `sx` object from app-pro-ui
- A new app-pro-ui wrapper that still hardcodes reusable styling
- A new pattern-library component when a prop or theme variant is enough
- Feature-named props or public exports apps do not use
- Expanding the DS to encode a questionable one-off design
- Invented APIs — if the right home is unclear, say so and ask

Each recommendation that is **not** leave-local must include:

- The intent (one sentence)
- Which reasoning-layer rule applies
- Evidence from the branch (paths + line numbers)
- What already exists in pattern-library (or “none found”)
- The proposed change
- How the app or Storybook would look after (`sx` is fine for leftover local composition)
- Related-surface follow-up when a PL change is required (consuming-app PR; version bump per current repo convention)

A leave-local recommendation needs intent, why the local solution is appropriate, and what would change that call later (reuse, a clearer product pattern). If the whole diff is leave-local, the report can be a short summary and no clusters.

## Output

Keep the report proportional to the diff. Prefer 1 strong recommendation over 6 minor variants of the same finding. Omit Inventory unless there are enough distinct findings to justify it. Do not generate findings for things the abstraction test judged correct.

```markdown
# Design System: [branch name]

## Summary

[1–3 sentences: what matters, including “no DS change” when that is the answer]

## Clusters and recommendations

### 1. [Intent name]

**Reasoning:** [abstraction test + theme]
**Evidence:** `path:L`, `path:L`
**Exists in pattern-library:** [cite path or “none found”]
**Recommendation:** Leave local | Existing system solution | Token | Variant/prop | Theme default | New primitive | Revert/narrow
**Change:** [concrete API, token, or “none”]
**After:** [what feature or Storybook code should use]
**Related surface:** [consuming app / none]

## Do not promote

- [finding] — [local composition, exception, or out of scope]

## Next actions

1. [existing system cleanup, if any]
2. [pattern-library follow-up, if any]
3. [consuming-app PR / version convention, if any]
```

Include **Inventory** only when several distinct gaps would otherwise be hard to scan:

| Severity     | File     | Intent   | Candidate                                               |
| ------------ | -------- | -------- | ------------------------------------------------------- |
| High/Med/Low | `path:L` | [intent] | `style` / unused PL API / scope leak / non-generic prop |

Severity (after judgment, not detection). Only issues that should change go here:

- **High** — reusable look recreated with `style`/`styled`/custom CSS, or a feature leak into a shared component
- **Medium** — a recipe the system should own; unused existing PL API; non-generic prop
- **Low** — legitimate issue with small impact, such as a reusable recipe that has limited current scope

## Constraints

- Read-only unless the user asks to implement.
- Work from the branch diff only; do not audit unrelated existing files.
- Never invent pattern-library components, tokens, or props.
- Do not flag local `sx` composition as architecture debt.
- English only.
