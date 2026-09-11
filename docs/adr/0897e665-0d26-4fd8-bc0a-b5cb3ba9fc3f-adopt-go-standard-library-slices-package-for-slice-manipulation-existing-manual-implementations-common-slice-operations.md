# Adopt Go Standard Library `slices` Package for Slice Manipulation: Existing Manual Implementations Common Slice Operations

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- The codebase frequently performs common operations on slices (e.g., searching, sorting, filtering, modifying).
- Prior to Go 1.21, these operations often required manual implementation or reliance on third-party utility libraries.
- The Go standard library introduced the `slices` package in Go 1.21 to provide standardized, efficient, and type-safe functions for these common operations.
- Multiple internal modules and command-line tools within the project demonstrate consistent usage of the `slices` package.

## Problem Statement

The project requires a consistent, efficient, and idiomatic approach to slice manipulation across its various components to reduce boilerplate, improve readability, and leverage modern Go language features.

## Decision

1. SHOULD: Existing manual implementations of common slice operations SHOULD be refactored to use the `slices` package when practical and beneficial for readability or performance.

## Policy Block

- SHOULD Existing manual implementations of common slice operations SHOULD be refactored to use the `slices` package when practical and beneficial for readability or performance.

In scope:
- All Go source files within the project that perform slice manipulation.
- New feature development and refactoring efforts.

Out of scope:
- Codebases not written in Go.
- Projects using Go versions older than 1.21.

## Rationale

- The `slices` package provides a standardized and idiomatic way to perform common slice operations, improving code consistency and maintainability.
- Using the standard library reduces external dependencies and ensures high-quality, well-tested implementations.
- The functions in `slices` are often optimized for performance and provide type safety, reducing the likelihood of common programming errors.
- Adopting `slices` aligns the codebase with modern Go practices and leverages recent language enhancements.

## Consequences

Positive:
- Improved code readability and maintainability due to standardized slice operations.
- Reduced boilerplate code for common slice manipulations.
- Enhanced type safety and fewer runtime errors related to slice handling.
- Better performance for slice operations due to optimized standard library implementations.

Negative:
- Requires the project to use Go 1.21 or newer, potentially necessitating a Go version upgrade if not already on it.
- Developers unfamiliar with the `slices` package will need to learn its API.

## Alternatives

- Continue with manual slice manipulation or custom utility functions. (rejected)
  Rejected because: Leads to inconsistent code, potential for bugs, and increased boilerplate. Does not leverage modern Go standard library features.
  When valid: Only if the project is constrained to an older Go version (< 1.21) or requires highly specialized, non-standard slice behavior.
- Adopt a third-party utility library for slice operations. (rejected)
  Rejected because: Introduces an additional external dependency, which can increase build times, supply chain risk, and maintenance overhead, especially when a suitable standard library alternative exists.
  When valid: When the standard library `slices` package does not offer a critical feature required by the project, and a well-vetted, actively maintained third-party library provides a significant advantage.

## Risks

- Compatibility issues if the project's Go version is older than 1.21.
  Mitigation: Ensure the project's Go toolchain is updated to version 1.21 or newer. Clearly document the minimum Go version requirement.
  Owner: engineering team
- Developer learning curve for the new `slices` API.
  Mitigation: Provide internal documentation, code examples, and conduct code reviews to ensure correct and idiomatic usage.
  Owner: engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- When refactoring existing code, prioritize areas with high complexity or frequent slice operations to maximize the benefit of using the `slices` package.
- Encourage the use of `go vet` and static analysis tools to identify non-idiomatic slice operations that could be replaced by `slices` functions.

## Continuation Context


Verify commands:
- Discover the project's build command and run it to ensure no compilation errors related to `slices` package usage.
- Discover the project's test command and execute it to confirm all tests pass with the new `slices` implementations.
- Discover the project's static analysis or linting command and run it to check for adherence to `slices` usage guidelines.

Accept when:
- The project builds successfully without errors.
- All automated tests pass.
- Static analysis and linting tools report no violations related to slice manipulation patterns.

## Enforcement

- Verified by: Automated CI/CD pipelines will include checks for Go version compatibility and potential `slices` package misuse.
- Verified by: Code reviews will ensure new code adheres to the `slices` package adoption policy.
- Violation handling: CI/CD pipeline failures will block merges to main branches.
- Violation handling: Code review comments will require remediation before approval.
- Exception process: Exceptions to this policy require a separate ADR detailing the specific justification, alternative approach, and associated risks, approved by at least two senior engineers.