# Adopt Go Standard Library `slices` Package for Slice Manipulation: Existing Manual Implementations Common Slice Operations

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- R-SLICES-001 SHOULD: Existing manual implementations of common slice operations SHOULD be refactored to use the `slices` package when practical and beneficial for readability or performance.

### Verify

```bash
# Discover and run the project's build command to ensure no compilation errors related to `slices` package usage.
# Discover and execute the project's test command to confirm all tests pass with the new `slices` implementations.
# Discover and run the project's static analysis or linting command to check for adherence to `slices` usage guidelines.
```

**Accept when:**
- The project builds successfully without errors.
- All automated tests pass.
- Static analysis and linting tools report no violations related to slice manipulation patterns.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>