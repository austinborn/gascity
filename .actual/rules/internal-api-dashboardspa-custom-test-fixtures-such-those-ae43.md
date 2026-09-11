# Adoption of Playwright for Frontend Testing: Custom Test Fixtures Such Those Base

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-PLAYWRIGHT-001** SHOULD: Custom test fixtures, such as those using 'test as base', SHOULD be used to extend Playwright's testing capabilities and promote reusability.

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