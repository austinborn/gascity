# Go Standard Library Cryptography and Encoding Usage: Components Requiring Cryptographic Hashing Utilize Packages

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-CRYPTO-001** MUST: Components requiring cryptographic hashing MUST utilize packages from the `crypto/sha1`, `crypto/sha256`, or `crypto/ed25519` standard library.

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