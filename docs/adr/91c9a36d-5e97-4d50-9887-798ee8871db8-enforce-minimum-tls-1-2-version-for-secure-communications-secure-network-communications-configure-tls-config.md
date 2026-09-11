# Enforce Minimum TLS 1.2 Version for Secure Communications: Secure Network Communications Configure Tls Config

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- Secure communication is critical for protecting data in transit.
- Older TLS versions (e.g., TLS 1.0, TLS 1.1) have known vulnerabilities.
- The `crypto/tls` package in Go provides mechanisms for configuring TLS connections.
- The codebase demonstrates explicit configuration of `MinVersion` for TLS connections.

## Problem Statement

Ensuring that all secure network communications adhere to a minimum security standard by disallowing outdated and vulnerable TLS protocols.

## Decision

1. MUST: All secure network communications MUST configure `tls.Config` to enforce a minimum TLS version of 1.2.

## Policy Block

- MUST All secure network communications MUST configure `tls.Config` to enforce a minimum TLS version of 1.2.

In scope:
- Components making outbound HTTP/S requests
- Internal API clients
- Product metrics reporting

Out of scope:
- Inbound connections where client capabilities might dictate a lower minimum (with appropriate risk assessment)
- Non-TLS connections

## Rationale

- Explicitly setting `MinVersion: tls.VersionTLS12` ensures that connections are established using a secure and modern protocol.
- This practice mitigates risks associated with known vulnerabilities in older TLS versions.
- Consistency in TLS configuration across the codebase improves overall security posture.
- The `crypto/tls` package provides the necessary primitives for this enforcement.

## Consequences

Positive:
- Improved security posture for data in transit.
- Compliance with modern security standards and best practices.
- Reduced risk of man-in-the-middle attacks and data interception.

Negative:
- Potential compatibility issues with very old systems or clients that do not support TLS 1.2.
- Requires careful configuration to avoid breaking existing integrations.

## Alternatives

- Do not explicitly set `MinVersion`. (rejected)
  Rejected because: Relying on default TLS versions may lead to using outdated and insecure protocols if the default changes or is configured insecurely.
  When valid: For non-sensitive internal communications within a highly controlled environment where security risks are minimal.
- Enforce TLS 1.3 as minimum. (deferred)
  Rejected because: While more secure, TLS 1.3 might have broader compatibility implications with existing infrastructure and services not yet supporting it.
  When valid: When all target systems and clients are confirmed to support TLS 1.3, and maximum security is paramount.

## Risks

- Compatibility issues with legacy systems.
  Mitigation: Conduct thorough compatibility testing with all external and internal services.
  Owner: Engineering team.
- Accidental misconfiguration leading to insecure connections or connection failures.
  Mitigation: Implement automated tests and code reviews to verify TLS configurations.
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
- Ensure all new `http.Client` instances or custom `http.Transport` configurations explicitly set `MinVersion: tls.VersionTLS12`.
- Review existing codebases for any `tls.Config` instances that do not specify `MinVersion` or specify an older version.

## Continuation Context


Verify commands:
- Discover and execute the project's unit tests related to network communication.
- Discover and execute the project's integration tests involving external service calls.
- Discover and execute the project's security scanning tools.

Accept when:
- All relevant tests pass without errors.
- Security scans report no use of TLS versions older than 1.2.
- Code reviews confirm explicit `MinVersion: tls.VersionTLS12` configuration where applicable.

## Enforcement

- Verified by: CI/CD pipelines
- Verified by: Code reviews
- Verified by: Security audits
- Violation handling: Automated build failures
- Violation handling: Code review rejection
- Violation handling: Security incident reporting
- Exception process: Formal security review and approval process documented in a separate security policy.