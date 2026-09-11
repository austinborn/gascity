# Adoption of Playwright for Frontend Testing: Before Implementing Tests Playwright Test Developers

These rules are ALWAYS ACTIVE for all frontend end-to-end tests and frontend component tests.

### Rules

- **R-PLAYWRIGHT-001** MUST: Before implementing tests with '@playwright/test', developers MUST discover the project's dependency manifest and resolve the exact locked version of the library.

### Verify

```bash
# Discover the project's frontend test runner command and execute all end-to-end tests.
$(discover_frontend_e2e_test_command)

# Discover the project's frontend test runner command and execute all component tests.
$(discover_frontend_component_test_command)
```

**Accept when:**
- All end-to-end tests pass successfully.
- All component tests pass successfully.
- No new test failures are introduced.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>