---
name: improve-codebase-architecture
description: Explore a codebase to find opportunities for architectural improvement, focusing on making the codebase more testable by deepening shallow modules. Use when the user wants to improve architecture, find refactoring opportunities, consolidate tightly-coupled modules, or make a codebase more AI-navigable.
argument-hint: <scope-or-subsystem>
metadata:
  short-description: Find deep-module refactors in a codebase
---

# Improve Codebase Architecture

Explore the codebase the way an implementation agent would, surface architectural friction, and propose refactors that deepen modules and improve testability.

## Process

### 1. Explore the Codebase

Inspect the codebase organically and note where understanding breaks down:

- A concept is spread across too many small files
- The interface is nearly as complex as the implementation
- Pure helpers were extracted for testability, but bugs still live in the seams
- Tight coupling creates integration risk
- Tests are hard to write or obviously low-value

Treat that friction as signal.

### 2. Present Candidates

Show a numbered list of deepening opportunities. For each candidate include:

- Cluster: which modules or concepts are involved
- Why they are coupled
- Dependency category from `REFERENCE.md`
- Test impact: what shallow tests could be replaced by boundary tests

Do not propose interfaces yet. Ask the user which candidate to pursue.

### 3. Frame the Problem Space

For the chosen candidate, explain:

- Constraints the new interface must satisfy
- Dependencies it must rely on
- A rough illustrative sketch to ground the trade-offs

This is not the final proposal. It is a constraints document.

### 4. Design Multiple Interfaces

Design at least 3 materially different interfaces:

1. Minimal surface area
2. Flexible and extensible
3. Optimized for the dominant caller

If the user explicitly asks for sub-agents or parallel design exploration, you may delegate variants in parallel. Otherwise do this locally.

For each design include:

1. Interface signature
2. Usage example
3. Complexity hidden internally
4. Dependency strategy
5. Trade-offs

Then compare them and give a recommendation. If a hybrid is strongest, propose it directly.

### 5. Create The RFC

If the user wants the recommendation captured as a GitHub issue or other tracked artifact, use the template in `REFERENCE.md` and create it in the appropriate system.

Do not publish external artifacts without the user asking for that action.
