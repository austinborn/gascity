# Adoption of Core Go Standard Libraries: Use Standard Library Common Programming Tasks

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- Go projects inherently rely on its comprehensive standard library for fundamental operations.
- The codebase frequently requires functionalities such as URL parsing, JSON serialization, string manipulation, and error handling.
- Utilizing standard libraries ensures consistency, performance, and reduces external dependencies.
- The `net/url` package is particularly prevalent for handling URI structures across various internal components.

## Problem Statement

The project needs a consistent and robust approach to common programming tasks without introducing unnecessary third-party dependencies, ensuring maintainability and performance.

## Decision

1. MUST: MUST use the Go standard library for common programming tasks such as URL parsing, JSON encoding/decoding, string manipulation, and error handling.

## Policy Block

- MUST MUST use the Go standard library for common programming tasks such as URL parsing, JSON encoding/decoding, string manipulation, and error handling.

In scope:
- All Go source files within the project.
- Any new Go module or package developed within the project.

Out of scope:

## Rationale

- The Go standard library is mature, well-tested, and provides highly optimized implementations for common functionalities.
- Reliance on standard libraries minimizes the project's external dependency footprint, reducing supply chain risks and build complexity.
- Consistent use of standard library packages promotes code readability and simplifies onboarding for new Go developers.
- The widespread detection of `net/url` and `encoding/json` indicates their established utility and reliability within the existing codebase.

## Consequences

Positive:
- Reduced external dependency count and associated maintenance overhead.
- Improved code consistency and predictability across the codebase.
- Enhanced performance due to optimized standard library implementations.
- Easier code reviews and debugging due to familiarity with standard Go idioms.

Negative:
- May require more verbose code compared to highly specialized third-party libraries for certain niche tasks.
- Developers might need to implement some functionalities manually if a direct standard library equivalent is not available.

## Alternatives

- Introduce third-party libraries for common utilities (e.g., a custom URL parsing library, a different JSON marshaller). (rejected)
  Rejected because: Increases external dependency count, introduces potential compatibility issues, and adds unnecessary complexity when robust standard library alternatives exist.
  When valid: For highly specialized requirements where the standard library cannot meet performance or feature demands.

## Risks

- Developers might unknowingly introduce third-party libraries for functionalities already covered by the standard library.
  Mitigation: Code reviews and automated linters to enforce standard library preference.
  Owner: Engineering team.

## Implementation Notes

- DISCOVERY POLICY (MANDATORY): This ADR omits all tool names, file names, commands, package managers, and version numbers. The consumer MUST derive them from the project repository.

LOCK-VERSION GROUNDING (MANDATORY) — before writing code that uses a versioned library, execute in order:
1. Find the dependency manifest in the repo. It declares ranges, not installed versions.
2. Identify the build tool from the manifest.
3. Inspect the repository lock or resolution artifact to determine the exact resolved version. This artifact is authoritative; build-tool output only verifies the active environment matches it.
4. Look up the official documentation, changelog, or public API reference for that exact version. Do not use training-data recall — fetch or search the public internet for version-specific docs.
5. Confirm every API, class, or function you will call exists in that exact version's documentation before using it.
6. For version-sensitive behavior, re-run steps 3-5 per dependency at point of use.
- Familiarize yourself with the `net/url` package documentation for comprehensive URL handling capabilities.
- Ensure proper error handling when interacting with standard library functions that return errors.

## Continuation Context


Verify commands:
- Run the project's test suite to ensure all existing functionalities relying on standard libraries continue to work as expected.
- Execute the project's linter and static analysis tools to identify any non-standard library imports for common utilities.
- Inspect the dependency manifest to confirm minimal external dependencies.

Accept when:
- All existing tests pass without errors.
- Linter and static analysis tools report no violations related to prohibited third-party utility libraries.
- New code adheres to the preference for Go standard libraries for common tasks.

## Enforcement

- Verified by: Code reviews, automated CI checks, static analysis tools.
- Violation handling: Code will be rejected during code review or fail CI/CD pipelines.
- Exception process: Exceptions require explicit approval from a lead engineer, documented with a clear rationale for deviating from standard library usage.