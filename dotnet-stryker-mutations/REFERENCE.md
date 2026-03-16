# Stryker.NET Reference

Detailed configuration, setup, and troubleshooting for Stryker.NET mutation testing.

## Configuration

Stryker reads `stryker-config.json` from the current directory. This file ensures consistent reporters, thresholds, and reproducible settings.

### Config Template

Create `stryker-config.json` in your test project directory:

```json
{
  "stryker-config": {
    "project-info": {
      "name": "YourSolution",
      "module": "YourProject",
      "version": ""
    },
    "concurrency": 8,
    "mutation-level": "Standard",
    "language-version": "latest",
    "additional-timeout": 5000,
    "mutate": ["**/src/YourProject/**/*.cs"],
    "target-framework": "net<VERSION>.0",
    "project": "YourProject.csproj",
    "coverage-analysis": "perTest",
    "thresholds": {
      "high": 80,
      "low": 60,
      "break": 0
    },
    "reporters": ["Progress", "Html", "Json", "Cleartext"],
    "test-projects": ["YourProject.UnitTest.csproj"],
    "ignore-methods": ["System.Diagnostics.Debug.Assert"]
  }
}
```

Replace `YourProject` and `YourSolution` with actual names.

**Detect `target-framework` automatically** -- don't hardcode it. Use one of:
```bash
# From global.json (preferred -- look for sdk.version or check TargetFramework in .csproj)
grep -oP '"version"\s*:\s*"\K[0-9]+' global.json | head -1  # e.g. "10" -> net10.0

# From the test project's .csproj
grep -oP '<TargetFramework>\K[^<]+' path/to/YourProject.UnitTest.csproj  # e.g. net10.0

# From dotnet SDK (fallback)
dotnet --version | cut -d. -f1  # e.g. "10" -> net10.0
```

### Required Settings

| Setting | Value | Why |
|---------|-------|-----|
| `reporters` | `["Progress", "Html", "Json", "Cleartext"]` | HTML for review, JSON for tooling, Cleartext for console |
| `mutate` | Glob for source files | Tells Stryker what to mutate |

### When to Add `"configuration": "Release"`

Only needed when the source project disables reference assemblies in Debug mode. This can be set directly in a `.csproj`, in `Directory.Build.props`, or in a shared `.props` file.

**Detect it automatically** using MSBuild property evaluation:
```bash
# Find the source .csproj that Stryker will mutate
dotnet msbuild -getProperty:ProduceReferenceAssembly -p:Configuration=Debug path/to/YourProject.csproj
```

- If the result is `false` --> add `"configuration": "Release"` to `stryker-config.json`
- If the result is `true` (or empty) --> leave it out, Debug works fine

### Recommended Settings

| Setting | Default | Description |
|---------|---------|-------------|
| `concurrency` | CPU cores | Parallel test runners |
| `thresholds.high` | 80 | Target mutation score |
| `thresholds.low` | 60 | Warning threshold |
| `thresholds.break` | 0 | Fail threshold (0 = never fail) |
| `coverage-analysis` | `"perTest"` | Only run tests covering the mutated code |
| `ignore-methods` | `[]` | Skip mutations in specific method calls |

## Common Commands

| Command | Description |
|---------|-------------|
| `dotnet stryker` | Run full mutation testing |
| `dotnet stryker --mutate "**/MyClass.cs"` | Mutate a specific file only |
| `dotnet stryker --since:main` | Only test code changed since main |
| `dotnet stryker -o` | Open HTML report after run |
| `dotnet stryker --verbosity debug` | Verbose output for debugging |

The `--mutate` flag restricts which files are mutated but all other config settings still apply.

## Reports

Stryker writes reports to `StrykerOutput/<timestamp>/reports/`:

- **HTML** (`mutation-report.html`): Interactive, filterable -- best for reviewing mutants
- **JSON** (`mutation-report.json`): Machine-readable, good for CI integration
- **Cleartext**: Printed to console during run

Find the latest reports:
```bash
ls -td StrykerOutput/*/ | head -1 | xargs -I{} ls {}reports/
```

## Adding Stryker to a New Test Project

1. Ensure `dotnet-stryker` is installed as a local tool:
   ```bash
   dotnet tool install dotnet-stryker
   # or update: dotnet tool update dotnet-stryker
   ```

2. Navigate to the test project directory:
   ```bash
   cd path/to/YourProject.UnitTest
   ```

3. Create `stryker-config.json` using the template above.

4. Check if Release mode is needed (see "When to Add `configuration: Release`"):
   ```bash
   dotnet msbuild -getProperty:ProduceReferenceAssembly -p:Configuration=Debug path/to/YourProject.csproj
   ```
   If `false`, add `"configuration": "Release"` to the config.

5. Verify the config:
   - `"mutate"` pattern matches your source files
   - `"project"` points to the source project's `.csproj`

6. Run: `dotnet stryker`

## Troubleshooting

### "Could not find file ... ref/... .dll" (FileNotFoundException)

**Cause:** The source project disables reference assembly production in Debug mode.

**Diagnose:**
```bash
dotnet msbuild -getProperty:ProduceReferenceAssembly -p:Configuration=Debug path/to/YourProject.csproj
```

**Fix:** If the result is `false`, add `"configuration": "Release"` to `stryker-config.json`, or pass `--configuration Release` on the command line.

### "Commandline could not be parsed" (FormatException)

**Cause:** Stryker version too old for your .NET SDK.

**Fix:** Update to latest Stryker:
```bash
dotnet tool update dotnet-stryker
```

Check version in `.config/dotnet-tools.json`. Minimum 4.11.0 for .NET 10 support.

### Tests Failing in Initial Run

Some tests may fail during mutation testing that pass normally (e.g., timing-sensitive tests). Check the Stryker output for details. Use `--break-on-initial-test-failure` to stop early.

### Build Takes Too Long

First run compiles all dependencies. Subsequent runs use incremental compilation. Use `--since:main` for faster iterations during development.

## Stryker Disable Comments (Use Sparingly)

When a mutant is genuinely equivalent (both versions produce identical observable output for all inputs), you may suppress it:

```csharp
// Stryker disable once Arithmetic : explain why this is equivalent
var result = x + 0;
```

Available granularity:
- `// Stryker disable once <Mutator>` -- single line
- `// Stryker disable all` / `// Stryker restore all` -- block

**Mutator names:** `Arithmetic`, `Equality`, `Boolean`, `String`, `Linq`, `Assignment`, `Unary`, `Update`, `Checked`, `Block`, `MethodCall`, `Regex`, `NullCoalescing`, `Conditional`, `StringMethod`.

## References

- [Stryker.NET Introduction](https://stryker-mutator.io/docs/stryker-net/introduction/)
- [Configuration Reference](https://stryker-mutator.io/docs/stryker-net/configuration/)
- [Stryker.NET GitHub](https://github.com/stryker-mutator/stryker-net)
- [Mutation Types](https://stryker-mutator.io/docs/stryker-net/mutations/)
