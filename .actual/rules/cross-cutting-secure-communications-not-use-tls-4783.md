# Enforce Minimum TLS 1.2 Version for Secure Communications: Secure Communications Not Use Tls Versions

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-TLS-001** MUST_NOT: Secure communications MUST NOT use TLS versions older than 1.2.

### Verify

```bash
# Discover and execute the project's unit tests related to network communication.
# (e.g., go test ./...)

# Discover and execute the project's integration tests involving external service calls.
# (e.g., make integration-tests)

# Discover and execute the project's security scanning tools.
# (e.g., run security scanner for TLS configuration)
```

**Accept when:**
- All relevant tests pass without errors.
- Security scans report no use of TLS versions older than 1.2.
- Code reviews confirm explicit `MinVersion: tls.VersionTLS12` configuration where applicable.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>