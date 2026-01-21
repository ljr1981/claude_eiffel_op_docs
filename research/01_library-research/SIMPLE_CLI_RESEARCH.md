# simple_cli Research Report

**Date:** 2025-12-08
**Library:** simple_cli (Command Line Interface Argument Parsing)

---

## Step 1: Specifications Research

### POSIX/IEEE Standards
- **IEEE Std 1003.1-2017** (POSIX.1-2017): Defines `getopt()` function and Utility Syntax Guidelines
- **Utility Argument Syntax**: Single-character options preceded by `-`, can be combined (`-abc` = `-a -b -c`)
- **Long options** (GNU extension): `--` prefix, multicharacter (`--verbose`)
- **Option arguments**: Can be required (`:` in optstring) or optional (`::` GNU extension)
- **`--` terminator**: Signals end of options, remaining args are operands

### Key Conventions
- `-h, --help`: Display help
- `-V, --version`: Display version (note: `-v` commonly used for verbose, NOT version)
- Options before operands (POSIX), though GNU allows intermixing
- Environment variable `POSIXLY_CORRECT` enforces strict POSIX mode

**Sources:**
- [POSIX getopt specification](https://pubs.opengroup.org/onlinepubs/9699919799/functions/getopt.html)
- [GNU Argument Syntax](https://www.gnu.org/software/libc/manual/html_node/Argument-Syntax.html)
- [getopt Wikipedia](https://en.wikipedia.org/wiki/Getopt)

---

## Step 2: Tech-Stack Library Analysis

### Rust: Clap (The Gold Standard)
**Key Features:**
- **Derive API**: Define struct with `#[derive(Parser)]`, fields become arguments
- **Builder API**: Runtime flexibility with `Command::new()`
- **Type-safe**: Arguments parsed directly to Rust types
- **Subcommands**: First-class support with nesting
- **Auto-generated**: Help, version, shell completions, man pages
- **Validation**: Built-in and custom validators
- **Environment variables**: Fallback to env vars
- **Value hints**: Tab completion suggestions

**API Design Pattern:**
```rust
#[derive(Parser)]
struct Cli {
    #[arg(short, long)]
    verbose: bool,

    #[arg(short, long, default_value = "output.txt")]
    output: String,

    #[command(subcommand)]
    command: Commands,
}
```

### Go: Cobra
**Key Features:**
- **Command/Args/Flags pattern**: "Commands represent actions, Args are things, Flags are modifiers"
- **Persistent flags**: Inherited by subcommands
- **Pre/Post run hooks**: `PersistentPreRun`, `PreRun`, `PostRun`
- **Auto-generated**: Help, shell completions, man pages
- **Intelligent suggestions**: "Did you mean...?"
- **Used by**: Kubernetes, Docker, Hugo, GitHub CLI

**API Design Pattern:**
```go
var rootCmd = &cobra.Command{
    Use:   "app",
    Short: "Application description",
    Run: func(cmd *cobra.Command, args []string) { },
}
rootCmd.PersistentFlags().BoolVarP(&verbose, "verbose", "v", false, "verbose output")
```

### Python: Typer (Modern) / Click (Mature)
**Key Features:**
- **Type hints**: Function parameters define CLI interface
- **Decorators**: `@app.command()` for subcommands
- **Auto --help**: Generated from docstrings and type info
- **Shell completion**: Bash, Zsh, Fish, PowerShell
- **Progressive disclosure**: Simple API for simple cases

**API Design Pattern:**
```python
@app.command()
def process(
    input_file: Path,
    output: str = "out.txt",
    verbose: bool = typer.Option(False, "--verbose", "-v"),
):
    """Process the input file."""
```

**Sources:**
- [Clap docs](https://docs.rs/clap)
- [Cobra GitHub](https://github.com/spf13/cobra)
- [Typer features](https://typer.tiangolo.com/features/)
- [Click documentation](https://click.palletsprojects.com/)

---

## Step 3: Eiffel Ecosystem Analysis

### ISE argument_parser (OBSOLETE)
- Location: `$ISE_LIBRARY/library/argument_parser/`
- Status: **Marked obsolete in 2012-12-20**
- Classes: `ARGUMENT_BASE_PARSER`, `ARGUMENT_SWITCH`, various switch types
- Note: "Use the new library with Unicode argument support at $ISE_LIBRARY/library/runtime/process/arg_parser"

### ISE arg_parser (Current)
- Location: `$ISE_LIBRARY/library/runtime/process/arg_parser/`
- Full Unicode support (IMMUTABLE_STRING_32)
- Classes: `ARGUMENT_BASE_PARSER`, `ARGUMENT_SINGLE_PARSER`, `ARGUMENT_MULTI_PARSER`
- Switch types: `ARGUMENT_FLAG_SWITCH`, `ARGUMENT_VALUE_SWITCH`, `ARGUMENT_INTEGER_SWITCH`, `ARGUMENT_FILE_SWITCH`, `ARGUMENT_DIRECTORY_SWITCH`
- Validators: `ARGUMENT_DEFAULT_VALIDATOR` and custom validators
- Features: Case sensitivity, grouped switches, separated values

### Gobo Argument Library
- Four-step pattern: Create parser → Create options → Register → Parse
- Classes: `AP_PARSER`, `AP_OPTION` descendants
- Auto-generated help text
- POSIX-style and GNU-style support
- No special "option description language" - pure Eiffel

**Key Gap:** Neither ISE nor Gobo provide:
- Subcommand support (like `git add`, `docker run`)
- Environment variable fallback
- Shell completion generation

---

## Step 4: Developer Pain Points

### Common Frustrations (from research)
1. **Boilerplate explosion**: vLLM has 500+ lines just for command registration
2. **Cross-platform quoting**: JSON in shell args differs between bash/PowerShell/CMD
3. **Convention confusion**: `-v` for verbose vs version
4. **Verbose APIs**: Simple cases require too much setup
5. **Scaling issues**: Simple parsers don't grow well to complex CLIs
6. **Missing features**: Often need to combine multiple libraries

### Specific Complaints
- "None of these libraries fulfill all the requirements"
- "Hardcoded paths and positional arguments with no flexibility"
- "Not-quite-standard command line parsing causes common shortcuts to fail"
- "Multiple incompatible conventions across tools"

**Sources:**
- [Argbash](https://argbash.dev/)
- [CLI Best Practices HN discussion](https://news.ycombinator.com/item?id=12687711)
- [Command line arguments anatomy](https://betterdev.blog/command-line-arguments-anatomy-explained/)

---

## Step 5: Innovation Opportunities

### What Would Make Developers' Lives Easier?

1. **Fluent API with Method Chaining**
   ```eiffel
   cli.flag ("v|verbose", "Enable verbose output")
       .option ("o|output", "Output file", "FILE").with_default ("out.txt")
       .option ("format", "Output format").one_of (<<"json", "xml", "csv">>)
       .required
   ```

2. **Subcommand Support**
   ```eiffel
   cli.command ("add", "Add files to index")
       .option ("all", "Add all files")
       .action (agent handle_add)

   cli.command ("commit", "Record changes")
       .option ("m|message", "Commit message")
       .action (agent handle_commit)
   ```

3. **Type-Safe Value Access**
   ```eiffel
   port := cli.integer_value ("port")  -- Returns INTEGER
   files := cli.path_values ("input")  -- Returns LIST [PATH]
   ```

4. **Environment Variable Fallback**
   ```eiffel
   cli.option ("api-key", "API key").env ("MY_APP_API_KEY")
   ```

5. **Validation Contracts**
   ```eiffel
   cli.option ("port", "Server port")
       .as_integer
       .in_range (1, 65535)

   cli.option ("config", "Config file")
       .as_file
       .must_exist
   ```

6. **Auto Help with Rich Formatting**
   - Grouped options
   - Examples section
   - Default values shown
   - Environment variable hints

7. **Common Patterns Built-In**
   - `--help` / `-h` automatic
   - `--version` / `-V` automatic
   - `--` end-of-options automatic

---

## Step 6: Design Strategy Synthesis

### Core Design Principles
1. **Simple by default**: One-liner for basic cases
2. **Progressive disclosure**: Power available but not required
3. **Fail-fast**: Clear errors at parse time
4. **Contract-driven**: Eiffel contracts for validation

### What simple_cli Should Be
- A simple wrapper that provides clean API
- Support for common 80% use cases
- Optional advanced features

### What simple_cli Should NOT Be
- A full replacement for ISE arg_parser
- A subcommand framework (that could be simple_cli_commands)
- A configuration file parser (that's simple_config)

### API Surface Design

**Minimal API (covers 80% of cases):**
```eiffel
create cli.make
cli.set_app_info ("myapp", "My application", "1.0.0")
cli.add_flag ("v|verbose", "Enable verbose output")
cli.add_option ("o|output", "Output file", "FILE")
cli.parse

if cli.has_flag ("verbose") then ...
if attached cli.option_value ("output") as out then ...
```

**Extended API (optional power features):**
```eiffel
-- Default values
cli.add_option_with_default ("port", "Server port", "PORT", "8080")

-- Required options
cli.add_required_option ("config", "Config file", "FILE")

-- Environment fallback
cli.add_option_with_env ("api-key", "API key", "KEY", "MY_API_KEY")

-- Type-safe access
port := cli.integer_option ("port", 8080)  -- with default
```

---

## Step 7: Gap Analysis - Current Implementation

### What Current Implementation Has
- ✅ Short/long flag support (`-v`, `--verbose`)
- ✅ Short/long option support with values (`-o file`, `--output=file`)
- ✅ Automatic help text generation
- ✅ Error collection and reporting
- ✅ Positional arguments access
- ✅ Command extraction (first positional arg)
- ✅ Case-insensitive matching

### What Current Implementation Lacks
- ❌ Default values for options
- ❌ Required flag for options/arguments
- ❌ Type validation (integer, file exists, etc.)
- ❌ Environment variable fallback
- ❌ Built-in `--help` and `--version` handling
- ❌ Mutually exclusive option groups
- ❌ Option value validation (one-of, range)
- ❌ Subcommand support

### Technical Issues
- Inherits from `ARGUMENTS_32` (okay, but not leveraging ISE arg_parser)
- No Unicode considerations documented
- No shell completion support

---

## Recommendations

### Phase 1: Essential Improvements (Recommended)
1. Add default values for options
2. Add required option support
3. Add built-in `--help` and `--version` handling
4. Add integer/boolean type helpers

### Phase 2: Nice-to-Have (Optional)
1. Environment variable fallback
2. One-of validation for options
3. File/directory existence validation

### Phase 3: Future (Out of Scope for Now)
1. Subcommand framework
2. Shell completion generation
3. Man page generation

---

## Conclusion

The current simple_cli implementation provides a functional basic CLI parser. However, compared to modern standards (Clap, Cobra, Typer), it lacks several features that developers expect:

**Must Have:**
- Default values
- Required options
- Built-in help/version flags

**Should Have:**
- Type-safe accessors (integer, boolean)
- Basic validation

The implementation is a reasonable starting point but should be enhanced before being considered production-ready for the simple_* ecosystem.
