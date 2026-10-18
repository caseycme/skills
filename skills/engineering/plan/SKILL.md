---
name: task-refinement
description: >-
  Resolves what a story or task is building and how at a high level from
  ticket, repo, and design evidence, surfaces what the ticket missed as typed
  refinement findings, then asks only the remaining decisions that past sprint
  refinement comments show will otherwise surface later. Use when refining,
  grooming, or pre-refining a Jira story or task, writing acceptance criteria,
  splitting work, or asking what a ticket means before it enters a sprint.
---

# Task Refinement

Determine **what we are building** and **how at a high level**.

Use patterns from past refinement to predict what will become a question
later, resolve what can be resolved now, and surface only the remaining
decisions. Reduce questions. Do not automate asking more of them.

**Bias output toward the delta.** Give a compact restatement so findings have
context, then spend the words on what the ticket missed, contradicted, or left
implied — not a second copy of the ticket.

This is not a feasibility check, design critique, data-architecture review,
rollout plan, or readiness gate. Use `feasibility` if they need a
repo-evidence ready/not-ready call. Do not implement production code.

## When invoked

1. Gather context: ticket key or paste, linked design, parent/epic, and
   anything already in the chat.
2. Inspect what exists. If a ticket key is given and Jira tools are available,
   fetch summary, description, and comments. Skim the repo and design only
   enough to answer the six lenses below. Do not invent ticket or API content.
3. For each lens, write down **Fact**, **Assumption**, and **Unknown**
   privately. Do not dump this table at the user — use it to populate
   **Refinement findings** and **Resolved from evidence**.
4. Write **Current understanding** (2–4 sentences), not a full brief.
5. Emit typed **Refinement findings**, **Resolved from evidence**, and
   **Suggested ticket changes**.
6. Ask only leftover **Questions requiring a decision**. Highest-risk first.
   At most 3–6. Prefer concrete choices.
7. After answers (or if nothing was unanswered): emit the consolidated
   **Refinement brief**. Repeat only if new unknowns block estimate.
8. Stop when what/how are clear enough to estimate, or name the leftover
   decisions that still block that.

## Rules

- Ask only unanswered questions. Skip anything already in the ticket, comments,
  design, or repo, and cite where.
- Distinguish **Fact** / **Assumption** / **Unknown** internally; express
  assumptions and gaps as typed findings, not as a generic concern list.
- Never invent endpoints, flags, teams, or designs.
- No file-by-file implementation plans.
- One concern per question.
- If the title does not match the body, resolve that before anything else.
- Do not reprint ticket AC in **Current understanding**. Cite the ticket.
- If nothing was missed, say so in one line under **Refinement findings** —
  do not invent gaps.

## Lenses

Internal gather checklist. Each lens owns one kind of uncertainty. Do not
re-ask the same edge case under a second lens.

### 1. Outcome and scope

_What is this work, and what is not this work?_

- User and outcome: what they can do after that they cannot do today.
- Story vs task vs spike. System-voiced titles (“as the app…”) are usually
  tasks.
- Title vs body: one source of truth.
- **Bug, change, or new capability?** Does current behavior violate existing
  AC/design, or is the request a new product decision? Matching the original
  intent is not automatically a bug.
- In vs out. Nearby UI (chip, filter, column, second surface) is in, out, or
  a follow-up — implied work is how tickets grow.
- Overlaps, duplicates, and whether this is **one coherent task**.
- If this work **exposes an existing platform or product problem**, does that
  block this slice or become a follow-up?

### 2. Acceptance criteria

_What does done look like, observably?_

AC define done. They are not the place to decompose implementation.

- Observable checks (checklist or Given/When/Then), not “should work.”
- Specific enough to demo or test. User-facing, not file names.
- Smallest **valuable** slice (still a complete outcome, not a layer of the
  stack).
- If AC already live only in Slack or comments, pull them into
  **Suggested ticket changes** and tell them to paste into the ticket.

### 3. Design and specification

