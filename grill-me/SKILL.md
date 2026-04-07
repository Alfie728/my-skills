---
name: grill-me
description: Interview the user relentlessly about a plan or design until reaching shared understanding, resolving each branch of the decision tree. Use when the user wants to stress-test a plan, get grilled on their design, or explicitly mentions "grill me".
argument-hint: <plan-or-design>
metadata:
  short-description: Stress-test a plan through pointed questions
---

# Grill Me

Interview the user relentlessly about the plan until the decision tree is clear and the unresolved branches are closed.

## Operating Rules

- Ask one sharp question at a time when that keeps the thread moving.
- If a question can be answered by exploring the codebase, inspect the codebase instead of asking.
- Challenge fuzzy language. Convert "probably", "maybe", and "simple" into concrete behavior, constraints, and trade-offs.
- Keep drilling until scope, interfaces, risks, rollout, and validation are explicit.
- Summarize decisions periodically so the user can confirm or correct them.

## Areas To Cover

Work through the relevant branches:

1. Problem and motivation
2. Users and workflows
3. Constraints and non-goals
4. Interfaces, contracts, and data flow
5. Failure modes and edge cases
6. Migration and rollout
7. Testing and verification
8. Open risks or unknowns

## Completion

When the interview reaches a natural conclusion and the plan is solid, suggest the next step based on where the work is in the dev cycle:

- **Mon/Tue alignment phase, no docs yet** → suggest `$user-flow` to draft a non-technical user flow doc for business approval. This is the default starting point for new features.
- **User flow already approved, need architecture write-up** → suggest `$arch-brief` for a Slack-sized architecture overview.
- **Both alignment artifacts exist, need to break work into Linear issues** → suggest `$prd-to-issues`.
- **Scope is small and build-ready (no alignment needed)** → suggest `$tdd`.
- **You explicitly want a single combined PRD doc instead of the split flow** → suggest `$write-a-prd` (kept as a reference for the older single-doc workflow).
