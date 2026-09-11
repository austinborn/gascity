# Adoption of Go Standard Library `sort` Package: Sort Package Standard Library Used Ordering

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- The project requires consistent and efficient ordering of various data structures.
- Ensuring deterministic behavior across different command-line tool operations is critical.
- A standardized approach to data manipulation within the Go ecosystem is desired.
- Leveraging built-in language features for common tasks promotes maintainability and reduces external dependencies.

## Decision

1. MUST: The `sort` package from the Go standard library MUST be used for ordering collections of data.

## Policy Block

- MUST The `sort` package from the Go standard library MUST be used for ordering collections of data.

In scope:
- Any Go module or package requiring ordered data structures.

Out of scope:
- Situations where custom, non-standard sorting algorithms are explicitly required for performance or specific domain logic.

## Rationale

- The `sort` package provides efficient, well-tested, and idiomatic sorting algorithms for Go.
- Utilizing the standard library promotes code consistency, reduces the introduction of external dependencies, and leverages optimized implementations.
- This approach aligns with Go's philosophy of providing robust built-in capabilities for common programming tasks.

## Consequences

Positive:
- Improved code readability and reduced boilerplate for sorting operations.
- Consistent and predictable behavior across the codebase due to standardized sorting.
- Leveraging highly optimized and maintained standard library implementations.

Negative:
- Potential for minor performance overhead if highly specialized, custom sorting algorithms are required for extreme edge cases, though `sort.Slice` offers flexibility.

## Alternatives

- Implement custom sorting algorithms. (rejected)
  Rejected because: Reinventing the wheel introduces potential for bugs, increased development effort, and higher maintenance burden when a robust standard solution exists.
  When valid: For highly specialized, performance-critical scenarios where the standard library's capabilities are demonstrably insufficient.
- Use a third-party sorting library. (rejected)
  Rejected because: Introduces an external dependency, potential for API instability, and the standard library already provides comprehensive and robust functionality.
  When valid: If the standard library `sort` package demonstrably lacks a specific feature or performance characteristic that a third-party library provides.

## Risks

- Misunderstanding of `sort` package behavior (e.g., stability of sort for equal elements).
  Mitigation: Thorough testing, code reviews, and adherence to official documentation.
  Owner: Engineering team
- Performance bottlenecks for extremely large datasets if the chosen sorting approach is not optimal.
  Mitigation: Profiling critical sections and careful selection of the most appropriate `sort` function or data structure.
  Owner: Engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Consider the stability requirements of the sort when choosing between `sort.Slice` and other specific functions, as some sorting algorithms are not stable by default.
- Ensure that custom comparison functions provided to `sort.Slice` are correct, efficient, and adhere to the strict weak ordering requirements.

## Continuation Context


Verify commands:
- Inspect relevant Go source files for `import "sort"` statements to confirm usage.
- Execute project tests that involve data ordering to confirm sorting logic behaves as expected.
- Review build configurations and dependency manifests to identify the Go version used and its standard library.

Accept when:
- The `sort` package is consistently imported and used for ordering data structures across the codebase.
- No custom, non-standard sorting implementations are found without clear, documented architectural justification.
- All relevant unit and integration tests involving sorted data pass successfully.

## Enforcement

- Verified by: Automated static analysis tools checking for `import "sort"` and absence of custom sorting.
- Verified by: Code reviews by peers and lead developers.
- Violation handling: Code review comments requiring refactoring to use the `sort` package.
- Violation handling: Automated build failures for non-compliant code.
- Exception process: A formal architectural exception request must be submitted and approved by the lead architect, detailing the specific justification for deviating from this ADR.