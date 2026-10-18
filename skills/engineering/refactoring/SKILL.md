---
name: refactoring
description: Guides systematic refactoring to align features with established patterns, remove duplication, and improve code reuse. Includes DataGrid/pattern-library, typing, fetch pagination, view-model vs override-prop guidance, and MUI sx-first layout (avoid deprecated Box style shorthand props). Use when refactoring features, consolidating duplicate code, extracting shared utilities, or aligning patterns across similar features.
---

# Refactoring Skill

Use this skill when refactoring features to align with established patterns, remove duplication, and improve code reuse.

For whether UI belongs in pattern-library vs local composition, use the `design-system` skill.

## Pre-Refactoring Analysis

### 1. Identify Similar Features

- Find comparable features in the codebase with similar functionality
- Compare directory structures, component patterns, and utility usage
- Look for features with similar functionality but different implementations
- Use semantic search to find related features: "features with similar table/data grid patterns"

### 2. Find Duplication Opportunities

Search for duplicate code patterns:

- **Components**: Look for similar component files across features (same name or similar functionality)
- **Hooks**: Check for duplicate hook logic (table hooks, filter hooks, aggregation hooks)
- **Utilities**: Find duplicate utility functions (aggregations, formatters, validators, transformers)
- **Styles**: Identify duplicate `sx` prop patterns or inline styles
- **Constants**: Check for duplicate constants or configuration objects
- **Types**: Look for duplicate type definitions or interfaces

### 3. Check Shared Utilities

Before creating new utilities, verify existing ones:

- Search `@/util/` directories for domain-specific utilities
- Check `@/util/dates` for date formatting functions
- Check `@/util/numbers` for number formatting functions
- Check `@/components/` for reusable components
- Check `pattern-library` for design system components
- Use grep/search to find existing implementations before creating new ones

## Refactoring Process

### Step 1: Extract Shared Code

When finding duplicates:

1. **Create shared utilities** in appropriate `@/util/` directories:

   ```typescript
   // Example: util/[domain]/[category].ts
   export const sharedFunction = () => { ... };
   export const sharedConstant = { ... };
   ```

2. **Update all features** to import from shared location:

   ```typescript
   // Feature-specific file - re-export shared
   export { sharedFunction, sharedConstant } from "@/util/[domain]/[category]";
   ```

3. **Remove duplicate implementations** from feature directories
4. **Update all imports** across the codebase

### Step 2: Consolidate Components

When finding duplicate components:

1. **Identify the most complete/feature-rich version** (check props, functionality, tests)
2. **Extract to shared location** (`@/components/` or feature-agnostic location)
3. **Make props flexible** to support all use cases (use union types, optional props, or generics)
4. **Update imports** in all features using the component
5. **Delete duplicate component files**
6. **Verify all usages** still work correctly

### Step 3: Align Loading Patterns

For loading states:

1. **Check `loading.tsx` files** - should be simple skeletons, not full page layouts
2. **Remove component-level skeleton loaders** - rely on page-level `loading.tsx` for initial load
3. **During refetches**: Show existing data, not skeletons (unless explicitly needed for UX)
4. **Match patterns** from similar features in the codebase
5. **Use Next.js App Router conventions** - `loading.tsx` handles initial load, components handle refetch states

### Step 4: Consolidate Styles

For styling:

1. **Extract reusable style patterns** to shared components or style utilities
2. **Remove inline `sx` props** when a shared component exists
3. **Use design system components** from `pattern-library` instead of custom styles
4. **Check for duplicate style objects** that could be shared utilities or theme extensions
5. **Follow Material-UI patterns** - use theme values, avoid magic numbers

### Step 5: Verify Pattern Alignment

Ensure consistency with established features:

