# Standardized Integration Test Report Processing with Glob and JSON: When Interacting Versioned Dependencies Consumers Discover

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- Automated integration tests generate structured reports in JSON format.
- These reports need to be collected and processed by CI/CD scripts for various purposes.
- Consistent parsing and aggregation are required for reliable pipeline execution and decision-making.
- Environment variables are frequently used for dynamic configuration of report directories and output paths.

## Problem Statement

The project requires a consistent and reliable method for discovering, parsing, and utilizing JSON-formatted integration test reports within automated workflows to enable features like summary generation, retry logic, and policy enforcement.

## Decision

1. MUST: When interacting with versioned dependencies, consumers MUST discover the ecosystem's lock file and resolve the exact locked version before implementation.

## Policy Block

- MUST When interacting with versioned dependencies, consumers MUST discover the ecosystem's lock file and resolve the exact locked version before implementation.

In scope:
- Python scripts within automated workflows that process integration test reports.

Out of scope:
- Manual report analysis.
- Test reports not in JSON format.
- Unit test reports that do not require aggregation.

## Rationale

- Using `glob` provides a flexible and standard way to discover files matching patterns, essential for dynamic report directories.
- `json` offers a universal, language-agnostic format for structured data, ensuring interoperability and ease of parsing across different scripts.
- Standardizing these libraries reduces cognitive load and promotes consistency in CI/CD scripting.
- This approach supports automated decision-making based on test results, such as triggering retries or generating summaries.

## Consequences

Positive:
- Consistent and reliable processing of integration test reports.
- Easier maintenance and understanding of CI/CD scripts.
- Facilitates automated actions based on test outcomes.
- Improved data exchange between different workflow steps.

Negative:
- Reliance on specific Python libraries for report handling.
- Requires reports to conform to a JSON structure.

## Alternatives

- Manual file listing and parsing. (rejected)
  Rejected because: Prone to errors, difficult to maintain, and not scalable for dynamic report generation.
  When valid: For very simple, static report structures with minimal files.
- Using a custom parsing library or framework. (rejected)
  Rejected because: Introduces additional dependency overhead and complexity without clear benefits over standard `json` and `glob`.
  When valid: If reports require highly specialized parsing logic not easily handled by `json`.

## Risks

- Changes in report format break parsing logic.
  Mitigation: Implement schema validation for JSON reports; ensure robust error handling in parsing scripts.
  Owner: engineering team
- Performance issues with `glob` on very large directories.
  Mitigation: Optimize `glob` patterns; consider alternative file discovery methods for extremely large scale.
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
- Ensure error handling is robust for `json.load` to gracefully manage malformed report files.
- Consider using `pathlib` for more object-oriented path manipulation in conjunction with `glob`.

## Continuation Context


Verify commands:
- Discover the project's CI/CD workflow definitions and identify scripts that process test reports.
- Execute the identified scripts in a test environment with sample JSON reports.
- Inspect the output or generated artifacts for correct processing and aggregation.

Accept when:
- The scripts successfully discover all relevant JSON report files using `glob`.
- The scripts successfully parse the JSON content without errors using `json`.
- The aggregated or processed results accurately reflect the input reports.

## Enforcement

- Verified by: Code reviews
- Verified by: Automated linting for `glob` and `json` usage in relevant scripts
- Verified by: CI/CD pipeline execution logs
- Violation handling: Code review comments
- Violation handling: CI/CD job failures
- Violation handling: Automated alerts
- Exception process: Documented architectural review and explicit approval from lead engineers for deviations.