# Review Pull Request

Analyze the attached PR diff, CI checks, and project rules. Generate a review with all findings ordered by severity.

## Project Context

**Repository**: app-pro-ui - Main Next.js application for the Moz SEO platform

### Key Technologies

- Next.js 14 with App Router
- React 18 with TypeScript
- React Query (`@tanstack/react-query`) for server state
- React Hook Form for form state
- pattern-library for UI components
- app-api for backend services

### Project Rules Location

- `.cursor/rules/architecture.mdc` - Architecture and naming conventions
- `.cursor/rules/global.mdc` - Global development rules
- `.cursor/rules/verification.mdc` - Verification requirements
- `.cursor/rules/repo-context.mdc` - Repository relationships
- `.cursorrules` - Commit message format (72 char limit)

## Steps

1. Read PR diff (attached or via `gh pr diff`) and CI check status (`gh pr checks`)
2. Read `.cursor/rules/*.mdc` for project-specific rules
3. If Jira ticket provided, read requirements
4. Review using the checklist below; identify all issues with rule citations
5. Generate `review-result.md` with findings ordered from most critical to least critical
6. **AUTHORIZATION GATE**: Present to user for approval before posting
7. Post overall review summary as PR comment, then post each finding as inline comment on the relevant line (use `suggestion` blocks for fixes)
8. Note in comments: "AI Review by [MODEL_NAME]" (replace with your model identifier)

## Review Checklist

### Architecture & Patterns

- [ ] Uses pattern-library components only (no custom CSS/inline styles)
- [ ] Follows file naming conventions:
  - Directories: kebab-case (`domain-overview`, `keyword-research`)
  - Components: PascalCase (`BrandAuthority.tsx`)
  - Hooks: camelCase (`useKeywordTopics.ts`)
- [ ] Uses `_module/` directories for feature-specific components
- [ ] Follows route patterns: `app/(default)/[feature]/page.tsx`
- [ ] Uses barrel exports and path aliases for imports

### State Management

- [ ] Uses React Query for server state
- [ ] Uses React Hook Form for form state
- [ ] Uses `useState`/`useReducer` for local component state
- [ ] Uses Next.js `useSearchParams`/`useRouter` for URL state
- [ ] No global state outside approved patterns

### Type Safety

- [ ] Avoids weak casts (`as any`, `as unknown as T`)
- [ ] Validates DTOs/schemas at boundaries
- [ ] Proper TypeScript interfaces for props and data

### Code Quality

- [ ] No TODOs, console logs, or temporary shims in production code
- [ ] No unused imports or exports
- [ ] Single responsibility functions
- [ ] Clean, readable code structure

### Testing

- [ ] Tests follow project patterns (Jest, Testing Library)
- [ ] One facet per test
- [ ] Worth the maintenance cost

### Security

- [ ] No hardcoded secrets or credentials
- [ ] Proper input validation
- [ ] XSS prevention (using `xss` library when needed)

## Review Focus

- **Design System**: pattern-library components only, no inline styles
- **Naming Conventions**: PascalCase components, camelCase hooks, kebab-case directories
- **State Management**: React Query for server state, React Hook Form for forms
- **Route Structure**: `app/(default)/[feature]/_module/` pattern
- **Import Organization**: External libraries first, then internal modules
- **Type Safety**: Proper TypeScript at all boundaries
- **CI failures**: `npm run lint`, `npm run ts-build`, `npm run test`

## Output Format (review-result.md)

```markdown
## Summary

[1-2 sentences]

## CI Status

[Pass/Fail summary with links to failing checks]

## Findings (Critical → Minor)

### 1. [Issue Name] — CRITICAL | HIGH | MEDIUM | LOW

**File:** `path/to/file.ts:L42`
**Rule:** [.cursor/rules/architecture.mdc - Section X]
**Issue:** [Description]
**Fix:**
\`\`\`suggestion
// corrected code
\`\`\`

## Verdict

[Approved | Approved with changes | Request changes]
```

## Posting Rules

- Every comment MUST address an issue found in review-result.md
- Every comment MUST be descriptive and provide a solution
- DO NOT post comments that are not in review-result.md
- Reference specific rules from `.cursor/rules/*.mdc` when citing violations

## Inline Comment Tips (gh CLI)

- Use `gh api repos/{owner}/{repo}/pulls/{number}/comments` for inline comments
- Required fields: `body`, `commit_id`, `path`, `line` (or `start_line`/`line` for multi-line)
- Get `commit_id` from `gh pr view {number} --json headRefOid`

## Common Rule Violations

| Violation                | Rule Reference                      | Fix                                    |
| ------------------------ | ----------------------------------- | -------------------------------------- |
| Custom CSS/inline styles | architecture.mdc - Design System    | Use pattern-library components         |
| Wrong file naming        | architecture.mdc - Naming & Paths   | Follow PascalCase/camelCase/kebab-case |
| Direct API calls         | architecture.mdc - State Management | Use React Query                        |
| Global state             | architecture.mdc - State Management | Use approved hooks/patterns            |
| Console logs             | global.mdc - Production-Only Code   | Remove before commit                   |
| Commit title > 72 chars  | .cursorrules                        | Shorten title, use body for details    |
