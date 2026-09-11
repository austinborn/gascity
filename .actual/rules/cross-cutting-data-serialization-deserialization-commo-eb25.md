# Go Standard Library Cryptography and Encoding Usage: Data Serialization Deserialization Common Formats Like

These rules are ALWAYS ACTIVE for all Go modules and packages within the project that perform cryptographic operations, data encoding/decoding, or secure network connections.

### Rules

- **R-GO-001** MUST: Data serialization and deserialization for common formats like hexadecimal, JSON, and Base64 MUST utilize the `encoding/hex`, `encoding/json`, and `encoding/base64` standard library packages, respectively.

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