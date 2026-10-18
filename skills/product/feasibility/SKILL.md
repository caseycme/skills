---
name: feasibility
description: Helps Product, UX, and Engineering assess whether a proposed feature is ready for prototyping or implementation.
---

# Feature Feasibility

Product/UX skill: help Product, UX, and Engineering evaluate whether a proposed feature is ready for prototyping or implementation.

This skill determines **what should be built and whether it is ready**—not how to implement production code (see engineering skills for that).

## When invoked

1. Summarize the feature request.
2. Search the repository for related components, APIs, services, Storybook
   examples, and existing patterns.
3. Identify opportunities to reuse existing code instead of building something
   new.
4. Highlight technical constraints, dependencies, and implementation
   considerations.
5. List assumptions and unanswered questions.
6. Identify UX, product, and technical risks.
7. Recommend one of:

- Ready to implement
- Prototype first
- Requires additional research
- Not currently feasible

8. End with the next recommended actions.

## Guidelines

- Prefer evidence from the repository over assumptions.
- Clearly distinguish facts from speculation.
- Never invent APIs or existing functionality.
- Encourage convention over custom implementation.
- Keep the output concise, actionable, and suitable for feature planning discussions.

## Research workflow

Copy and track:

```
Feasibility Progress:
- [ ] Summarize the request
- [ ] Search related UI (components, hooks, routes, Storybook)
- [ ] Search related APIs / services / flags / analytics
- [ ] Map reuse opportunities vs net-new work
- [ ] Note constraints, dependencies, considerations
- [ ] List assumptions and open questions
- [ ] Assess UX / product / technical risks
- [ ] Recommend readiness + next actions
```

### Search focus (adapt to the active repo)

Prioritize evidence in this order:

1. **Existing UI** — feature modules, shared components, hooks, screens
2. **Design system / Storybook** — established patterns before custom UI
3. **APIs & data** — client wrappers, actions, types, mocks used by similar features
4. **Flags & analytics** — feature-flag and tracking conventions
5. **Conventions** — architecture rules, naming, routing, state patterns

Cite concrete paths for every reuse claim. If something is not found, say so explicitly—do not invent it.

### Facts vs speculation

Label each non-obvious claim:

- **Fact** — verified in the repository (cite path)
- **Assumption** — inferred; needs confirmation
- **Unknown** — not found / not specified

## Output template

Use this structure:

```markdown
# Feature Feasibility: [Feature name]

## Summary

[2–4 sentences: what is proposed, for whom, and the intended outcome]

## Evidence from the repository

### Related existing work

- [path] — [what it does / why relevant]

### Reuse opportunities

- [what to reuse] — [where] — [how it reduces scope]

### Gaps (net-new)

- [what does not exist today]

## Constraints & dependencies

- [technical constraints, upstream services, flags, auth, data availability]

## Implementation considerations

- [high-level notes only—no full design; prefer conventions]

## Assumptions

- [assumption]

## Open questions

- [question that blocks a clearer recommendation]

## Risks

| Area      | Risk | Severity     | Notes |
| --------- | ---- | ------------ | ----- |
| UX        |      | Low/Med/High |       |
| Product   |      | Low/Med/High |       |
| Technical |      | Low/Med/High |       |

## Recommendation

**[Ready to implement | Prototype first | Requires additional research | Not currently feasible]**

[1–3 sentences justifying the choice from evidence above]

## Next actions

1. [concrete next step]
2. [concrete next step]
3. [concrete next step]
```

## Recommendation criteria

| Recommendation               | Use when                                                                                    |
| ---------------------------- | ------------------------------------------------------------------------------------------- |
| Ready to implement           | Clear requirements; reusable patterns/APIs exist; residual risks are manageable             |
| Prototype first              | Interaction/UX uncertainty is the main risk; core building blocks likely exist              |
| Requires additional research | Missing APIs, unclear data, unresolved product questions, or thin repo evidence             |
| Not currently feasible       | Blocked by platform limits, missing dependencies, or conflicts with conventions/constraints |

## Anti-patterns

- Do not draft full implementation plans or production code in this skill.
- Do not invent endpoints, components, or flags.
- Do not treat absence of evidence as proof of impossibility—mark as **Unknown** and recommend research.
- Do not expand scope beyond the proposed feature.
