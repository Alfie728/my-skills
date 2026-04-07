---
name: prd-to-issues
description: Break a PRD into Linear issues and sub-issues using a parent -> epic -> story hierarchy optimized for parallel agent work. Use when the user wants to convert a PRD to issues, create implementation tickets, or break down a PRD into work items.
argument-hint: <linear-project-or-issue>
metadata:
  short-description: Turn a PRD into executable Linear work
---

# PRD To Issues

IMPORTANT: Never auto-commit. Never create or modify Linear issues until the proposed breakdown has been shown to the user and approved.

## Goal

Break a PRD into a parent -> epic -> story hierarchy that is executable, dependency-aware, and optimized for parallel work.

## Process

### 1. Locate the PRD

Determine where the PRD lives:

- Linear project description
- Existing issue description
- Issue comment on a parent issue

If it was created earlier in the same session, confirm the destination with the user. Otherwise fetch it with Linear MCP tools or the Linear CLI.

### 2. Understand Current Linear State

Inspect:

- Team, project, milestones, statuses, labels
- Existing issues that already cover part of the scope
- Parent/sub-issue structure
- Existing dependencies

Do not duplicate issues that already exist.

### 3. Explore the Codebase

If needed, inspect the repo to identify:

- Existing models and workflows
- Reusable code and registration patterns
- Likely file paths and boundaries for each slice

### 4. Draft the Hierarchy

Use this structure:

- Parent issue: overall feature and integration point
- Epics: one workspace each, coherent and independently verifiable
- Stories: small, specific tasks, ideally 1 point each

Each story should include:

- Exact file path(s) to modify or create
- What to change
- Existing patterns to follow
- Verification steps

Prefer AFK slices over HITL slices where possible.

### 5. Design the Dependency Graph

Build a clear DAG:

- Foundations first
- Core logic next
- API/UI last

Draft workspace sequencing in the parent issue description so the user knows exactly what can run in parallel and what is blocked.

### 6. Present the Breakdown

Show a numbered list of proposed epics. For each epic include:

- Title
- HITL or AFK
- Workspace number
- Blocked by
- Stories
- User stories covered
- Estimate

Ask the user to confirm:

- Granularity
- Dependencies
- Merge/split opportunities
- HITL vs AFK labeling
- Milestone or sprint placement

Iterate until approved.

### 7. Create the Linear Issues

After approval, create or update the parent, epics, and stories using Linear MCP tools or the Linear CLI.

Set:

- Title
- Description
- Priority
- Estimate
- Parent
- Dependencies at the epic level
- Project
- Milestone if relevant

### 8. Update the Parent

After creation:

- Update the parent issue with real issue IDs in workspace sequencing
- Add stub interfaces or scaffolds to create up front
- Add end-to-end verification criteria

Do not modify or close the original PRD content.

## Quality Checks

- Parent issue includes workspace sequencing, stubs, and E2E criteria
- Every epic includes workspace number, parallelism notes, and ordered stories
- Every story references concrete file paths
- Dependencies are at the epic level, not between stories across epics
- The breakdown is specific enough to implement without rereading the full PRD

## Next Step

Once the issues exist, suggest `$tdd <issue-id>` for implementation.
