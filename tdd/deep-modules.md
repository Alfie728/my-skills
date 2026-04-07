# Deep Modules

A deep module has a small interface and hides substantial implementation complexity.

Use this heuristic during TDD:

- Reduce the number of public entry points
- Simplify parameters where possible
- Hide complexity behind a stable interface
- Prefer testing at the boundary instead of across many shallow seams
