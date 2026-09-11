# Enforce Minimum TLS 1.2 Version for Secure Communications: Secure Network Communications Configure Tls Config

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-TLS-001** MUST: All secure network communications MUST configure `tls.Config` to enforce a minimum TLS version of 1.2.

### Verify

```bash
# Discover and execute the project's unit tests related to network communication.
# Discover and execute the project's integration tests involving external service calls.
# Discover and execute the project's security scanning tools.
```

**Accept when:**
- All relevant tests pass without errors.
- Security scans report no use of TLS versions older than 1.2.
- Code reviews confirm explicit `MinVersion: tls.VersionTLS12` configuration where applicable.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>