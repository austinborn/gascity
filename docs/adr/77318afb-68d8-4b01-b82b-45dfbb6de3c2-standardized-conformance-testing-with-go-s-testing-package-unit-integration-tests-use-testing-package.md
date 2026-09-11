# Standardized Conformance Testing with Go's `testing` Package: Unit Integration Tests Use Testing Package

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- Internal modules require robust and consistent validation against their defined contracts and expected behaviors.
- The codebase exhibits a recurring pattern of encapsulating common test logic into reusable functions.
- Different architectural layers (data access, service boundaries) and concurrency models are subject to specific conformance checks.
- The Go `testing` package provides the foundational primitives for structuring these tests.

## Decision

1. MUST: All unit and integration tests MUST use the `testing` package.

## Policy Block

- MUST All unit and integration tests MUST use the `testing` package.

In scope:
- Internal Go modules and their associated test suites.

Out of scope:
- External-facing API contracts, end-to-end tests, or non-Go language components.

## Rationale

- Standardizing test patterns ensures consistency across the codebase, making tests easier to understand and maintain.
- Encapsulating test logic in `Run*Tests` functions promotes reusability and reduces boilerplate, improving developer efficiency.
- Consistent use of `testing` package primitives like `t.Run` and `t.Fatalf` enforces a uniform approach to test execution and failure reporting.
- This pattern helps enforce architectural contracts and prevent regressions in internal module behaviors.

## Consequences

Positive:
- Increased test coverage and reliability for internal modules.
- Reduced effort in writing new tests due to reusable conformance suites.
- Improved maintainability and readability of test code.
- Stronger enforcement of module contracts and architectural boundaries.

Negative:
- Initial overhead in developing and maintaining generic `Run*Tests` functions.
- Potential for over-abstraction if test patterns are too rigidly applied to unique testing scenarios.

## Alternatives

- Ad-hoc testing with direct use of `testing` primitives. (rejected)
  Rejected because: Leads to inconsistent test structures, increased boilerplate, and difficulty in enforcing common conformance checks.
  When valid: For very simple, single-file utilities where a full conformance suite is overkill.
- Use a third-party testing framework. (rejected)
  Rejected because: Introduces an external dependency and deviates from the observed pattern of leveraging Go's native `testing` package, which is already widely adopted in the codebase.
  When valid: If the native `testing` package proves insufficient for complex testing requirements not met by the current pattern.

## Risks

- Over-engineering of `Run*Tests` functions.
  Mitigation: Regularly review and refactor generic test functions to ensure they remain focused and avoid unnecessary complexity.
  Owner: engineering team
- Stale or outdated conformance tests.
  Mitigation: Integrate test suite maintenance into regular development cycles and ensure tests are updated with module changes.
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
- New internal modules should identify existing `Run*Tests` functions that align with their contracts and extend them or create new ones following the established pattern.
- Ensure that test helper functions are well-documented to facilitate their reuse and understanding.

## Continuation Context


Verify commands:
- Discover the project's build command for running tests.
- Execute the build command to run all unit and integration tests.
- Inspect the test output for any failures or skipped conformance checks.

Accept when:
- All unit and integration tests pass without errors.
- No conformance checks are unexpectedly skipped or ignored.
- Test output indicates adherence to expected module behaviors.

## Enforcement

- Verified by: CI/CD pipelines
- Verified by: Code reviews
- Violation handling: Code review comments
- Violation handling: CI/CD pipeline failures
- Exception process: Documented architectural review and approval process