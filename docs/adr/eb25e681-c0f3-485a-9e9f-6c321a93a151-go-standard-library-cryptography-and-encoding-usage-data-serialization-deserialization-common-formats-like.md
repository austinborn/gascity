# Go Standard Library Cryptography and Encoding Usage: Data Serialization Deserialization Common Formats Like

Status: proposed
Date: 2024-07-30
Deciders: Detection Pipeline (automated)

## Context

- The codebase requires robust cryptographic operations for security, data integrity, and unique identifier generation.
- Efficient and reliable data serialization and deserialization are necessary for inter-component communication and persistence.
- Secure network communication is a fundamental requirement for client-server interactions.
- Leveraging built-in language features reduces external dependencies and potential supply chain risks.

## Decision

1. MUST: Data serialization and deserialization for common formats like hexadecimal, JSON, and Base64 MUST utilize the `encoding/hex`, `encoding/json`, and `encoding/base64` standard library packages, respectively.

## Policy Block

- MUST Data serialization and deserialization for common formats like hexadecimal, JSON, and Base64 MUST utilize the `encoding/hex`, `encoding/json`, and `encoding/base64` standard library packages, respectively.

In scope:
- All Go modules and packages within the project that perform cryptographic operations.
- All Go modules and packages within the project that perform data encoding or decoding.
- All Go modules and packages within the project that establish secure network connections.

Out of scope:

## Rationale

- The Go standard library provides battle-tested, highly optimized, and secure implementations for cryptographic primitives and encoding schemes.
- Using standard library features ensures consistency, reduces the learning curve for new developers, and simplifies maintenance.
- Reliance on core language features minimizes external dependencies, reducing the attack surface and build complexity.
- The observed pattern demonstrates a preference for direct standard library usage over third-party alternatives for these fundamental capabilities.

## Consequences

Positive:
- Enhanced security posture due to the use of well-vetted cryptographic implementations.
- Improved performance and resource efficiency from optimized standard library code.
- Reduced dependency management overhead and fewer transitive dependencies.
- Consistent approach to data handling and security across the codebase.

Negative:
- Developers must adhere to the specific APIs and conventions of the Go standard library, which may require familiarity with its nuances.
- Migration to alternative cryptographic or encoding libraries would require significant refactoring.

## Alternatives

- Use third-party cryptographic libraries (e.g., `golang.org/x/crypto`). (rejected)
  Rejected because: The existing codebase demonstrates a strong preference for standard library usage, which offers sufficient functionality and security without introducing additional external dependencies.
  When valid: For highly specialized or experimental cryptographic algorithms not available in the standard library.
- Implement custom cryptographic or encoding functions. (rejected)
  Rejected because: Custom implementations are prone to security vulnerabilities and are unlikely to match the performance and reliability of the standard library.
  When valid: Never, for security-sensitive operations.

## Risks

- Misuse of cryptographic primitives leading to security vulnerabilities.
  Mitigation: Comprehensive code reviews, security audits, and developer training on secure coding practices.
  Owner: engineering team
- Performance bottlenecks if standard library implementations are not optimally used for specific high-throughput scenarios.
  Mitigation: Performance profiling and optimization for critical paths, potentially exploring specialized libraries if standard library proves insufficient.
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
- Developers should prioritize the most appropriate `crypto/*` package for the specific security requirement (e.g., `sha256` for general-purpose hashing, `ed25519` for digital signatures).
- Ensure proper error handling and context management when interacting with `context` and `io` packages.

## Continuation Context


Verify commands:
- Discover and execute the project's static analysis tools to identify non-compliant cryptographic or encoding implementations.
- Discover and execute the project's unit and integration tests covering security-sensitive components.
- Discover and execute the project's dependency audit tools to ensure no unauthorized cryptographic libraries are introduced.

Accept when:
- Static analysis reports no violations of standard library usage for cryptography and encoding.
- All security-related unit and integration tests pass successfully.
- Dependency audits confirm adherence to approved library usage.

## Enforcement

- Verified by: CI/CD pipelines
- Verified by: Code reviews
- Verified by: Security audits
- Violation handling: Violations will result in build failures in CI/CD and require remediation before merging.
- Exception process: Exceptions require explicit approval from the architecture review board, documented with a clear justification and mitigation plan.