1. **Compare directory structures** - should follow similar patterns (`_components/`, `_hooks/`, `_utils/`)
2. **Check hook patterns** - simpler hooks are fine if feature requirements differ
3. **Verify data flow** - should match similar features where possible
4. **Review component composition** - should use similar patterns (wrappers, composition, etc.)
5. **Check naming conventions** - follow established patterns (PascalCase for components, camelCase for hooks)

### Step 6: Utility Function Consolidation

For utility functions:

1. **Check if function exists** in shared utilities first (search codebase)
2. **Compare implementations** - use the more complete/robust version
3. **Consolidate duplicates** - move to shared location, update all imports
4. **Document differences** - if implementations differ intentionally, document why
5. **Preserve functionality** - ensure consolidated version handles all use cases

## Step 7: Clean Up Comments

After refactoring, remove unnecessary comments:

1. **Remove migration notes** - Comments explaining where code was moved from (e.g., "isCollecting is now exported from Table/utils.ts")
2. **Remove obvious comments** - Comments that just restate what the code does (e.g., "// Re-export shared aggregations" when the code clearly shows re-exports)
3. **Keep explanatory comments** - Comments that explain WHY something is done (e.g., "// Use setTimeout to ensure form state is updated")
4. **Keep complex logic comments** - Comments that explain non-obvious business logic or edge cases
5. **Remove TODO/FIXME comments** - Unless they're legitimate future work items

**Examples of unnecessary comments:**

```typescript
// ❌ Unnecessary - obvious from code
// Re-export shared aggregations from util
export { latest, totalRange } from "@/util/keywords/gridAggregations";

// ❌ Unnecessary - migration note, no longer needed
// isCollecting is now exported from Table/utils.ts to avoid duplication
// Import it from there: import { isCollecting } from "../KeywordListDetail/Table/utils";
```

**Examples of good comments:**

```typescript
// ✅ Good - explains WHY
// Use setTimeout to ensure form state is updated after CSV processing
setTimeout(() => {
  void handleSubmit(onSubmit)();
}, 100);

// ✅ Good - explains complex logic
// A keyword is updated if:
// 1. Its lastUpdated value changed (most reliable indicator), OR
// 2. It's no longer in collecting state AND we've polled enough times (fallback)
```

## Step 8: Code Quality Improvements

Apply consistent code quality patterns throughout refactored code:

### Null/Undefined Checks

Simplify null and undefined checks using loose equality:

```typescript
// ❌ Verbose
if (value !== undefined && value !== null) { ... }
if (value !== undefined && value !== null && value !== "") { ... }

// ✅ Simplified
if (value != null) { ... }
if (value != null && value !== "") { ... }
```

**Note**: Use `!= null` (loose equality) to check for both `null` and `undefined` in a single check. This is idiomatic TypeScript/JavaScript and more concise.

### Type-Only Imports

Use `import type` for type-only imports to improve build performance and make intent clear:

```typescript
// ❌ Runtime import for type-only usage
import { SomeType } from "app-api/types/domain";

// ✅ Type-only import
import type { SomeType } from "app-api/types/domain";
```

**Apply to**:

- Type definitions from `app-api`
- Interface imports
- Type-only utility types
- Any import used only in type annotations

### Remove Unnecessary Type Assertions

Remove type assertions when TypeScript can infer the type correctly:

```typescript
// ❌ Unnecessary assertion
const items = data.map((item) => item as SomeType);

// ✅ Let TypeScript infer
const items = data.map((item) => item);
```

**When to keep assertions**: Only when TypeScript genuinely cannot infer the type, or when narrowing types in a way TypeScript can't verify.

### Remove Unnecessary Nullish Coalescing

Remove `?? null` when the value is already nullable or TypeScript infers the correct type:

```typescript
// ❌ Unnecessary - valueGetter already returns nullable
valueGetter: (value, row) => {
  return row.someField ?? null;
};

// ✅ Simplified
valueGetter: (value, row) => {
  return row.someField;
};
```

**When to keep**: When explicitly converting `undefined` to `null` for API compatibility or when the type system requires it.

### Remove Unused Props

