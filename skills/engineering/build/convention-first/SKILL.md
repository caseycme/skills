---
name: convention-first
description: Guides development by prioritizing existing patterns and conventions over new solutions. Ensures production-ready code by rejecting TODOs, temporary fixes, and invented patterns. Use when implementing features, refactoring code, or when the user asks to follow established patterns or write production-ready code.
---

# Convention-First Development

This skill enforces a pattern-driven development approach that prioritizes reusing existing conventions and delivering production-ready code.

## Core Principles

### 1. Conventions Over Working Code

**Working code that breaks conventions must be rejected.**

Even if code functions correctly, it's unacceptable if it violates established patterns. Consistency and maintainability trump short-term solutions.

### 2. Search Before Inventing

**Never invent a new pattern when precedent exists.**

Before implementing any solution:

1. Search the codebase for similar implementations
2. Identify the established pattern
3. Follow that pattern exactly
4. Cite the file/component you're using as reference

**Example workflow:**

```markdown
Task: Add a new form to the application

Before implementing:

- Search: `useForm`, `FormProvider`, form validation
- Find: `components/UserSettingsForm.tsx` uses React Hook Form
- Cite: "Following pattern from `components/UserSettingsForm.tsx`"
- Implement: Use the same libraries, structure, and validation approach
```

### 3. Ask When Patterns Are Unclear

**If uncertain, ask before coding. Never bypass rules to "get it working."**

Stop and ask when:

- Multiple patterns exist for the same task
- No clear precedent exists
- Existing patterns seem outdated or conflicting
- You're unsure about naming conventions, file structure, or architecture

**Never:**

- Guess at API signatures
- Invent temporary solutions
- Mix incompatible patterns
- Assume a pattern without verification

### 4. Plan Before Building

**Show a plan before generating code.**

For any non-trivial change:

1. Outline the approach
2. Identify patterns to follow
3. List files that will change
4. Highlight any uncertainties
5. Get confirmation before proceeding

This prevents wasted effort on incorrect approaches.

### 5. Production-Only Code

**No TODOs, mocks (outside unit tests), console logs, inline CSS, or temporary shims.**

Every line of code must be production-ready:

❌ **Rejected:**

```typescript
// TODO: Add error handling later
const data = await fetchData();

// Temporary fix until we refactor
if (env === 'dev') console.log(data);

// Quick styling for now
<div style={{ marginTop: '10px' }}>
```

✅ **Accepted:**

```typescript
const data = await fetchData().catch(handleError);

<Box mt={2}>
```

### 6. Clean Code After Each Change

After every implementation or edit:

**Required cleanup checklist:**

- [ ] Remove unused imports
- [ ] Remove unused exports
- [ ] Remove unused variables/functions
- [ ] Confirm type safety (no `any`, no type errors)
- [ ] Remove stray comments or debugging code
- [ ] Verify no linter errors introduced

**This applies to every prompt, including corrections and refinements.**

## Pattern Discovery Workflow

When implementing a new feature, follow this systematic approach:

### Step 1: Identify the Domain

What type of problem are you solving?

- UI component → Search for similar components
- Data fetching → Search for similar queries/hooks
- Form handling → Search for form implementations
- State management → Search for state patterns
- Routing → Search for route structures

### Step 2: Search for Precedent

Use targeted searches to find existing patterns:

```bash
# Component patterns
grep -r "export.*Component" components/

# Hook patterns
grep -r "export.*use" hooks/

# Query patterns
grep -r "useQuery\|useMutation" .

# Form patterns
grep -r "useForm\|FormProvider" .
```

### Step 3: Analyze the Pattern

When you find a relevant example:

1. Note the file structure
2. Identify dependencies used
3. Document naming conventions
4. Understand the pattern's constraints
5. Check if it's consistently used

### Step 4: Cite and Implement

When implementing, explicitly cite your source:

```markdown
Following pattern from `components/KeywordListForm.tsx`:

- Using React Hook Form with `useFieldArray`
- Validation with Zod schema
- Submit handler pattern
- Error display approach
```

Then implement using the exact same libraries, structure, and conventions.

### Step 5: Verify Consistency

After implementation:

- Does your code look like it came from the same codebase?
- Are naming conventions identical to the cited example?
- Are imports structured the same way?
- Would another developer recognize the pattern?

## Handling Edge Cases

