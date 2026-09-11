# Adopt Go Standard Library `slices` Package for Slice Manipulation: Before Implementing Modifying Code That Uses

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-SLICES-001** MUST: Before implementing or modifying code that uses the `slices` package, the developer MUST identify the project's dependency manifest, determine the exact resolved Go version from the repository's lock or resolution artifact, and consult the official documentation for that specific Go version to confirm API existence and behavior.

### Verify

```bash
# Discover and run the project's build command to ensure no compilation errors related to `slices` package usage.
# Example: go build ./...
<PROJECT_BUILD_COMMAND>

# Discover and run the project's test command to confirm all tests pass with the new `slices` implementations.
# Example: go test ./...
<PROJECT_TEST_COMMAND>

# Discover and run the project's static analysis or linting command to check for adherence to `slices` usage guidelines.
# Example: golangci-lint run
<PROJECT_LINT_COMMAND>
```

**Accept when:**
- The project builds successfully without errors.
- All automated tests pass.
- Static analysis and linting tools report no violations related to slice manipulation patterns.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>