Remove props from component interfaces and implementations when they're not used:

```typescript
// ❌ Unused prop
interface ComponentProps {
  item: Item;
  unusedField: string; // Never used in component
}

// ✅ Clean interface
interface ComponentProps {
  item: Item;
}
```

### Add Explicit Types to Callbacks

Add explicit type annotations to callback parameters when TypeScript can't infer them:

```typescript
// ❌ Implicit any
const items = ids.map((id) => { ... });

// ✅ Explicit type
const items = ids.map((id: string | number) => { ... });
```

**Apply to**: Array methods (`map`, `filter`, `forEach`), event handlers, and other callbacks where types aren't clear from context.

### Extract Repeated JSX Patterns

Extract repeated JSX patterns into reusable components:

```typescript
// ❌ Repeated pattern
{
  isAggregation ? (
    <Typography
      variant="body2"
      fontWeight="medium"
      fontStyle="italic"
      color="text.secondary"
    >
      {formattedValue}
    </Typography>
  ) : (
    <Typography variant="body2" fontWeight="medium">
      {formattedValue}
    </Typography>
  );
}

// ✅ Extracted component
<AggregationTypography
  formattedValue={formattedValue}
  isAggregation={isAggregation}
/>;
```

### Use Factory Functions for Varied Implementations

When logic is similar but needs to handle different data structures, use factory functions:

```typescript
// ✅ Factory function pattern
export const createIntentOperators = <T extends { intent?: Intent | Intent[] }>(
  normalizeToArray: (value: T) => Intent[]
) => {
  return {
    // Operators that work with normalized array
  };
};

// Usage in different features
const operators1 = createIntentOperators<Feature1Type>((item) => [item.intent]);
const operators2 = createIntentOperators<Feature2Type>(
  (item) => item.intents || []
);
```

**Benefits**: Reuses core logic while allowing different data structures to be handled appropriately.

## Code review hardening (recurring feedback)

Apply these when refactoring touches grids, dashboards, fetches, or shared “row + metrics” shapes (aligns with repeated Moz Pro UI PR feedback).

### DataGrid: detail panel toggle

- **Prefer `DetailPanelToggle` from `pattern-library`** for expand/collapse on a detail-panel column instead of ad hoc `IconButton` + `@mui/icons` in a custom cell.
- **Wire accessibility**: pass `buttonTabIndex={0}` when the cell should be tab-focusable (see `app/(default)/competitive-research/top-competing-content/_module/components/Table/columns.tsx`, `SerpFeaturesCell`).
- **Layout / centering**: use **`sx`** on `Box`/`Stack` (or `Stack`’s `direction` / `spacing` when that’s the established pattern). **Do not** replace `sx` with legacy MUI **system style props** on `Box` (e.g. `display`, `alignItems` as top-level props)—MUI is **deprecating that approach** in favor of **`sx`**.

### Typing: avoid `object` / anonymous row shapes

- **Do not** thread `readonly object[]` for `brand_definitions`, row fragments, or API-adjacent lists when a structural type exists or can be defined once.
- **Define a shared “like” interface** (e.g. `BrandDefinitionLike`) in the feature `utils` module; **export** it and a single **display-line** helper (e.g. `getBrandDefinitionDisplayLine`) instead of duplicating `getBrandName` with casts inside cells.
- **Remove redundant casts** (`as BrandDefinitionLike[]`) after parameters are typed.
- **Reuse domain helpers** from `*_module/utils/dataHelpers` when the shape is a real `BrandDefinition`; use the looser helper only where prompt-row payloads differ.

### Typography vs `Box` (pattern-library)

- For **text** (empty states, inline “—”, emphasized spans), **prefer `Typography`** from `pattern-library` with `component="span"` when appropriate—not `Box` pretending to be copy.
- **`Typography` API**: use **`variant`**, **`color`**, **`fontWeight`**, **`component`** where they express semantics (these are not deprecated).
- **`Box` / layout wrappers**: prefer **`sx`** for spacing, flex layout, sizing, and theme values. **Avoid** drive-by refactors that move styling from **`sx`** to **`Box`** “style” shorthand props; that conflicts with current MUI guidance.

