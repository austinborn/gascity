# Adoption of Playwright for Frontend Testing: Before Implementing Tests Playwright Test Developers

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- The frontend application requires robust end-to-end testing.
- Reliable browser automation is necessary to simulate user interactions.
- A consistent testing framework across the frontend codebase improves maintainability.
- The existing codebase demonstrates usage of '@playwright/test' for various testing concerns.

## Problem Statement

Ensuring the quality and functional correctness of the frontend application through automated testing requires a standardized and capable testing framework.

## Decision

1. MUST: Before implementing tests with '@playwright/test', developers MUST discover the project's dependency manifest and resolve the exact locked version of the library.

## Policy Block

- MUST Before implementing tests with '@playwright/test', developers MUST discover the project's dependency manifest and resolve the exact locked version of the library.

In scope:
- Frontend end-to-end tests
- Frontend component tests

Out of scope:
- Backend tests
- Unit tests not involving browser interaction
- Integration tests not requiring a full browser environment

## Rationale

- '@playwright/test' provides a comprehensive solution for browser automation and testing, supporting various browser engines.
- Its API design promotes readable and maintainable test code, as evidenced by existing test files.
- The framework's capabilities align with the project's need for reliable and efficient frontend testing.

## Consequences

Positive:
- Improved test reliability and reduced flakiness in frontend tests.
- Standardized approach to writing end-to-end and component tests.
- Better developer experience for writing and debugging browser tests.

Negative:
- Learning curve for developers unfamiliar with Playwright.
- Potential for increased test execution time compared to unit tests.

## Alternatives

- Cypress (rejected)
  Rejected because: While a capable E2E framework, Playwright is already adopted and integrated into the existing codebase, making Cypress a redundant addition.
  When valid: For new projects without an established E2E framework.
- Jest with JSDOM (rejected)
  Rejected because: Jest with JSDOM does not provide a true browser environment, which is essential for accurate end-to-end testing of the frontend application.
  When valid: For pure unit testing of JavaScript logic without DOM interaction.

## Risks

- Test maintenance burden due to UI changes.
  Mitigation: Implement robust page object models and reusable test components.
  Owner: engineering team
- Performance overhead of running browser-based tests.
  Mitigation: Optimize test execution, run tests in parallel, and utilize CI/CD for efficient execution.
  Owner: engineering team

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.
- LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Developers should refer to the official Playwright documentation for best practices and advanced features.
- Ensure that test selectors are resilient to UI changes to minimize test flakiness.

## Continuation Context


Verify commands:
- Discover the project's frontend test runner command and execute all end-to-end tests.
- Discover the project's frontend test runner command and execute all component tests.

Accept when:
- All end-to-end tests pass successfully.
- All component tests pass successfully.
- No new test failures are introduced.

## Enforcement

- Verified by: Continuous Integration (CI) pipelines
- Verified by: Code reviews
- Violation handling: CI pipeline failures
- Violation handling: Code review comments requiring remediation
- Exception process: Exceptions require explicit approval from a lead engineer, documented with a clear rationale.