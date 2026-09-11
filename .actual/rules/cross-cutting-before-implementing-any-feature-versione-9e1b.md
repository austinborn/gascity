# Go Standard Library Cryptography and Encoding Usage: Before Implementing Any Feature Versioned Dependency

These rules are ALWAYS ACTIVE for all Go modules and packages within the project that perform cryptographic operations, data encoding or decoding, or establish secure network connections.

### Rules

- **R-DEP-VER-001** MUST: Before implementing any feature using a versioned dependency, the consumer MUST discover the project's dependency manifest, identify the build tool, inspect the repository lock or resolution artifact to determine the exact resolved version, and consult the official documentation for that specific version.

### Verify

```bash
# The ADR's "DISCOVERY POLICY" omits specific tool names and commands.
# The following describes the types of verification required:

# Discover and execute the project's static analysis tools to identify non-compliant cryptographic or encoding implementations.
# Discover and execute the project's unit and integration tests covering security-sensitive components.
# Discover and execute the project's dependency audit tools to ensure no unauthorized cryptographic libraries are introduced.
```

**Accept when:**
- Static analysis reports no violations of standard library usage for cryptography and encoding.
- All security-related unit and integration tests pass successfully.
- Dependency audits confirm adherence to approved library usage.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>