### Component boundaries: props vs internal state

- **Derive booleans from `data`** inside the component when the parent does not need them for layout or URL state (e.g. “has brand terms” from primary brand’s `brand_terms`).
- **Keep UI-only toggles internal** (e.g. switching between two chart modes in one card) unless another sibling or route must drive the same state—then document why lifting is required.

### Control flow and JSX

- Prefer **early `return`** for loading/empty branches instead of a **large ternary** assigned to a variable immediately before a single `return`, unless memoization genuinely needs one expression.
- **Sort comparators**: never hardcode a magic sentinel (e.g. `5`) tied to an assumed domain maximum; use **`Number.POSITIVE_INFINITY` / explicit “sort last”** for null/undefined so behavior stays correct if limits change.

### Fetches and pagination

- **Avoid a fixed small `limit`** (e.g. 100) for dashboard-scale lists when the backend is paged and projects can exceed one page.
- **Loop** with a stable page size until a page returns **fewer rows than the limit** or **zero rows**, or follow the action’s documented contract (see `moz-pro-service` action docs / Bruno for max `limit`).
- Surface **`total`** or derived totals consistently if callers need counts.

### Presentation wrappers: don’t repeat the obvious

- Drop **redundant alt-state fields** when the parent `Card` already sets the visible title (e.g. avoid duplicating `cardContentTitle` that matches the card title).
- **Null checks**: if a prop is typed `string`, don’t guard `!= null` unless refining from a wider external type at the boundary.

### Dead code

- **Grep for imports** before keeping a component file; if **nothing imports it**, delete it in the same refactor rather than leaving speculative helpers.
- Applies to alternate cell/group components that were superseded by the main grid implementation.

### Dashboard weight: hooks vs “override” props

- **Large blocks of derived state** in a page-level dashboard (`useMemo` chains that only feed one chart/table) are candidates for a **dedicated hook** or for **moving next to the consumer component**.
- **Prefer a single processed model** passed into presentational components over parallel **`data` + optional “override” props** that fork logic inside the child—especially when filters change the effective dataset. When a full view-model refactor is out of scope, **document** the follow-up in the PR.

### Comments and generated noise

- **Avoid JSDoc that restates the signature** or obvious behavior (“English description of something readable from the code”). Keep **`Step 7: Clean Up Comments`**—brief notes for _why_, not _what_.

## Validation Checklist

After refactoring, verify:

- [ ] No duplicate code remains between features
- [ ] Shared utilities are in appropriate `@/util/` locations (domain-specific subdirectories)
- [ ] Components are imported from shared locations
- [ ] Loading patterns match established features
- [ ] Styles use design system components or shared utilities
- [ ] All imports updated and working
- [ ] No linter errors introduced
- [ ] TypeScript types are correct
- [ ] Feature functionality unchanged (only structure improved)
- [ ] Tests still pass (if applicable)
- [ ] Unnecessary comments removed (migration notes, obvious statements)
- [ ] Only meaningful comments remain (explain WHY, not WHAT)
- [ ] Null checks simplified (`!= null` instead of `!== undefined && !== null`)
- [ ] Type-only imports use `import type` syntax
- [ ] Unnecessary type assertions removed
- [ ] Unnecessary `?? null` removed where not needed
- [ ] Unused props removed from component interfaces
- [ ] Explicit types added to callback parameters where needed
- [ ] Repeated JSX patterns extracted to reusable components
- [ ] DataGrid detail columns use `DetailPanelToggle` + a11y tab index where applicable
- [ ] No unnecessary `readonly object[]` for structured API rows; shared `*Like` types + one display helper
- [ ] Paged fetches loop until last page; no arbitrary sub-scale `limit` for full dashboards
- [ ] Sort/nullable comparators avoid magic domain caps (use explicit “last” ordering)
- [ ] Unused component files removed (verified via grep)
- [ ] Internal-only toggles not lifted without a documented cross-component need