_What is specified to look at or reuse?_

- Figma or screenshot with a node, not just a file. Missing copy/assets, and
  who owns them.
- Design-system names, not marketing names.
- Design vs AC: do they describe the same thing?
- Reuse, lift-and-shift, or a new pattern. Search before assuming new.

### 4. Data and ownership

_Where does the truth come from, and is it available?_

- Dummy vs real this sprint. If dummy, the hookup is a follow-up, not a
  guess in this ticket.
- Owning service / API / action.
- Available in the env we will test, or blocked on someone else.
- Payload and state assumptions we are treating as true (ids in, what the
  server already knows). Sample payload or a test account if we claim real
  data.
- **Scale and limits** only when they change the solution: expected volume,
  account size, latency, or shared caps across pages.

Product behavior for empty, partial, or failed data belongs in lens 5, not
here.

### 5. Behavior and states

_What should the product do in each situation?_

- Happy path.
- Loading, empty, error, partial / in-progress data.
- Access, permission, quota, and validation (including field vs submit, and
  spend that succeeds when data does not).
- Defaults and what persists (filters, dates, granularity, toggles).
- Change over time: new enums, engines, or icons without a deploy — does
  the page survive?
- Any visual “meaning” (bands, targets) needs a definition, not just a color.

Off-flag and lower-tier **UX** live here. Flag as a ship vehicle lives in
lens 6.

### 6. Dependencies and delivery

_What must be true to ship this slice, not what the product does?_

- Blockers (team, creds, design date, another PR).
- Repo/team dependencies and sequencing (land together or not).
- Flag, beta vs GA, environment — as delivery, not as product states.
- **Verification:** do we have the account, data, state, and environment
  needed to exercise the AC, or is special setup or mocking required? Who
  signs off, and on what preview or env.
- Analytics in this ticket or a follow-up.

## Refinement findings

Typed findings replace a generic **Concerns** list. Omit empty types.

- **Missing** — not in ticket, design, or AC (error state, implied column).
- **Conflict** — ticket vs repo, design, or AC (e.g. “community unchanged”
  but repo leaves research nav unlocked).
- **Assumption** — treating as true, not established in evidence.
- **Scope risk** — nearby surface that may grow the ticket if not called out.
- **Dependency** — named blocker or sequencing, not a product-behavior
  question.

## Output

Two modes in one interaction. Default to **during refinement** first.

### During refinement

Lead with the delta once evidence is gathered.

```markdown
# Refinement: [ticket or working title]

## Current understanding

[2–4 sentences: who/outcome, high-level how, in vs out. Cite the ticket.
Do not reprint AC.]

## Refinement findings

- **Missing:** …
- **Conflict:** …
- **Assumption:** …
- **Scope risk:** …
- **Dependency:** …

[If nothing was missed: "No gaps found against ticket, repo, and design."]

## Resolved from evidence

- [Fact we will not re-ask, with cite]

## Questions requiring a decision

- [At most 3–6]

## Suggested ticket changes

- [AC to add, out-of-scope to mark, link to add]
```

Clear enough to estimate when remaining unanswered items are named people or
named artifacts — not an open “what are we building?”

### After decisions

Once answers land (or nothing was unanswered). Consolidate the **agreed**
feature — not a second delta pass.

```markdown
# Refinement brief: [ticket or working title]

## What we are building

[User, outcome, bug vs change vs new, in-scope slice.]

## How (high level)

- Surface:
- Reuse or new:
- Data:
- Delivery:
- Out of scope:

## Acceptance criteria (draft)

- [ ] …

## Follow-ups

- [Work that is not this ticket]
```

Named blockers belong under **How (delivery)** or **Follow-ups**, not a
separate concerns section.

## Evaluation

To test the skill against historical Jira data: take a pre-refinement ticket,
run the skill, and compare **Refinement findings** to subsequent Jira comments
and description edits. Judge what it caught that the original task missed —
not how well it paraphrases the ticket.
