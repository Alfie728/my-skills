---
name: write-a-prd
description: Create a PRD through user interview, codebase exploration, and module design, then optionally submit it to Linear after user review. Use when the user wants to write a PRD, create a product requirements document, or plan a new feature.
argument-hint: <feature-or-problem>
metadata:
  short-description: Draft a PRD from discovery through design
---

# Write A PRD

IMPORTANT: Never auto-commit. Never auto-submit to Linear without showing the PRD to the user first and getting confirmation.

Use this skill when the user wants a serious feature spec, not just a quick implementation sketch.

## Process

1. Ask the user for a detailed description of the problem, the desired outcome, and any existing solution ideas.
2. Explore the repo to verify assumptions and understand the current codebase state.
3. Interview the user until the major branches of the design are resolved.
4. Sketch the major modules that will need to be built or changed. Look for deep modules with simple interfaces and testable boundaries.
5. Confirm the module boundaries and ask which modules deserve focused testing.
6. Ask where the PRD should live:
   - New Linear project
   - Existing Linear project
   - Existing Linear issue
   - New issue under an existing parent
7. If Linear is in scope, determine the correct team, project, and parent item.
8. Draft the PRD using the template below.
9. Present the PRD for review. After approval, use Linear MCP tools or the Linear CLI to create or update the destination item.
10. After submission, suggest `$prd-to-issues`.

## PRD Structure

The PRD is for two audiences:

- Stakeholders read top-down and stop at the `---` divider.
- Engineers continue past the divider into the implementation sections.

Keep the sections above the divider free of schema names, enum values, database columns, and code-level detail.

<prd-template>

## Problem Statement

The problem from the user's perspective.

## Context: How We Get Here

A mermaid sequence diagram showing the journey that leads into this feature.

Follow with 2-3 sentences of plain-language narrative using concrete examples.

## Solution

The solution from the user's perspective.

### Layout

If the feature has a UI, describe what the user sees and how information is organized.

### Lifecycle

If the feature has stateful objects, include a mermaid flowchart of user-visible states and transitions.

### Interaction Flow

A mermaid sequence diagram of the primary happy path using realistic actions or dialogue.

### Statuses / States

If applicable, a simple table of user-visible states, meaning, and visual treatment.

## User Stories

A long, numbered list of user stories in this format:

1. As an <actor>, I want a <feature>, so that <benefit>

---

## Implementation Decisions

The sections below are for engineering.

Include:

- Modules to build or modify, each as a named section
- Interface changes
- Architectural decisions
- Schema or API changes
- Technical clarifications

Do not include file paths or code snippets.

## Testing Decisions

Include:

- What makes a good test for this feature
- Which modules should be tested
- Relevant test prior art in the codebase
- Which modules deserve extra verification during implementation

## Out of Scope

A bulleted list of explicit non-goals with brief reasons.

## Further Notes

Reusable patterns, existing components, and domain context engineers will need.

</prd-template>