## Common Patterns to Look For

### Aggregations & Data Transformations

- Grid/data table aggregations → Extract to `@/util/[domain]/gridAggregations.ts` or similar
- Data transformation functions → Extract to shared utilities
- Filter operators → Check if differences are intentional (different data structures) or can be unified

### Component Patterns

- Similar cell rendering logic → Consider if a wrapper component pattern could be reused
- Duplicate form components → Extract to shared form components
- Repeated UI patterns → Extract to shared components

### Hook Patterns

- Table/data grid hooks → Simpler versions are fine if feature requirements differ
- Filter hooks → Extract shared logic, keep feature-specific parts separate
- State management hooks → Consolidate when logic is identical

### Formatting Utilities

- Date formatting → Use `@/util/dates` (check for existing formatters)
- Number formatting → Use `@/util/numbers` (check for existing formatters)
- Domain-specific formatting → Extract to `@/util/[domain]/displayFormatter.ts` or similar

### Constants & Configuration

- Duplicate constants → Extract to shared constants file
- Configuration objects → Consolidate when structure is identical
- Type definitions → Extract to shared types when used across features

## When NOT to Refactor

- **Different data structures**: If features use fundamentally different data shapes, duplication may be intentional
- **Feature-specific requirements**: Simpler implementations are fine if they meet feature needs
- **Performance optimizations**: If duplication exists for performance reasons, document it
- **Temporary code**: If code is marked for removal or is experimental, don't refactor
- **Different contexts**: If code serves different purposes despite similar appearance, keep separate

## Example Refactoring Flow

1. **Identify**: Found duplicate aggregation functions in Feature A and Feature B
2. **Analyze**: Both implementations are identical (compare line-by-line)
3. **Check**: Verify no existing shared utility exists
4. **Extract**: Create `@/util/[domain]/[category].ts` with shared code
5. **Update**: Both features re-export from shared location
6. **Verify**: Imports work, no functionality changed, linter passes, tests pass
7. **Document**: Update this skill if pattern is reusable

## Best Practices

- **Always compare with similar features** - use semantic search to find related code
- **Prefer extracting to `@/util/`** over feature-specific locations when code is reusable
- **Keep feature-specific code** when differences are intentional and documented
- **Test functionality** remains unchanged after refactoring
- **Use TypeScript** to catch breaking changes during refactoring
- **Update imports systematically** - use IDE refactoring tools when possible
- **Document intentional differences** - if code looks similar but serves different purposes, add comments
- **Clean up comments** - remove migration notes, obvious statements, and comments that just restate code
- **Keep meaningful comments** - preserve comments that explain WHY, complex logic, or non-obvious behavior
- **Simplify null checks** - use `!= null` instead of verbose `!== undefined && !== null` checks
- **Use type-only imports** - prefer `import type` for type-only imports to improve build performance
- **Remove unnecessary assertions** - let TypeScript infer types when possible
- **Extract repeated patterns** - create reusable components for repeated JSX patterns
- **Use factory functions** - when logic is similar but handles different data structures, use factory functions with generics
- **Grid expand controls** - use `DetailPanelToggle` from `pattern-library`; match accessibility patterns from existing tables
- **Strong row/types** - replace `object` casts with exported narrow interfaces; one display helper per variant shape
- **Pagination** - align fetches with service paging rules; don’t cap dashboards at a single small page
- **Single view-model** - prefer one derived input to chart/table components over optional override props that double the branches
- **MUI + `sx`** - use **`sx`** for layout and styles on `Box`/similar; do not prioritize migrating **`sx`** to legacy **`Box`** style shorthand props (MUI deprecates that in favor of **`sx`**)
