# Adoption of Playwright for Frontend Testing: Test Configurations Playwright Managed Through Defineconfig

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-PLAYWRIGHT-001** SHOULD: Test configurations for '@playwright/test' SHOULD be managed through 'defineConfig' as observed in the project's configuration files.

### Verify

```bash
# Discover the project's frontend test runner command and execute all end-to-end tests.
# Discover the project's frontend test runner command and execute all component tests.
```

**Accept when:**
- All end-to-end tests pass successfully.
- All component tests pass successfully.
- No new test failures are introduced.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>