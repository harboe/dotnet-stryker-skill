---
name: dotnet-stryker-mutations
description: Run Stryker.NET mutation testing to evaluate test quality, identify weak tests, and kill surviving mutants. Use when user mentions "mutation testing", "stryker", "mutant", "mutation score", or wants to verify test effectiveness beyond code coverage.
---

# Stryker.NET Mutation Testing

[Stryker.NET](https://stryker-mutator.io/docs/stryker-net/introduction/) mutates your source code and checks whether tests detect the changes. A high mutation score means your tests catch real bugs, not just execute lines.

## Quick Start

**Always run from a directory containing `stryker-config.json`.**

```bash
# Navigate to the test project directory
cd path/to/MyProject.UnitTest

# Verify config exists (see REFERENCE.md to create one)
ls stryker-config.json

# Run mutation testing
dotnet stryker

# Focus on a single file
dotnet stryker --mutate "**/MyClass.cs"

# Only mutate code changed since main
dotnet stryker --since:main

# Run and open HTML report
dotnet stryker -o
```

## Workflow: Killing Survived Mutants

1. Run Stryker and find the reports:
   ```bash
   # HTML (interactive), JSON (machine-readable), and Cleartext (console)
   ls -td StrykerOutput/*/ | head -1 | xargs -I{} ls {}reports/
   ```
2. Identify **Survived** mutants -- these are gaps in your tests.
3. For each survivor, read the mutation and surrounding source carefully.
4. **Write a test that distinguishes mutated from original behaviour.**
5. Re-run Stryker to confirm the mutant is killed.

### Key Rule: Prefer Real Tests Over Suppression

**Always write a test to kill a mutant.** Only consider `// Stryker disable once <mutator>` as a last resort for genuinely equivalent mutants, and only with explicit user approval. Never pad the score with disable comments.

### Writing Tests That Kill Mutants

If the `tdd` skill is available, load it -- its test philosophy applies directly:
- **Test observable behaviour through public interfaces**, not implementation details. Tests coupled to internals are weak mutant killers.
- **One test per mutant** -- vertical slices, not batch. Understand the mutation, write a focused test, confirm the kill, move on.
- **Mock only at system boundaries** (external APIs, I/O). Mocking internal collaborators hides the very mutations you need to catch.

## Common Mutant Patterns

**Boundary mutations** (`>` to `>=`): Add a test for the exact boundary value.
```csharp
// Mutant: count > 0  -->  count >= 0 (survived)
// Fix: test with count == 0
```

**String check mutations** (`IsNullOrWhiteSpace` to `IsNullOrEmpty`): Cover null, empty, AND whitespace.
```csharp
[TestCase(null)]
[TestCase("")]
[TestCase("   ")]  // kills the IsNullOrEmpty mutation
public void Method_WhenInvalid_Fails(string? input) { ... }
```

**Boolean/logic mutations**: Ensure tests cover both true and false branches.

## Understanding Results

| Status | Meaning | Action |
|--------|---------|--------|
| Killed | Test caught the mutant | None -- working as intended |
| Survived | No test detected the change | Write a better test |
| No Coverage | Code not reached by any test | Add test coverage first |
| Timeout | Test hung on mutant | Usually counts as killed |

**Target:** 80%+ mutation score.

## Configuration & Troubleshooting

See [REFERENCE.md](REFERENCE.md) for:
- `stryker-config.json` template and required settings
- Adding Stryker to a new test project
- Troubleshooting common errors (missing DLLs, version issues)
- Recommended and advanced settings
