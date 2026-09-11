# Adopt Go Standard Library `slices` Package for Slice Manipulation: New Modified Code Requiring Common Slice

These rules are ALWAYS ACTIVE for all Go source files within the project that perform slice manipulation, and apply to new feature development and refactoring efforts.

### Rules

- **R-SLICES-001** MUST: All new and modified code requiring common slice manipulation operations MUST utilize functions provided by the `slices` package from the Go standard library.

### Verify

```bash
$(discover_build_command) # Ensure no compilation errors related to `slices` package usage.
$(discover_test_command) # Confirm all tests pass with the new `slices` implementations.
$(discover_static_analysis_command) # Check for adherence to `slices` usage guidelines.
```

**Accept when:**
- The project builds successfully without errors.
- All automated tests pass.
- Static analysis and linting tools report no violations related to slice manipulation patterns.

<enforcement>
Claude Code MUST NOT skip or defer verification.
</enforcement>