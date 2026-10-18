# PR Review Command

You are a helpful assistant that reviews PR comments and provides feedback on the code.

## Context

You are reviewing all PR comments for a given PR in the **app-pro-ui** repository - the main Next.js application for the Moz SEO platform.

## Project Architecture

### Key Technologies

- Next.js 14 with App Router
- React 18 with TypeScript
- React Query (`@tanstack/react-query`) for server state
- React Hook Form for form state
- pattern-library for UI components (no custom CSS/inline styles)
- app-api for backend services

### File Naming Conventions

- Directories: kebab-case (`domain-overview`, `keyword-research`)
- Components: PascalCase (`BrandAuthority.tsx`, `DomainAuthority.tsx`)
- Hooks: camelCase (`useKeywordTopics.ts`, `useDomainOverview.ts`)
- Feature modules: `app/(default)/[feature]/_module/` structure

### Import Patterns

- Use barrel exports: `import { Button, Card } from "pattern-library"`
- Use path aliases: `import { Component } from "@/components"`
- Group imports: external libraries first, then internal modules

## Instructions

1. Ensure current branch is the branch of the PR.

2. Use gh cli to iterate over all unaddressed comments:

   ```bash
   gh pr view --comments --json comments
   gh api repos/{owner}/{repo}/pulls/{pr_number}/comments
   ```

3. Review each comment for the following criteria:

   - Is the comment addressing a valid issue in the updated code?
   - Is the comment within scope of the PR (not addressing unrelated existing code)?
   - Is the comment pointing out a bug or potential issue?
   - Would the proposed change follow project patterns:
     - Use pattern-library components only
     - Follow naming conventions (PascalCase components, camelCase hooks)
     - Use React Query for server state, React Hook Form for forms
     - No inline styles or custom CSS
     - Proper TypeScript types at boundaries
   - Would the proposed change improve readability and maintainability?
   - Would adding a test be worth the maintenance cost?

4. For valid comments, generate a task and add it to `review-result.md`.

5. For comments that do NOT meet the criteria, skip them (do not add to review-result.md).

6. When all comments are reviewed, output the `review-result.md` file.

7. **Wait for user approval** before implementing any changes.

8. After approval, implement changes following all project rules:

   - Use pattern-library components only
   - Follow established hook patterns in `hooks/` and `_module/hooks/`
   - Follow route patterns: `app/(default)/[feature]/page.tsx`
   - Use `_module/` directories for feature-specific code
   - No TODOs, console logs, or temporary shims in production code

9. Run verification checks:

   ```bash
   # Linting (must pass with 0 warnings)
   npm run lint

   # TypeScript check
   npm run ts-build

   # Unit tests
   npm run test
   ```

10. Fix any errors from the checks above.

11. **Do not commit planning files** (e.g., `implementation-plan.md`, `review-result.md`).

12. Create commit following conventional commits:

    - **Title MUST be 72 characters or less**
    - Format: `<type>(<scope>): <subject>`
    - Types: feat, fix, refactor, test, docs, style, perf, build, ci, revert
    - Example: `fix(ai-visibility): resolve hydration mismatch in summary`

13. Push the commit to the remote repository.

## Review Result Template

```markdown
# PR Review Results

## Comments to Address

### 1. [File Path] - Line X

**Comment**: [Original comment text]
**Action**: [What needs to be done]
**Priority**: [High/Medium/Low]

### 2. [File Path] - Line Y

...

## Comments Skipped (with reason)

### 1. [File Path] - Line X

**Comment**: [Original comment text]
**Reason**: [Why this was skipped - e.g., "Out of scope", "Addresses existing code not modified in PR"]
```
