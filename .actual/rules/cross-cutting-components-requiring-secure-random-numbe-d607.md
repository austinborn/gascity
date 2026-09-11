# Go Standard Library Cryptography and Encoding Usage: Components Requiring Secure Random Number Generation

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-CRYPTO-001** MUST: Components requiring secure random number generation MUST utilize the `crypto/rand` standard library package.

### Verify

```bash
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