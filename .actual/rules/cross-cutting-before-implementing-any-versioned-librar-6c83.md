# Enforce Minimum TLS 1.2 Version for Secure Communications: Before Implementing Any Versioned Library Exact

These rules are ALWAYS ACTIVE for all code that implements or uses versioned libraries, and for all secure network communications within this project.

### Rules

- **R-LIB-001** MUST: Before implementing any versioned library, the exact resolved version MUST be determined from the project's lock file or resolution artifact.

### Verify

```bash
# Discover and execute the project's unit tests related to network communication.
# Discover and execute the project's integration tests involving external service calls.
# Discover and execute the project's security scanning tools.
#
# For R-LIB-001, verification involves:
# - Reviewing code changes to ensure new library implementations reference exact versions from lock files.
# - Checking build logs or dependency reports for unexpected version resolutions.
```

**Accept when:**
- All relevant tests pass without errors.
- Security scans report no use of TLS versions older than 1.2.
- Code reviews confirm explicit `MinVersion: tls.VersionTLS12` configuration where applicable.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>