# Enforce Minimum TLS 1.2 Version for Secure Communications: When Establishing New Outbound Http Connections

These rules are ALWAYS ACTIVE for all components making outbound HTTP/S requests, internal API clients, and product metrics reporting.

### Rules

- R-TLS-001 SHOULD: When establishing new outbound HTTP/S connections, `http.Client`'s `Transport.TLSClientConfig` SHOULD be configured with `MinVersion: tls.VersionTLS12`.

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