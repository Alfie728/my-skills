---
name: tdd
description: Test-driven development with a red-green-refactor loop for deterministic code, and an eval-driven loop for agents and LLM features. Use when the user wants to build features or fix bugs using TDD, mentions red-green-refactor, wants integration tests, or is building agents that need eval coverage. Can take a Linear issue ID to pull acceptance criteria and file paths.
argument-hint: <linear-issue-id-optional>
metadata:
  short-description: Implement changes with disciplined TDD
---

# Test-Driven Development

IMPORTANT: Never auto-commit. Present the final output for review before any commit.

## Linear Integration

If the user provides a Linear issue ID, fetch the issue first with Linear MCP tools or the Linear CLI.

Use the issue to bootstrap planning:

- Acceptance criteria -> behaviors to test
- File paths -> likely work areas
- "What to do" -> implementation context
- Parent epic -> slice context

If no issue ID is provided, proceed from the user's request.

## Agent And LLM Work — Eval-Driven Loop

**If the work involves an agent, prompt, LLM call, tool-using model, or any non-deterministic generative behavior, switch to the eval-driven loop instead of the standard red-green-refactor.**

Triggers: agent build, prompt tuning, tool calling, RAG pipeline, classification with an LLM, structured output generation, system prompt design, multi-step reasoning chains, anything where the same input can produce different outputs.

### Why this is different

Traditional TDD assumes deterministic outputs. LLM systems do not give you that. A single pass/fail test on one input is meaningless — you need a dataset, a grading rubric, and a quality bar that you defend over time.

Agents go off rails. The eval set is the data that proves they don't.

### Eval-driven loop

Read [`evals.md`](evals.md) for the full protocol. The loop is:

1. **Define behaviors and failure modes.** Before writing any prompt, list:
   - What the agent must always do
   - What it must never do
   - The hardest realistic inputs
   - The dumbest realistic inputs
   - Adversarial inputs (jailbreaks, injection, ambiguous requests)

2. **Build a golden dataset.** 10-30 examples to start. Each row: input, expected behavior or expected output, grading criteria. This is your tracer bullet for non-deterministic systems.

3. **Write the eval harness before the prompt.** Run, expect failure, see the failure mode. This is your RED.

4. **Tune the prompt or agent.** Make the change. Re-run the full eval set, not just one example. This is your GREEN.

5. **Track scores between iterations.** Save each run. If a tuning step regresses earlier examples, that is a real failure — fix or accept the tradeoff explicitly.

6. **Lock in the dataset before merging.** New examples that surface during tuning go into the dataset. The dataset only grows; it does not get pruned to make scores look better.

7. **Set a quality bar.** What pass rate is shippable? Get the user's commitment up front, not after the run.

### Eval QA & Close

Before marking the Linear story Done for an agent build:

- Pass rate meets the agreed bar
- The dataset includes adversarial cases, not just happy path
- Regressions from prior runs are explicitly acknowledged
- The eval harness is committed alongside the agent code so future changes can be re-graded
- Production observability is wired up (logs of inputs, outputs, latency, cost) — you cannot tune what you cannot see

The standard TDD loop (below) still applies to surrounding deterministic code: data plumbing, validation, retries, response parsing, integration with the rest of the system. Use both loops together when the feature has both halves.

## Philosophy

Core principle: tests should verify behavior through public interfaces, not implementation details.

Good tests:

- Exercise real code paths through a public interface
- Describe what the system does
- Survive internal refactors

Bad tests:

- Mock internal collaborators
- Test private helpers
- Assert implementation details instead of behavior

Read the companion docs as needed:

- `tests.md` for examples of good and bad tests
- `mocking.md` for boundary mocking rules
- `deep-modules.md` for interface design heuristics
- `interface-design.md` for testable boundaries
- `refactoring.md` for cleanup targets after GREEN
- `evals.md` for the full eval-driven protocol for agent / LLM work

## Anti-Pattern: Horizontal Slices

Do not write all tests first and all implementation second.

Work in vertical slices:

1. RED: write one failing test
2. GREEN: write the minimal code to pass
3. Repeat for the next behavior
4. REFACTOR only after the relevant tests are green

## Workflow

### 1. Planning

Before coding:

- Confirm the interface change with the user when it is ambiguous or risky
- Confirm the highest-value behaviors to test
- Look for opportunities to deepen modules behind simple interfaces
- List the behaviors to test
- Get user approval on the plan when the design surface is material

### 2. Tracer Bullet

Start with one end-to-end test for one behavior.

### 3. Incremental Loop

For each remaining behavior:

- RED: write the next failing test
- GREEN: write the minimum code to pass

Rules:

- One test at a time
- No speculative features
- Keep tests focused on observable behavior

For non-trivial GREEN steps involving state, async work, persistence, auth, or complex transforms, do an extra verification pass before moving on.

### 4. Refactor

Once tests are green:

- Remove duplication
- Deepen shallow modules
- Improve names and interfaces
- Re-run tests after each refactor step

Never refactor while RED.

### 5. QA And Close

When working from a Linear story:

- Verify every acceptance criterion
- Run the relevant broader test suite for regressions
- For agent / LLM work, run the full eval set and confirm the pass rate meets the agreed bar
- If the criteria are satisfied, update the story status
- If this was the last story in an epic, mention that the epic may now be closable

Once the branch has an open PR and review comments come in, suggest `$pr-triage` to filter AI reviewer noise, address valid comments locally, and close out the review round before merge. This is the final step before the cycle closes.

## Per-Cycle Checklist

- Test describes behavior, not implementation
- Test uses a public interface
- Code is minimal for the current behavior
- No speculative behavior was added
