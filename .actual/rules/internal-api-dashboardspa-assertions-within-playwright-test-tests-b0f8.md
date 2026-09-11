# Adoption of Playwright for Frontend Testing: Assertions Within Playwright Test Tests Use

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-PLAYWRIGHT-001** MUST: Assertions within '@playwright/test' tests MUST use the 'expect' API provided by the library.

### Verify

```bash
# Discover the project's frontend test runner command and execute all end-to-end tests.
# Example: npx playwright test --project=e2e

# Discover the project's frontend test runner command and execute all component tests.
# Example: npx playwright test --project=component
```

**Accept when:**
- All end-to-end tests pass successfully.
- All component tests pass successfully.
- No new test failures are introduced.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>