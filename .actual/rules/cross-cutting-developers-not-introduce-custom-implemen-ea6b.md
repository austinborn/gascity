# Adopt Go Standard Library `slices` Package for Slice Manipulation: Developers Not Introduce Custom Implementations Slice

These rules are ALWAYS ACTIVE for all files matching the configured scope.

### Rules

- **R-ADR-001** MUST_NOT: Developers MUST NOT introduce custom implementations for slice operations that are already provided by the `slices` package, unless a specific performance or functional requirement cannot be met by the standard library implementation.

### Verify

```bash
# Discover the project's build command and run it to ensure no compilation errors related to `slices` package usage.
# Discover the project's test command and execute it to confirm all tests pass with the new `slices` implementations.
# Discover the project's static analysis or linting command and run it to check for adherence to `slices` usage guidelines.
```

**Accept when:**
- The project builds successfully without errors.
- All automated tests pass.
- Static analysis and linting tools report no violations related to slice manipulation patterns.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>