# Adoption of Playwright for Frontend Testing: New End Component Tests Frontend Utilize

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-PLAYWRIGHT-001** MUST: All new end-to-end and component tests for the frontend MUST utilize the '@playwright/test' library.

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