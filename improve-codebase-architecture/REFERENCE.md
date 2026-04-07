# Reference

## Dependency Categories

When assessing a candidate for deepening, classify its dependencies:

### 1. In-process

Pure computation or in-memory state with no I/O. Merge directly and test at the boundary.

### 2. Local-substitutable

Dependencies with realistic local stand-ins, such as an in-memory filesystem or embedded database. Deepen the module and test it with the local stand-in.

### 3. Remote But Owned

Services you control across a network boundary. Prefer a ports-and-adapters design so the deep module owns logic while transport remains injectable.

### 4. True External

Third-party services you do not control. Mock only at the boundary.

## Testing Strategy

Replace shallow unit tests with stronger boundary tests when the deepened module exists.

- Assert on observable outcomes
- Test through the public interface
- Delete redundant seam-level tests

## Issue Template

<issue-template>

## Problem

Describe:

- Which modules are shallow and tightly coupled
- What risk exists in the seams
- Why navigation, maintenance, or testing is harder than it should be

## Proposed Interface

- Interface signature
- Usage example
- Complexity hidden internally

## Dependency Strategy

- In-process
- Local-substitutable
- Ports and adapters
- Mock boundary

## Testing Strategy

- New boundary tests to write
- Old tests to delete
- Test environment needs

## Implementation Recommendations

- What the module should own
- What it should hide
- What it should expose
- How callers migrate

</issue-template>
