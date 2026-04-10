---
name: arch-brief
description: Draft a scannable architecture brief for Slack after the user flow is approved. Use when the user wants an architectural write-up, needs to align the team on an approach, or is deciding whether a huddle is needed before building.
argument-hint: <feature-name>
metadata:
  short-description: Produce a Slack-sized architecture brief
---

# Architecture Brief

IMPORTANT: Never auto-commit. Never auto-post to Slack without showing the brief and getting confirmation first.

## When To Use

After the user flow doc is approved by business. Before the engineering spec (`$write-a-prd`) and issue breakdown. This is a short, scannable write-up posted to Slack so the whole team has visibility on the approach before any detailed spec work begins.

Must fit in a Slack message. Granular detail belongs in `$write-a-prd` and sub-issues, not in the brief.

## Process

1. Ask for or pull the approved user flow doc from context. If the user flow hasn't been done yet, suggest `$user-flow` first.

2. Explore the codebase to ground the architecture:
   - Existing models and workflows relevant to the feature
   - Patterns worth following
   - Integration points and system interactions
   - Reusable modules or scaffolding

   While exploring, watch for architectural friction: shallow modules with leaky interfaces, tight coupling across boundaries, or areas where the new feature would have to work around existing messiness. If significant friction is found, pause and suggest `$improve-codebase-architecture` before designing approaches — the right approach for the feature may depend on cleaning up the foundation first.

3. Design at least 2–3 materially different approaches. For each one:
   - Describe the approach in 1–2 sentences
   - List concrete pros (performance, simplicity, extensibility, reuse)
   - List concrete cons (complexity, coupling, migration cost, lock-in)
   - Note any open questions or unknowns that would change the recommendation

   Do not pick an approach yet. Present all options and ask the user which direction to commit to, or give a recommendation and ask for confirmation.

4. Agree on the approach before drafting anything. If the user asks for a hybrid, propose it explicitly.

5. Draft the brief using the template below. The chosen approach becomes the "Approach" section; the rejected alternatives become the "Tradeoffs" section. Keep it brutally short — the whole point is scannability.

6. Present for review. Iterate until the brief is short, clear, and decisive.

7. Decide whether a huddle is warranted:
   - Complex extensibility tradeoffs → huddle
   - Touches multiple owners or surface areas → huddle
   - Non-obvious system interactions → huddle
   - Straightforward approach with clear patterns → no huddle

   If a huddle is needed, suggest which people to pull in.

8. After approval, format the brief for Slack posting.

9. Wait for the user's explicit "post it" confirmation before sending anything.

10. Suggest `$write-a-prd` as the next step once team alignment is confirmed — the full engineering spec lives there.

## Template

<arch-brief-template>

# <Feature Name> — Architecture

## Approach

2 to 3 sentences in plain English describing how we'll build this. No code, no full interfaces. Enough for a teammate to nod or push back.

## Key Changes

Bullet list of the main modules, services, or layers being added or modified. One line each.

- **<Module or layer>** — <what it does>
- **<Module or layer>** — <what it does>

## Tradeoffs

What we are choosing and why. What we considered and rejected.

- **Chose:** <approach>. **Instead of:** <alternative>. **Because:** <reason>.
- **Chose:** <approach>. **Instead of:** <alternative>. **Because:** <reason>.

## Extensibility

One or two lines on what this enables for future work and what it locks us out of.

## Risks And Open Questions

Things that could derail us or need more thought before we commit.

- <Risk or question>
- <Risk or question>

## Huddle Recommendation

- **Needed:** Yes / No
- **If yes, who:** <names or roles>
- **Why:** <reason>

</arch-brief-template>

## Quality Checks

- Fits comfortably in a Slack message without scroll fatigue
- A teammate can scan the whole thing in under 2 minutes
- No file paths, no code snippets, no schema, no function names
- Tradeoffs are explicit and include the alternative that was rejected
- Huddle recommendation is decisive, not "maybe we should talk"
- Extensibility section is honest about what gets locked out

## Next Step

After team alignment (async or via huddle), suggest `$write-a-prd` to write the full engineering spec — schemas, API contracts, module interfaces, and testing decisions. Once the dev PRD is approved and submitted to Linear, suggest `$prd-to-issues` to break the work into epics and stories.