### When No Pattern Exists

If you genuinely cannot find a precedent:

1. **Stop and document the search:**

   ```markdown
   Searched for: [list searches performed]
   Found: [what you found, if anything]
   Question: Should I use [option A] or [option B]?
   ```

2. **Propose options with trade-offs:**

   ```markdown
   Option A: Use library X

   - Pro: Well-documented, widely used
   - Con: Adds new dependency

   Option B: Build custom solution

   - Pro: No dependencies
   - Con: More code to maintain
   ```

3. **Wait for guidance before proceeding**

### When Patterns Conflict

If you find inconsistent patterns:

1. **Document the conflict:**

   ```markdown
   Found two patterns:

   - Pattern A in `componentA.tsx` (newer, 2 uses)
   - Pattern B in `componentB.tsx` (older, 5 uses)
   ```

2. **Ask which to follow:**
   ```markdown
   Which pattern should I use? Should I standardize the codebase?
   ```

### When Patterns Seem Wrong

If an established pattern appears problematic:

1. **Follow it anyway for consistency**
2. **Document the concern:**
   ```markdown
   Following existing pattern from `file.tsx`, though this approach
   may have [specific issue]. Consider addressing separately.
   ```
3. **Don't mix old and new patterns**

## Anti-Patterns to Avoid

### ❌ The "Just Get It Working" Approach

```typescript
// This works but violates conventions
const getData = async () => {
  // TODO: Use proper error handling
  try {
    const response = await fetch("/api/data");
    return response.json();
  } catch (e) {
    console.log("Error:", e); // Quick debug
    return null;
  }
};
```

**Why rejected:** TODOs, console logs, and inconsistent error handling.

### ❌ The "I'll Clean It Up Later" Approach

```typescript
import { useCallback, useMemo, useState } from "react";

import { Button, Card, TextField } from "ui-library";

// unused imports
// import { OldComponent } from './old'; // commented out code

export const MyComponent = () => {
  // Implementation
};

// export const UnusedHelper = () => {}; // unused export
```

**Why rejected:** Unused imports, commented code, unused exports.

### ❌ The "Creative Solution" Approach

```typescript
// Existing pattern uses: axios + React Query
// But I prefer: fetch + SWR
import useSWR from "swr";

export const useCustomData = () => {
  return useSWR("/api/data", fetch);
};
```

**Why rejected:** Invents new pattern when established pattern exists.

### ✅ The Convention-First Approach

```typescript
// Found pattern in: hooks/useUserProfile.ts
// Following exactly: axios + React Query + error handling
import { useQuery } from "@tanstack/react-query";

import { apiClient } from "@/lib/api";
import { handleApiError } from "@/lib/errors";

export const useCustomData = () => {
  return useQuery({
    queryKey: ["customData"],
    queryFn: async () => {
      const response = await apiClient.get("/data");
      return response.data;
    },
    onError: handleApiError,
  });
};
```

**Why accepted:** Cites precedent, follows established pattern, production-ready.

## Quick Reference

### Before Every Implementation

1. ✅ Search for existing patterns
2. ✅ Cite the file/component you're following
3. ✅ Ask if patterns are unclear or conflicting
4. ✅ Show a plan for non-trivial changes

### During Implementation

5. ✅ Follow the cited pattern exactly
6. ✅ Use the same libraries and conventions
7. ✅ Write production-ready code (no TODOs, temporary fixes)

### After Implementation

8. ✅ Remove unused imports/exports/variables
9. ✅ Verify type safety
10. ✅ Confirm no linter errors
11. ✅ Verify consistency with cited pattern

### Red Flags

🚩 You're inventing a new pattern
🚩 You're writing a TODO or temporary fix
🚩 You're mixing different patterns
🚩 You're not sure which pattern to follow
🚩 You're adding console logs or debugging code
🚩 You're using inline styles
🚩 You're leaving commented code

**When you see a red flag: STOP and ask.**

## Summary

Convention-first development means:

- **Consistency over creativity** - Follow established patterns
- **Quality over speed** - Write production-ready code
- **Clarity over assumption** - Ask when uncertain
- **Cleanliness over clutter** - No unused code or TODOs

This approach ensures:

- Maintainable codebases that scale
- Predictable patterns that are easy to understand
- High-quality code that's ready to ship
- Consistent conventions across the entire project
